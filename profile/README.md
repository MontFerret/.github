# Ferret

**Declarative data automation. Extract with intent.**

Ferret is an open-source, declarative-first, expression-oriented language with an embeddable runtime for querying and transforming structured data—whether it lives in your application, on the web, behind an API, in a database, or in a file.

```fql
let users = query "users" in db

return for user in users {
    filter user.active

    return {
        name: user.name,
        email: user.email
    }
}
```

Ferret started as a language for web scraping and browser automation. It has since grown into a general-purpose data automation language built around the same idea: **describe the data you want and let the runtime handle how to get it.**

Use Ferret from the command line, embed it into a Go or JavaScript application, extend it with modules, or run it as part of a larger service.

## Get started

- [Documentation](https://ferretlang.org/docs/)
- [Getting Started](https://ferretlang.org/docs/getting-started/)
- [Ferret v2](https://ferretlang.org/blog/ferret-v2-language-evolution/)
- [Module Registry](https://ferretlang.org/registry/)

## The ecosystem

Ferret is more than the language repository. The project includes the runtime, developer tools, modules, services, and infrastructure around it.

### Language & runtime

**[Ferret](https://github.com/MontFerret/ferret)**  
The language, compiler, VM, standard library, and embeddable Go runtime.

**[@montferret/ferret](https://www.npmjs.com/package/@montferret/ferret)**  
Ferret for JavaScript and TypeScript, powered by WebAssembly.

### Developer tools

**[CLI](https://github.com/MontFerret/cli)**  
Run, format, inspect, and work with Ferret programs from the command line.

**[Lab](https://github.com/MontFerret/lab)**  
A test runner for Ferret programs and automation scenarios.

**[Daemon](https://github.com/MontFerret/ferretd)**  
A long-running Ferret service providing workspaces, language intelligence, execution sessions, and the foundation for IDE and remote tooling.

**[VS Code](https://github.com/MontFerret/vscode-fql-syntax)**  
Ferret language support for Visual Studio Code.

### Modules & integrations

**[Contrib](https://github.com/MontFerret/contrib)**  
Official Ferret modules and integrations for databases, documents, APIs, browser automation, and other external systems.

**[Barn](https://github.com/MontFerret/barn)**  
The source registry behind the Ferret module ecosystem.

**[Registry](https://ferretlang.org/registry/)**  
Discover modules and integrations for Ferret.

### Runtime infrastructure

**[Worker](https://github.com/MontFerret/worker)**  
Run Ferret programs remotely over HTTP.

**[Chromium](https://github.com/MontFerret/chromium)**  
A containerized Chromium environment for Ferret browser automation.

### Specifications

**[Specs](https://github.com/MontFerret/specs)**  
Shared schemas, specifications, and validation rules used across the Ferret ecosystem.

## Why Ferret?

Ferret sits somewhere between a query language, an automation language, and an embeddable data runtime.

It keeps data operations declarative:

```fql
return for user in users {
    filter user.age >= 18
    sort user.name

    return user
}
```

while still providing the language features needed when automation gets more involved:

```fql
func classify(response) {
    return match response.status {
        200 => "ok"
        404 => "missing"
        _ => "error"
    }
}
```

The runtime can also work with capabilities supplied by its host, allowing Ferret programs to query databases, APIs, documents, browsers, application objects, and other systems without baking all of those integrations into the language itself.

That makes the same language useful for small scripts, application embedding, testing, data extraction, and larger automation systems.

## Project status

Ferret v2 is currently under active development.

v2 expands Ferret beyond its AQL-inspired roots with a more complete language, a redesigned runtime and embedding model, modules and registry infrastructure, improved developer tooling, and support for additional host environments.

Follow the [Ferret repository](https://github.com/MontFerret/ferret) and [ferretlang.org](https://ferretlang.org/) for releases and project updates.

## Contributing

Ferret is open source and contributions are welcome across the language, runtime, tooling, modules, documentation, and ecosystem projects.

Start with the repository that interests you, or explore the [documentation](https://ferretlang.org/docs/) to get familiar with the project.

## Contributors

Thanks to everyone who has helped build Ferret.

<p>
    <a href="https://github.com/MontFerret/ferret/graphs/contributors">
        <img src="https://contrib.rocks/image?repo=MontFerret/ferret" />
    </a>
</p>

## Support Ferret

If Ferret is useful to you and you'd like to support its development, you can sponsor the project through Open Collective.

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