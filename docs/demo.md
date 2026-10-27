# Demo

<div><pre><code id="source" onfocusout="highlightSource();">
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
</code></pre></div>

<input type="button" value="Format!" id="format" class="md-button md-button--primary"/>

<div>
<pre id="formatted_source_time"></pre>
<pre><code id="formatted_source" class="language-java"></code></pre>
</div>

<script src="./assets/js/open-java-format.js"></script>

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.12.0/styles/default.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.12.0/highlight.min.js"></script>

<script>
    hljs.configure({
      languages: ['java'],
      ignoreUnescapedHTML: true
    });
    hljs.highlightAll();

    function highlightCode() {
        highlightCodeImpl(document.getElementById('formatted_source'));
    }

    function highlightSource() {
        highlightCodeImpl(document.getElementById('source'));
    }

    function highlightCodeImpl(codeElement) {
        codeElement.removeAttribute('data-highlighted');
        hljs.highlightElement(codeElement);
    }
</script>