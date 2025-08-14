# Documentation

This directory contains comprehensive documentation for the `@wellcare/nestjs-elastic-apm` module.

## 📚 Documentation Files

### [`database.md`](./database.md)
Database schema documentation using DBML (Database Markup Language). This file describes the database structure, tables, relationships, and indexes used by the Elastic APM system.

### [`sequence.md`](./sequence.md)
System workflow documentation using Mermaid.js sequence diagrams. This file illustrates the interaction flows between the NestJS application, Elastic APM agent, and APM server, including error handling and performance monitoring workflows.

### [`api.md`](./api.md)
API documentation for the module's public interfaces. This file details the available methods, parameters, return types, and usage examples for the `ApmService` and related components.

### [`state-machine.md`](./state-machine.md)
State machine documentation using Mermaid.js state diagrams. This file describes the lifecycle states and transitions for APM transactions, spans, and error handling processes.

## 💡 Diagram Rendering

Some diagrams use [Mermaid.js](https://mermaid.js.org/) and [DBML](https://dbdiagram.io/docs). Open these files in **Cursor**, **Obsidian**, or **VS Code** with the appropriate plugins to render them.

### Recommended Extensions
- **VS Code**: Mermaid Preview, DBML Preview
- **Cursor**: Built-in Mermaid support
- **Obsidian**: Built-in Mermaid support

### Online Viewers
- **Mermaid.js**: [Mermaid Live Editor](https://mermaid.live/)
- **DBML**: [dbdiagram.io](https://dbdiagram.io/)

## 🔗 Related Links

- [Main README](../README.md) - Project overview and quick start guide
- [Elastic APM Documentation](https://www.elastic.co/guide/en/apm/agent/nodejs/current/index.html)
- [NestJS Documentation](https://nestjs.com/)
