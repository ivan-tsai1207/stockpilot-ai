# PR Description

## Summary

This PR expands the StockPilot AI system specification from a high-level analysis concept into a more implementation-ready architecture for LLM-powered stock analysis.

The focus of this change is:

- isolated stock-specific LLM sessions
- structured memory instead of a single long chat
- versioned user strategy profiles
- multi-user data isolation
- backend service boundaries and database schema

## Why

The product needs to support:

- analyzing different stocks independently
- avoiding context pollution between symbols
- reducing token waste from long shared sessions
- generating advice that matches each user's investing style
- preparing the repo so future engineers and LLMs understand the intended system behavior without repeated explanation

## Main changes

- expanded `docs/SDD.md` with:
  - LLM architecture
  - session isolation rules
  - memory layers
  - strategy profile design
  - multi-user boundaries
  - backend architecture
  - database schema
  - analysis flow sequences
- expanded `docs/api-contract.md` with:
  - strategy profile endpoints
  - stock session endpoints
  - session memory endpoint
  - stock analysis endpoint with strategy and session binding
- updated `docs/roadmap.md` to reflect:
  - session layer work
  - memory summarization
  - strategy management
- added `AGENTS.md` so future LLM collaborators can understand repository goals quickly

## Notes

- This branch does not modify `main`
- This branch is intended for review first
