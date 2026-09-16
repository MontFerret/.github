# Ferret

**Declarative data automation. Extract with intent.**

Ferret is an open-source, declarative-first, expression-oriented language and embeddable runtime for querying, transforming, and automating structured data — whether it lives in your application, on the web, behind an API, in a database, or in a file.

[Website](https://ferretlang.org/) · [Documentation](https://ferretlang.org/docs/) · [Getting Started](https://ferretlang.org/docs/getting-started/) · [Module Registry](https://ferretlang.org/registry/)

```fql
let page = web::html::open(
    "https://mockery.ferretlang.org/scenarios/ecommerce/products/"
)

let products = page[~ css`.product-card`]

return for product in products
    return {
        name: product[~ css`.product-title`],
        price: product[~ css`.product-price`],
        url: product[~ css`:attr("href", .product-link)`]
    }
```

Ferret keeps data logic in FQL while the host decides which capabilities are available. Run scripts locally, embed the runtime into an application, extend it with modules, or expose a configured runtime to external tooling.

## Run Ferret where the logic needs to live

| Mode | Use it for |
| --- | --- |
| **CLI** | Local scripts, automation, CI, formatting, inspection, debugging, and module workflows. |
| **Go** | Embed the native Ferret runtime and expose application data, functions, resources, and capabilities directly to FQL. |
| **JavaScript / WebAssembly** | Run Ferret from Node.js or modern browsers through the official WASM runtime. |
| **Remote runtimes** | Execute and debug against configured runtimes through the Universal API and Wire. |

Ferret is designed to complement general-purpose languages rather than replace them. The host application owns integrations, resources, and the execution boundary; FQL owns portable data querying, transformation, synchronization, and automation logic.

## The ecosystem

| Area | Projects |
| --- | --- |
| **Language & runtimes** | **[Ferret](https://github.com/MontFerret/ferret)** — language, compiler, VM, standard library, and embeddable Go runtime.<br>**[ferret-js](https://github.com/MontFerret/ferret-js)** — Ferret runtime for JavaScript, browsers, and Node.js, powered by WebAssembly. |
| **Developer tools** | **[CLI](https://github.com/MontFerret/cli)** — command-line workflow for FQL.<br>**[Lab](https://github.com/MontFerret/lab)** — test runner for Ferret programs and automation scenarios.<br>**[ferretd](https://github.com/MontFerret/ferretd)** — developer service for language intelligence, execution, debugging, module resolution, and runtime inspection.<br>**[Editorium](https://github.com/MontFerret/editorium)** — official editor integrations for Ferret. |
| **Runtime integration** | **[Universal API](https://github.com/MontFerret/api)** — common contracts for runtimes, compiled plans, sessions, output, diagnostics, and debugging.<br>**[Wire](https://github.com/MontFerret/wire)** — remote runtime protocol and Go SDK for executing and debugging Ferret across a transport boundary.<br>**[Worker](https://github.com/MontFerret/worker)** — containerized Ferret runtime for remote execution. |
| **Modules & ecosystem** | **[Contrib](https://github.com/MontFerret/contrib)** — official optional modules and integrations.<br>**[Registry](https://ferretlang.org/registry/)** — discover modules and the documentation published with their releases.<br>**[Barn](https://github.com/MontFerret/barn)** — registry infrastructure behind the Ferret module ecosystem. |

## A small language with a host-controlled runtime

Ferret started with web extraction and browser automation, but the core model is broader: FQL operates on values and capabilities supplied by its environment.

That separation lets the same language work with application objects, documents, APIs, databases, browser-backed pages, files, and custom resources without turning the language itself into a collection of hard-coded integrations.

Ferret is especially useful when data logic should be **portable, reviewable, testable, and able to evolve independently from the host application**.

## Ferret v2

Ferret v2 is currently in active alpha development.

v2 expands Ferret with a more complete language, a redesigned runtime and embedding model, reusable compiled plans and isolated sessions, modules and registry infrastructure, modern debugging support, and additional host environments.

New users should start with v2. Alpha releases are intended for experimentation, feedback, prototypes, internal tools, and early integrations; language and API details may still change before the stable release.

[Read the documentation](https://ferretlang.org/docs/) · [Run your first script](https://ferretlang.org/docs/getting-started/quick-start/) · [Follow Ferret releases](https://github.com/MontFerret/ferret/releases)

## Contributing

Ferret is open source, and contributions are welcome across the language, runtime, tooling, modules, documentation, and ecosystem projects.

Start with the repository that interests you, join the conversation in [GitHub Discussions](https://github.com/MontFerret/ferret/discussions), or visit the [contact page](https://ferretlang.org/contact/) for help, bug-reporting guidance, security contacts, and other ways to get involved.

## Contributors

Thanks to everyone who has helped build Ferret.

<p>
    <a href="https://github.com/MontFerret/ferret/graphs/contributors">
        <img src="https://contrib.rocks/image?repo=MontFerret/ferret" alt="Ferret contributors" />
    </a>
</p>
