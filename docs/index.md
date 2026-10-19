---
title: The first guardrail for AI-written Java
hide:
  - navigation
  - toc
---

# The first guardrail for AI-written Java

**open-java-format** is a deterministic formatter for Java. There is one style and nothing to tune,
so every file comes out the same way, whether a person or a model wrote it.

[Get started](get-started/index.md){ .md-button .md-button--primary }
[View on GitHub](https://github.com/openjavaformat/open-java-format){ .md-button }

## Why formatting comes first

<div class="grid cards" markdown>

-   :lucide-equal:{ .lg .middle } __One style, nothing to tune__

    ---

    The output depends only on the input. There is no line-length setting, no indentation setting
    and no per-team dialect for an agent to get wrong. The IDE, the command line and CI all produce
    the same bytes.

-   :lucide-git-pull-request-arrow:{ .lg .middle } __Diffs a reviewer can read__

    ---

    Models write a lot of code quickly. When all of it is laid out the same way, a diff shows what
    changed in the logic, not how a particular model likes to wrap its lines.

-   :lucide-zap:{ .lg .middle } __Fast enough to run after every edit__

    ---

    The formatter ships as a native binary with no JVM to start, so it fits into an
    [agent hook](ai-agents.md), a pre-commit hook or a CI step. It also comes as a Gradle plugin
    and as plugins for IntelliJ IDEA and Eclipse.

</div>

A formatter does not find bugs. It is the first check in the chain, ahead of the compiler, static
analysis, tests and review, and it is the cheapest one to automate.

## What the output looks like

Lines are up to 120 characters wide. A lambda stays on the line where it starts, and a long call
chain breaks into one call per line.

=== "open-java-format"

    ``` java
    private static void configureResolvedVersionsWithVersionMapping(Project project) {
        project.getPluginManager().withPlugin("maven-publish", plugin -> {
            project.getExtensions()
                    .getByType(PublishingExtension.class)
                    .getPublications()
                    .withType(MavenPublication.class)
                    .configureEach(publication -> publication.versionMapping(mapping -> {
                        mapping.allVariants(VariantVersionMappingStrategy::fromResolutionResult);
                    }));
        });
    }
    ```

=== "google-java-format"

    ``` java
    private static void configureResolvedVersionsWithVersionMapping(Project project) {
        project.getPluginManager()
                .withPlugin(
                        "maven-publish",
                        plugin -> {
                            project.getExtensions()
                                    .getByType(PublishingExtension.class)
                                    .getPublications()
                                    .withType(MavenPublication.class)
                                    .configureEach(
                                            publication ->
                                                    publication.versionMapping(
                                                            mapping -> {
                                                                mapping.allVariants(
                                                                        VariantVersionMappingStrategy
                                                                                ::fromResolutionResult);
                                                            }));
                        });
    }
    ```

## Quick start

### In a Gradle build

=== "Groovy"

    ``` groovy title="build.gradle"
    plugins {
        id 'dev.openjavaformat.java-format' version '{{ ojf_version }}'
    }
    ```

=== "Kotlin"

    ``` kotlin title="build.gradle.kts"
    plugins {
        id("dev.openjavaformat.java-format") version "{{ ojf_version }}"
    }
    ```

``` properties title="gradle.properties"
openjavaformat.native.formatter=true
```

The plugin adds the `formatDiff` task, which formats only the lines you changed in git, and it keeps
IntelliJ IDEA on the formatter version of the build. The property makes Gradle run the formatter as
a native binary on Linux, macOS and Windows. For other platforms and for multi-project builds see
[Gradle plugin](get-started/gradle.md).

### In a Maven build

``` xml title="pom.xml"
<build>
    <plugins>
        <plugin>
            <groupId>dev.openjavaformat</groupId>
            <artifactId>fmt-maven-plugin</artifactId>
            <version>{{ maven_plugin_version }}</version>
            <executions>
                <execution>
                    <goals>
                        <goal>format</goal>
                    </goals>
                </execution>
            </executions>
            <dependencies>
                <dependency>
                    <groupId>dev.openjavaformat</groupId>
                    <artifactId>open-java-format</artifactId>
                    <version>{{ ojf_version }}</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

The `format` goal formats `src/main/java` and `src/test/java` in the `process-sources` phase, so the
build formats the sources before it compiles them. The dependency picks the formatter version, and
the plugin does not run without it. Maven has to run on Java 21 or later. The `check` goal for CI is
on the [Maven plugin](get-started/maven.md) page.

### On the command line

Every [release](https://github.com/openjavaformat/open-java-format/releases/latest) carries native
binaries for Linux, macOS and Windows, and a runnable jar for Java 21 or later.

``` sh title="Format files in place"
open-java-format --replace src/main/java/com/example/Hello.java
```

``` sh title="Fail when anything is not formatted"
open-java-format --dry-run --set-exit-if-changed $(git ls-files '*.java')
```

Downloads, checksums and every option are on the [Command line](get-started/command-line.md) page.

### In CI with GitHub Actions

``` text title=".open-java-format-version"
{{ ojf_version }}
```

``` yaml title=".github/workflows/format.yml"
- uses: actions/checkout@v7

- uses: openjavaformat/open-java-format-action@v4
```

The [action](https://github.com/openjavaformat/open-java-format-action) downloads the native binary
and fails the job when a changed file is not formatted. It needs no Java on the runner, and takes
the formatter version from `.open-java-format-version` in the repository root. The
[GitHub Action and pre-commit](get-started/github-actions.md) page has the full workflow and a git
hook that reads the same file and runs the same check.

## A drop-in for palantir-java-format

The output is the same as palantir-java-format 2.98.0's, except for bug fixes, and the Java packages
are unchanged. Migrating means changing the coordinates and nothing else: [Migrate](migrate.md) lists
every name that changes and every fix, and any later change to the output comes with a new minor
version, see [Versions](about.md#versions).
