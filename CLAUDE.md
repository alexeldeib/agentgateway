# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Agentgateway is an open-source data plane for agentic AI connectivity, supporting MCP (Model Context Protocol) and A2A (Agent-to-Agent) protocols. It provides security, observability, and governance for agent-to-agent and agent-to-tool communication.

**Primary languages:** Rust (core gateway), TypeScript/React (UI dashboard)

## Build Commands

```bash
# Build release binary with UI (requires npm install in ui/ first)
make build

# Build UI separately
cd ui && npm install && npm run build

# Cross-compile to specific target
make build-target TARGET=<target> PROFILE=<profile>

# Build Docker images
make docker          # Standard image
make docker-musl     # MUSL-based image
```

## Testing

```bash
# Run all unit tests
make test

# Run tests for a specific crate
cargo test --package <crate-name>

# Run a single test
cargo test --lib <test_name>

# Validate example configs against the gateway
make validate
```

## Linting

```bash
# Check formatting and run clippy
make lint

# Auto-fix lint issues
make fix-lint

# UI linting
cd ui && npm run lint
```

## Code Generation

```bash
# Full generation pipeline (protos, schema, then fix-lint)
make gen

# Individual steps
make generate-apis      # Proto → Rust via buf
make generate-schema    # JSON schema via xtask
```

## Architecture

### Crate Structure

- `crates/agentgateway` - Core gateway implementation (MCP, A2A, HTTP, LLM, proxy logic)
- `crates/agentgateway-app` - Binary entry point
- `crates/core` - Shared utilities
- `crates/a2a-sdk` - Agent-to-Agent SDK
- `crates/xds` - xDS (Envoy-style dynamic config) API implementation
- `crates/hbone` - HBONE transport protocol
- `crates/mock-server` - Test/mock server
- `crates/xtask` - Build automation (schema generation)
- `ui/` - Next.js React dashboard

### Configuration System

Agentgateway supports three configuration modes (see `architecture/configuration.md`):
1. **Static** - Environment variables or YAML/JSON for global settings (logging, ports)
2. **Local** - File-based with hot-reload for backends, routes, policies
3. **XDS** - Remote control plane configuration via xDS protocol

Key difference from Envoy: resources point to their parent (routes reference listeners) rather than parents containing child lists. This reduces configuration fan-out.

### CEL Integration

CEL (Common Expression Language) is used extensively for:
- Log/trace attribute definitions
- HTTP header/body modifications
- Authorization policies
- Rate limit selectors

The `ContextBuilder` pattern dynamically captures only the data expressions need (see `architecture/cel.md`).

### Protocol Support

- **MCP** - `crates/agentgateway/src/mcp/` - Model Context Protocol for tool/resource access
- **A2A** - `crates/agentgateway/src/a2a/` - Agent-to-Agent protocol
- **HTTP** - `crates/agentgateway/src/http/` - HTTP routing/proxy
- **LLM** - `crates/agentgateway/src/llm/` - LLM provider integrations

## Conventions

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/): `feat:`, `fix:`, `docs:`, `refactor:`, `perf:`, `test:`, `chore:`

### Code Style

- **Rust**: Hard tabs, 2-space indentation, match block trailing comma (see `rustfmt.toml`)
- **Clippy**: Strict warnings as errors (`-D warnings`)
- **TypeScript**: ESLint + Prettier (see `ui/.eslintrc.json`)

### Feature Flags

Key Cargo features: `ui` (embedded dashboard), `jemalloc` (allocator), `schema` (JSON schema gen), `tls-ring` (TLS backend), `testing`

### Build Profiles

- `release` - LTO enabled, single codegen unit (production)
- `quick-release` - 16 codegen units, no LTO (faster dev builds)

## Examples

Working examples in `examples/` demonstrate various features:
- `basic/` - Simple MCP stdio
- `authorization/` - JWT + authz policies
- `multiplex/` - Multi-target routing
- `mcp-authentication/` - MCP with auth
- `a2a/` - A2A protocol
- `telemetry/` - Observability setup
- `tls/` - TLS termination

Run the gateway with an example: `cargo run -- -f examples/<name>/config.yaml`
