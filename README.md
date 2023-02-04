# Jenkins 2 diagrams

This script converts a folder of Jenkins files into a visual representation.

Whilst there are [Jenkins plugins to view pipeline graphs](https://plugins.jenkins.io/pipeline-graph-view/), you have to
enter a pipeline first.
If you have many pipelines and don't know the top-level/entrypoint pipelines, you might like this tool.
Visualise in the terminal, as textual diagrams, or render with your IDE/GitHub/Docusaurus etc.

# Inputs

Specify a path to your project.
`Jenkinsfile.*` and any files ending with `.jenkinsfile` will be scanned

# Options

With `.jenkinsfile`s, the file isn't enough to state whether they are used as a top-level pipeline in Jenkins (vs being
called by a higher-level pipeline).

To create a more accurate visual representation of your pipelines, add this line to the top of your Jenkins files which
are called directly from the Jenkins UI:

[//]: # (TODO: if there are no lines with build(job: `jobname`))... we have toplevel
[//]: # (TODO: might be called directly (toplevel) and/or by another job - best users annotate pipelines)

```groovy
// jenkins2diagram:toplevel
```

# Supported outputs

Here's a few options - I'm considering:

- Mermaid diagram
    - Renders in GitHub .md previews
    - Renders in Docusaurus (a static website tool)
- ~Ascii art~
    - Renders anywhere
- ~Image files~

I'll focus on mermaid first.
I think the app will return text to stdout, which can be piped `>` by the user into a file if they wish.

# Developing a POC (1/2)

This app is quite domain / keyword specific, so for this POC I aim to:

- [x] Get the basic file reading, pipeline mappings, etc. correct with simplified syntax:

```
# Jenkinsfile 
// jenkins2diagram:toplevel
build job: 'a'
build job: 'b'

// a.jenkinsfile

// b.jenkinsfile
build job: 'c'

// c.jenkinsfile
```

Should produce:

```mermaid
graph TD
    Z[Jenkinsfile]
    Z[Jenkinsfile] --> A[a]
    Z[Jenkinsfile] --> B[b]
    B[b] --> C[c]
```

## Notes on GraphViz DOT and Mermaid

Note: To kickstart generating the tree, I used [anytree](https://github.com/c0fec0de/anytree).
It can export to a Python dictionary, JSON, or "Dot" format.
The DOT language is a grammar for [Graphviz](https://graphviz.org/doc/info/lang.html).

- IntelliJ
    - recognised Mermaid in my code block, offered to install the plugin, and rendered correctly. Mermaid plays
      nice with GitHub and Docusaurus, as is already mentioned
    - making graph.dot let the IDE suggest a plugin install. A sample graph renders nicely, and copy to clipboard/save
      to file work great
- GitHub markdown rendering
    - [Including graphviz graphs is a little indirect](https://github.com/TLmaK0/gravizo) (some html tags and custom
      marks seem to be necessary)
        -
      Apparently [Graphviz is in C, and Mermaid is written in JS](https://forum.graphviz.org/t/github-adding-support-for-mermaid-diagrams/998)
        - and mermaid is a little prettier
- Docusaurus
    - I only see documentation on Mermaid support - nothing for Graphviz
- Converting Graphviz DOT to Mermaid
    - Kroki: A server that renders a bunch of different textual diagrams to images

Overall, I should probably aim to add a Mermaid exporter to `anytree`. There's no issues/PRs for it so far.
`anytree`'s `dotexporter.py` is ~ 400 lines, but more than half of it is examples.
I'll try a basic version in this POC first.

The mermaid tree generation is working nicely.
Seeing that anytree can import/export trees as Python dictionaries helped me see I can move to a minimal dependency
version, hopefully without too much challenge. That can happen later.
I will try to raise a basic PR for a Mermaid exporter now.

# Developing a POC (2/2)

- [ ] Change to support more proper syntax
- [ ] Be more flexible with matches (`build(job: '...',...)` vs `build job:'...'`, etc.))