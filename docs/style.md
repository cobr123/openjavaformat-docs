# Style

open-java-format has one style and no settings. It is the style of palantir-java-format 2.x, which
grew out of google-java-format: lines of up to 120 characters, 4 spaces for each level of
indentation, and 8 more for a line that continues a statement. Every example on this page is the
formatter's own output.

## Wrapping

When a declaration does not fit on one line, its parameters move to a continuation line together.

``` java
class Params {
    public static ResolvedConfiguration resolveConfigurationForProject(
            Project project, String configurationName, boolean includeTransitiveDependencies) {
        return resolve(project, configurationName, includeTransitiveDependencies);
    }
}
```

## Lambdas

A lambda starts on the line of the call it is passed to, and its body is indented by one level from
that line, however deeply the call is nested.

``` java
class Tasks {
    void register(Project project) {
        project.getTasks().register("formatDiff", FormatDiffTask.class, task -> {
            task.setGroup("formatting");
            task.setDescription("Formats the lines you changed");
        });
        executor.submit(() -> {
            runChecks();
        });
    }
}
```

A short lambda stays on the line of its call, also inside a chain.

``` java
class Lambda {
    private static GradleException notFound(String group, String name, Configuration configuration) {
        String actual = configuration.getIncoming().getResolutionResult().getAllComponents().stream()
                .map(ResolvedComponentResult::getModuleVersion)
                .map(mvi -> String.format("\t- %s:%s:%s", mvi.getGroup(), mvi.getName(), mvi.getVersion()))
                .collect(Collectors.joining("\n"));
        return new GradleException(actual);
    }
}
```

The [home page](index.md#what-the-output-looks-like) shows the same kind of code next to the output
of google-java-format.

## Call chains

A chain of calls stays on one line only if everything before its last dot fits in 80 columns, even
when the whole statement would fit in 120. Otherwise each call goes on a line of its own, which keeps
builders and streams easy to read and to diff.

``` java
class Chains {
    void f() {
        var request = HttpRequest.newBuilder()
                .uri(uri)
                .header("Accept", "application/json")
                .timeout(timeout)
                .build();
        var user = User.builder().name(name).email(email).build();
    }
}
```

On one line the first statement would be 118 characters long, with its last dot in column 110. The
last dot of the second one is in column 58.

When moving the whole chain onto the next line brings its last dot within the limit, the formatter
does that instead of splitting it.

``` java
class Chains {
    void f() {
        var foo =
                SomeType.builder().thing1(thing1).thing2(thing2).thing3(thing3).build();
    }
}
```

## Long strings

A string literal that runs past column 120 is split between words, and the rest continues after a
`+` on the next line. On the command line, `--skip-reflowing-long-strings` turns this off.

``` java
class Strings {
    String message =
            "The formatter reflows a string literal that runs past the column limit, and it keeps the words intact"
                    + " while doing so.";
}
```

## Imports

Static imports come first, then a blank line and the other imports, each group in ASCII order.
Imports the file does not use are removed: the input of this example also imported `java.util.Map`
and `java.util.Set`. On the command line, `--skip-sorting-imports` and
`--skip-removing-unused-imports` turn these off.

``` java
package com.example;

import static java.util.Objects.requireNonNull;

import com.google.common.collect.ImmutableList;
import java.util.List;

class Imports {
    List<String> names = ImmutableList.of(requireNonNull("a"));
}
```

## Comments

A `//` comment that runs past column 120 is wrapped onto a new `//` line. Javadoc and `/* */`
comments are kept exactly as written, however long their lines are.

``` java
class Comments {
    // A line comment that runs past the limit of one hundred and twenty characters is wrapped, and the rest continues
    // on a line of its own.
    int x;

    /** Javadoc is kept exactly as it is written, however long its lines are, because the formatter does not reflow it. */
    int y;
}
```

## Trade-offs

- **The layout follows the syntax.** The formatter does not know which grouping of arguments reads
  best. When a layout comes out awkward, a local variable or a small method usually fixes it, and the
  fix holds on every later run.
- **A `$NON-NLS$` marker can end up on another line.** Eclipse expects the marker on the line of the
  string it marks. When the formatter wraps such a statement, the string moves to a line of its own
  and the marker stays at the end of the statement:

    ``` java
    class Messages {
        void f() {
            label.setText(Messages.format(
                    "The configuration of the project could not be read",
                    projectName,
                    configurationFileName)); // $NON-NLS-1$
        }
    }
    ```

- **Layout changes of our own come with a new minor version.** The output is the same as
  palantir-java-format 2.98.0's, except where it fixes a bug, such as a JBang script whose
  directives the formatter broke. [Migrate](migrate.md#from-palantir-java-format) lists those
  fixes, [Versions](about.md#versions) says which number moves for what, and the
  [manifesto](manifesto.md) explains why output stability comes first.
