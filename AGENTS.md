# StockPilot AI Repo Guide

This file is for coding agents and LLM collaborators entering this repository.

## Product intent

StockPilot AI is not a stock quote website.

It is an AI Investment OS for Taiwan stock investors. The product should help users move from information gathering to structured decision-making.

The three core product experiences are:

1. Research Workspace
2. Portfolio Command Center
3. Intraday AI Chat

## What the system must do

- Analyze one stock at a time with isolated session memory
- Support multiple users with different investing styles
- Let each user configure strategy profiles
- Use LLMs to explain current status and future scenarios
- Keep outputs grounded in evidence, not certainty

## What the system must not do

- Do not present guaranteed predictions
- Do not let different stocks share the same live analysis context
- Do not store all chats in one giant session
- Do not ignore risk control or stop-loss logic
- Do not produce a buy or sell answer without rationale

## Core analysis rule

All important outputs should follow:

Evidence -> Inference -> Strategy -> Risk

## Session design rule

The primary conversation unit is:

`User + Strategy Profile + Symbol + Market + Timeframe`

Different stocks should have different sessions by default.

## Memory design rule

Prefer structured memory and compressed summaries over replaying the entire conversation history.

Required memory layers:

- Raw conversation log
- Session summary memory
- Structured fact memory
- Strategy output snapshot

## Strategy design rule

User strategy is configurable and versioned.

The system must separate:

- System Rulebook
- User Strategy Profile

System rules are global and non-negotiable.
User strategies personalize the recommendation style and risk preference.

## Multi-user rule

Anything user-specific must be isolated by `user_id`.

This includes:

- sessions
- memories
- strategy profiles
- holdings
- alerts

## Repo priorities

When updating this repo, prioritize these files:

- `docs/SDD.md`
- `docs/api-contract.md`
- `docs/roadmap.md`
- `README.md`
- `index.html`

## Git workflow

Do not work directly on `main`.

Expected workflow:

1. create a branch
2. make scoped changes
3. commit clearly
4. push the branch
5. let the user decide when to merge or deploy

## If you need more context

Read these in order:

1. `AGENTS.md`
2. `README.md`
3. `docs/SDD.md`
4. `docs/api-contract.md`
5. `docs/roadmap.md`
