# ADR-0005: Provider-agnostic LLM gateway

- Status: Proposed
- Date: 2026-10-15
- Deciders: product owner (@amandavictoriacouto)
- Related: [MVP architecture](../architecture/mvp-architecture.md), [threat model](../security/threat-model.md)

## Context

The product must not depend on a single LLM provider, and every AI output needs provenance (model, prompt version, input hash) and evaluation.

## Decision

Define an internal `LLMProvider` protocol with adapters (Anthropic and OpenAI first), structured outputs validated by Pydantic, token budgets and a provenance table.

## Alternatives considered

- LiteLLM: broad provider coverage, but less control over provenance hooks.
- LangChain: rich ecosystem, large dependency surface.

## Consequences

- **Positive:** Small, testable surface; evals and provenance built in.
- **Negative:** Each new provider needs an adapter; LiteLLM can be wrapped later.
