# Molt Marketplace — Documentation

Central documentation for the Molt Marketplace project (ERC-8004 Agent Identity NFT Marketplace on Monad).

## Table of Contents

| Document | Description |
|----------|-------------|
| [Architecture](./architecture.md) | System diagram, data flow, component descriptions |
| [API Contract](./api-contract.md) | API endpoint specification (request/response shapes) |
| [API Reference](./api-reference.md) | Complete API docs with curl examples |
| [Contract Features](./contract-features.md) | MoltMarketplace smart contract feature reference |
| [Deployment](./deployment.md) | Local dev setup guide + deployed contract addresses |
| [Monad Chains](./monad-chains.md) | Chain configuration, contract addresses, event signatures |
| [Shared Types](./shared-types.md) | TypeScript/Rust shared type definitions |
| [Implementation Plan](./implementation-plan.md) | Original development plan and phases |
| [llms.txt](./llms.txt) | AI-friendly project context summary |

## Related Docs

| Document | Location | Description |
|----------|----------|-------------|
| [Project README](../README.md) | Root | Project overview, quick start, tech stack |
| [CLAUDE.md](../CLAUDE.md) | Root | Agent team roles, git workflow, project guidelines |
| [Contract README](../contract/README.md) | contract/ | Smart contract docs, roles, deployment, testing |
| [Docs Review](../DOCS-REVIEW.md) | Root | Documentation completeness audit |

## Live Deployments

- **Frontend**: [mm-8004-frontend.vercel.app](https://mm-8004-frontend.vercel.app)
- **Backend**: [nad-8004-backend-production.up.railway.app](https://nad-8004-backend-production.up.railway.app)
- **Contracts**: See [deployment.md](./deployment.md) for addresses

## ⚠️ Documentation Status

Some docs were written for the original EIP-8004 Dashboard scope and have not been fully updated to reflect the MoltMarketplace integration. See [DOCS-REVIEW.md](../DOCS-REVIEW.md) for the full audit.

**Known gaps:**
- Marketplace API endpoints (10 endpoints) are not yet in api-contract.md or api-reference.md
- Marketplace shared types are not yet in shared-types.md
- Frontend route structure in several docs reflects old routes
- Backend default port in examples should be 3001 (not 8080)
