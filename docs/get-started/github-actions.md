# GitHub Action and pre-commit

Both checks run the native binary, so they need no Java, Maven or Gradle. They work on Linux with
glibc, on macOS and on Windows on x86-64. There is no native binary for musl-based Linux such as
Alpine or for Windows on ARM.

## Check pull requests and pushes

``` yaml title=".github/workflows/format.yml"
on:
  pull_request:
  push:
    branches: [main]

jobs:
  format:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7

      - uses: openjavaformat/open-java-format-action@v2
        with:
          version: '{{ ojf_version }}'
          mode: {% raw %}${{ github.event_name == 'push' && 'all' || 'changed' }}{% endraw %}
```

The [action](https://github.com/openjavaformat/open-java-format-action) downloads the binary, lists
the files that are not formatted and fails the job if there are any.

| Input | Default | Meaning |
| --- | --- | --- |
| `version` | `{{ ojf_version }}` | The formatter version to download |
| `mode` | `changed` | `changed` checks the files of the pull request or push, `all` checks every `.java` file |

A workflow that still pins 2.98.0.1 stays on `@v1`: that version needs the `--ojf` flag, which `@v2`
does not pass.

On a pull request `changed` takes the file list from the pull request itself. On a push it compares
the commits before and after, which needs that history in the checkout: either use `mode: all` for
pushes, as above, or check out with `fetch-depth: 0`.

## Fix what the check found

Run the formatter locally with the same version, then commit the result. See
[Command line](command-line.md) for the download.

``` sh
open-java-format --replace path/to/File.java
```

## Exclude files

Both the action and the hook read an optional `.open-java-format-exclude` file in the repository
root. Every line that is not empty and not a comment is a git pathspec, and a Java file that matches
one is skipped.

``` gitignore title=".open-java-format-exclude"
# Code copied from another project, kept as it came
third_party/**

# Generated sources
**/build/generated/**
```

## Git pre-commit hook

The same check can stop a commit before it reaches CI. Copy the hook script from the action's
repository into your project:

``` sh
curl -fsSL https://raw.githubusercontent.com/openjavaformat/open-java-format-action/v2/pre-commit -o .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

On its first run the hook downloads the native binary and caches it in `~/.cache/open-java-format`.
After that it checks only the staged `.java` files of each commit, and when one is not formatted it
prints the command that fixes it.

To share the hook with the whole team, keep it in the repository instead:

``` sh
mkdir -p .githooks
curl -fsSL https://raw.githubusercontent.com/openjavaformat/open-java-format-action/v2/pre-commit -o .githooks/pre-commit
chmod +x .githooks/pre-commit
git config core.hooksPath .githooks
```

The script pins its formatter version in `FORMATTER_VERSION` at the top. Keep it in step with the
`version` your workflow passes to the action.
