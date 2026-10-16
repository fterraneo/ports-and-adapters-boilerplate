# Ports and Adapters Boilerplate

TypeScript starter baseline for a Ports & Adapters (Hexagonal) architecture, developed with AI agents.
It comes with Jest, ESLint, Prettier and strict TypeScript already configured.

## Origin

This repository is a fork of [nodejs-boilerplate](https://github.com/fterraneo/nodejs-boilerplate),
on the same GitHub account. It keeps the same tooling and adds guidelines for hexagonal
architecture, outside-in TDD and agent-driven development.

## Principles

- Hexagonal architecture: the core has no infrastructure dependencies, adapters implement ports at the edge.
- Outside-in TDD: acceptance tests use in-memory adapters, unit tests are colocated as `*.test.ts`.
- Strict TypeScript, with no `any`.
- Small, atomic commits following Conventional Commits.

The full rules are in [AGENTS.md](./AGENTS.md).

## Recommended setup

Let your IDE pick up the automatic configuration from `.editorconfig`, `.prettierrc` and `eslint.config.js`.

## Commands

| Command                       | Purpose                       |
| ----------------------------- | ----------------------------- |
| `npm test` / `npm run test:w` | Run tests (or watch mode)     |
| `npm run tc` / `npm run tc:w` | Typecheck (or watch mode)     |
| `npm run build`               | Clean and compile to `.build` |
| `npm start`                   | Run the compiled app          |
| `npx eslint .`                | Lint                          |
| `npx prettier .`              | Format check                  |
