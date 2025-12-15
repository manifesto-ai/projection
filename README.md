# @manifesto-ai/projection

[![CI](https://github.com/manifesto-ai/projection/actions/workflows/ci.yml/badge.svg)](https://github.com/manifesto-ai/projection/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> Projection layers for Manifesto AI — Transform domain models into consumer-friendly representations

## Overview

This monorepo contains **projection packages** that transform [Manifesto Core](https://github.com/manifesto-ai/core)'s domain models and snapshots into various consumer formats. Projections are **read-only transformation layers** that bridge the gap between your domain logic and consumers like UIs, APIs, and AI agents.

```
┌─────────────────────────────────────────────────────────────┐
│                       Consumers                             │
│           (React, Vue, GraphQL, AI Agents, etc.)            │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                @manifesto-ai/projection-*                   │
│                                                             │
│   ┌────────────────┐ ┌────────────────┐ ┌────────────────┐  │
│   │ projection-ui  │ │projection-gql  │ │projection-agent│  │
│   └────────────────┘ └────────────────┘ └────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ (read-only)
┌─────────────────────────────────────────────────────────────┐
│                    @manifesto-ai/core                       │
│               (Domain Models, Runtime, Snapshots)           │
└─────────────────────────────────────────────────────────────┘
```

## Packages

| Package | Description | npm |
|---------|-------------|-----|
| [`@manifesto-ai/projection-ui`](./packages/projection-ui) | Framework-agnostic UI state projection | [![npm](https://img.shields.io/npm/v/@manifesto-ai/projection-ui)](https://www.npmjs.com/package/@manifesto-ai/projection-ui) |
| [`@manifesto-ai/projection-graphql`](./packages/projection-graphql) | GraphQL schema and resolver generation | [![npm](https://img.shields.io/npm/v/@manifesto-ai/projection-graphql)](https://www.npmjs.com/package/@manifesto-ai/projection-graphql) |
| [`@manifesto-ai/projection-agent`](./packages/projection-agent) | AI-friendly context projection for agents | [![npm](https://img.shields.io/npm/v/@manifesto-ai/projection-agent)](https://www.npmjs.com/package/@manifesto-ai/projection-agent) |

## Key Principles

### 1. Read-Only Transformations

Projections **never mutate state**. They only transform snapshots into consumer-friendly formats.

```typescript
// ✅ Projections do this
const uiState = projectFieldPolicy(policy, runtime);
const schema = generateGraphQLSchema(domain);
const context = projectAgentContext(runtime);

// ❌ Projections never do this
runtime.set('data.field', value);  // No mutations
await executeEffect(effect);        // No side effects
```

### 2. One-Way Dependencies

```
projection-* → @manifesto-ai/core
```

- Projection packages depend **only** on `@manifesto-ai/core`
- No cross-dependencies between projection packages
- Core knows nothing about projections

### 3. Framework Agnostic

Projection packages provide pure transformation logic. Framework-specific bindings (React hooks, Vue composables) belong in bridge packages or your application code.

## Installation

```bash
# Install individual packages as needed
pnpm add @manifesto-ai/projection-ui
pnpm add @manifesto-ai/projection-graphql
pnpm add @manifesto-ai/projection-agent

# Core is a peer dependency
pnpm add @manifesto-ai/core
```

## Quick Examples

### UI Projection

Transform domain policies into UI states:

```typescript
import { projectFieldPolicy, createProjectionManager } from '@manifesto-ai/projection-ui';

// Project a single field
const state = projectFieldPolicy(fieldPolicy, runtime);
// → { visible: true, enabled: false, required: true, validation: {...} }

// Or use the manager for multiple fields
const manager = createProjectionManager({
  runtime,
  domain,
  fields: { paths: ['data.name', 'data.email'] },
});

manager.subscribeFields((states, changedPaths) => {
  // React to field state changes
});
```

### GraphQL Projection

Generate GraphQL schemas from domain definitions:

```typescript
import { generateGraphQLSchema, createResolvers } from '@manifesto-ai/projection-graphql';

const schema = generateGraphQLSchema(domain);
const resolvers = createResolvers(domain, runtime);

// Use with Apollo, Yoga, or any GraphQL server
const server = createYoga({ schema, resolvers });
```

### Agent Projection

Create AI-readable context for LLM agents:

```typescript
import { projectAgentContext } from '@manifesto-ai/projection-agent';

const context = projectAgentContext(runtime, {
  includeAvailableActions: true,
  includeFieldDescriptions: true,
});

// Pass to your LLM
const response = await llm.chat({
  messages: [
    { role: 'system', content: context.systemPrompt },
    { role: 'user', content: userMessage },
  ],
});
```

## Development

### Prerequisites

- Node.js >= 22
- pnpm >= 9

### Setup

```bash
# Clone the repository
git clone https://github.com/manifesto-ai/projection.git
cd projection

# Install dependencies
pnpm install

# Build all packages
pnpm build

# Run tests
pnpm test

# Type check
pnpm typecheck
```

### Project Structure

```
projection/
├── packages/
│   ├── projection-ui/        # UI state projection
│   ├── projection-graphql/   # GraphQL projection
│   └── projection-agent/     # Agent context projection
├── .github/workflows/        # CI/CD
├── package.json              # Root workspace config
├── pnpm-workspace.yaml
└── turbo.json                # Build orchestration
```

## Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Related Projects

- [@manifesto-ai/core](https://github.com/manifesto-ai/core) - Core domain modeling and runtime
- [@manifesto-ai/bridge](https://github.com/manifesto-ai/core/tree/main/packages/bridge) - Framework bindings

## License

MIT © [Manifesto AI](https://github.com/manifesto-ai)
