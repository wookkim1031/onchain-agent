# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```sh
npm install       # Install dependencies
npm run dev       # Start development server (http://localhost:3000)
npm run build     # Production build
npm run lint      # Run ESLint
```

There are no tests configured in this project.

## Required Environment Variables

Set these in `.env`:

| Variable | Source |
|---|---|
| `OPENAI_API_KEY` | [OpenAI Platform](https://platform.openai.com/api-keys) |
| `PRIVATE_KEY` | Ethereum wallet private key (hex) |
| `NETWORK_ID` | e.g. `base-sepolia` or `base-mainnet` |
| `CDP_API_KEY_ID` | [CDP Portal](https://portal.cdp.coinbase.com/) (optional) |
| `CDP_API_KEY_SECRET` | CDP Portal (optional) |

If `PRIVATE_KEY` is absent, the app auto-generates one and writes it to `wallet_data.txt` at runtime.

## Architecture

This is a Next.js App Router project. The core flow is:

```
app/page.tsx  (chat UI)
  └─ app/hooks/useAgent.ts  (POST /api/agent, manages message state)
       └─ app/api/agent/route.ts  (Next.js API route)
            └─ app/api/agent/create-agent.ts  (singleton ReAct agent)
                 └─ app/api/agent/prepare-agentkit.ts  (wallet + action providers)
```

**Agent initialization** (`create-agent.ts`): A singleton LangGraph `createReactAgent` is created on first request using `gpt-4o-mini`. It holds in-memory conversation history via `MemorySaver` with thread ID `"AgentKit Discussion"` — all users share one thread.

**AgentKit setup** (`prepare-agentkit.ts`): Uses `ViemWalletProvider` backed by the configured `PRIVATE_KEY`. Default action providers are `wethActionProvider`, `pythActionProvider`, `walletActionProvider`, and `erc20ActionProvider`. `cdpApiActionProvider` is added only when both CDP env vars are present.

**Adding capabilities**: New onchain actions are added by importing and appending additional `ActionProvider` instances to the `actionProviders` array in `prepare-agentkit.ts`. Available providers: https://github.com/coinbase/agentkit/tree/main/typescript/agentkit#action-providers

**API contract** (`app/types/api.ts`): `AgentRequest = { userMessage: string }`, `AgentResponse = { response?: string; error?: string }`.
