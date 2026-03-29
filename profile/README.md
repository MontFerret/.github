# Welcome to Ferret

Ferret is an open-source web data extraction system built around its own declarative language, runtime, and tooling.

Instead of stitching together browser scripts, selectors, waits, and parsing logic by hand, Ferret lets you describe the data you want and the interactions needed to get it. This makes scraping, browser automation, UI testing, and structured data collection easier to write, easier to maintain, and easier to scale.

Ferret can be used as:
- a query language for extracting and transforming web data
- an embeddable runtime for integrating automation into applications
- a set of tools for local development, testing, and distributed execution

Whether you are prototyping a one-off script or building a production workflow, Ferret is designed to give you a higher-level way to work with the web.

## Get started

If you're new to Ferret, start here:

- Read the [introduction post](https://www.montferret.dev/blog/say-hello-to-ferret/)
- Follow the [getting started guide](https://www.montferret.dev/docs/getting-started/)
- Try Ferret online in the [Playground](https://www.montferret.dev/try/)

## Ecosystem

Ferret includes a growing ecosystem of tools and components:

- [Ferret runtime](https://github.com/MontFerret/ferret) - the core language, compiler, and execution engine. It is designed to be portable, embeddable, and suitable for both standalone use and integration into larger systems.
- [Contrib](https://github.com/MontFerret/contrib) - a collection of official extensions, integrations, and optional components built around the Ferret runtime. This is where ecosystem modules such as browser drivers and other add-ons can live without making the core heavier.
- [CLI](https://github.com/MontFerret/cli) - a command-line interface for formatting, running, and working with Ferret scripts locally.
- [Worker](https://github.com/MontFerret/worker) - an HTTP service for executing Ferret workloads remotely, useful for scalable and distributed deployments.
- [Lab](https://github.com/MontFerret/lab) - a test runner for building UI and workflow tests with Ferret scripts, including CI-friendly automation scenarios.
- [Chromium](https://github.com/MontFerret/chromium) - a Dockerized headless Chromium image tailored for Ferret-based browser execution.
- [VS Code Syntax Highlighting](https://github.com/MontFerret/vscode-fql-syntax) - syntax highlighting support for the Ferret Query Language in Visual Studio Code.
- [Ferret Playground](https://www.montferret.dev/try/) - an interactive environment for experimenting with Ferret scripts in the browser.

## Learn more

Visit the [documentation](https://www.montferret.dev/docs/introduction/) to learn more about the language, runtime, and tools.

## Contributors
Thanks to everyone who contributed.
<p>
    <a href="https://github.com/MontFerret/ferret/graphs/contributors">
        <img src="https://contrib.rocks/image?repo=MontFerret/ferret" />
    </a>
</p>

## Financial support
Support this project by becoming a sponsor. Your logo will show up here with a link to your website.

<p>
	<img src="https://opencollective.com/ferret/sponsors.svg?width=890&button=false" />
</p>
<p>
	<img src="https://opencollective.com/ferret/backers.svg?width=890&button=false" />
</p>

<p align="center">
    <a href="https://opencollective.com/ferret/donate" target="_blank">
        <img src="https://opencollective.com/ferret/donate/button@2x.png?color=blue" width="300" />
    </a>
</p>
