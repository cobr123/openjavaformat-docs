# GitHub Action and pre-commit

Both checks run the native binary, so they need no Java, Maven or Gradle. They work on Linux with
glibc, on macOS and on Windows on x86-64. There is no native binary for musl-based Linux such as
Alpine or for Windows on ARM.

## Name the version

Both checks take the formatter version from `.open-java-format-version` in the repository root, one
of the [releases](https://github.com/openjavaformat/open-java-format/releases). There is one file
for the two of them, so they cannot drift apart; without it, both stop and ask for it.

``` text title=".open-java-format-version"
{{ ojf_version }}
```

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

      - uses: openjavaformat/open-java-format-action@v4
        with:
          mode: {% raw %}${{ github.event_name == 'push' && 'all' || 'changed' }}{% endraw %}
```

The [action](https://github.com/openjavaformat/open-java-format-action) downloads the binary, lists
the files that are not formatted and fails the job if there are any.

| Input | Default | Meaning |
| --- | --- | --- |
| `mode` | `changed` | `changed` checks the files of the pull request or push, `all` checks every `.java` file |
| `code-scanning` | `false` | Also [report a newer release to code scanning](#see-when-a-newer-release-is-out) |

A workflow that still pins 2.98.0.1 stays on `@v1`: that version needs the `--ojf` flag, which later
majors do not pass. `@v2` and `@v3` take the version as a `version` input instead of the file.

On a pull request `changed` takes the file list from the pull request itself. On a push it compares
the commits before and after, which needs that history in the checkout: either use `mode: all` for
pushes, as above, or check out with `fetch-depth: 0`.

## See when a newer release is out

On every run the action compares the version in `.open-java-format-version` with the latest release
and warns in the job log when a newer one is out. The warning can also become a code scanning alert,
which shows under **Security → Code scanning** and in the checks of a pull request, points at
`.open-java-format-version` and closes on its own once the version is updated. That takes
`code-scanning: true` and the `security-events: write` permission, which the default token does not
have. The `permissions` block replaces the job's default permissions, so it names everything else
the job needs as well.

``` yaml title=".github/workflows/format.yml"
jobs:
  format:
    runs-on: ubuntu-latest
    permissions:
      contents: read # the checkout
      pull-requests: read # mode: changed reads the files of the pull request
      security-events: write # the code scanning upload
    steps:
      - uses: actions/checkout@v7

      - uses: openjavaformat/open-java-format-action@v4
        with:
          mode: {% raw %}${{ github.event_name == 'push' && 'all' || 'changed' }}{% endraw %}
          code-scanning: true
```

Without the permission the upload is skipped with a line in the log and the job passes as before.
Code scanning is free on public repositories; a private repository needs GitHub Code Security.

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
curl -fsSL https://raw.githubusercontent.com/openjavaformat/open-java-format-action/v4/pre-commit -o .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

On its first run the hook downloads the native binary and caches it in `~/.cache/open-java-format`.
After that it checks only the staged `.java` files of each commit, and when one is not formatted it
prints the command that fixes it.

To share the hook with the whole team, keep it in the repository instead:

``` sh
mkdir -p .githooks
curl -fsSL https://raw.githubusercontent.com/openjavaformat/open-java-format-action/v4/pre-commit -o .githooks/pre-commit
chmod +x .githooks/pre-commit
git config core.hooksPath .githooks
```

The hook reads the same [`.open-java-format-version`](#name-the-version) as the action, and stops
the commit with a message when the file is missing.
