# CloudGuard Mock AWS MCP Server

CloudGuard is a Node.js 22.14+ TypeScript MCP server for investigating mock AWS cloud spend. It exposes read-only inventory and evidence-based waste findings, creates cleanup plans, requires a human CLI approval separate from the model, executes only an approved mock action, and records before/after verification in an audit log.

All resources and operations in this version are local mock data. No AWS account is contacted and no real cloud resources can be changed. OpenAI, Anthropic, and Gemini are MCP host/model choices; this server does not make model API calls.

## Run

```powershell
npm install
npm run build
npm run scan
npm run dev
```

The stdio server writes protocol traffic to stdout; readiness diagnostics go to stderr. `.vscode/mcp.json` points VS Code at the compiled server, so run `npm run build` before connecting it.

## Cleanup workflow

1. Call `list_resources`, `inspect_resource`, and `scan_waste` to gather observed mock state and findings.
2. Call `create_cleanup_plan` for an actionable resource. This only creates a proposed plan.
3. Review the plan and resource evidence, then approve from a separate terminal with `npm run approve -- <plan-id>`. The CLI displays the plan and resource and requires the exact phrase `APPROVE <ACTION> <RESOURCE_ID>`.
4. Call `execute_approved_plan` with that one plan ID. The server checks the exact approved scope, performs one mock operation, and records before/after state.
5. Call `verify_resource` and `read_audit_log` to inspect the result.

Plans without explicit CLI approval cannot execute. Production and unknown-environment actions are blocked. Snapshot deletion requires explicitly expired retention and no recorded dependencies. Estimates derive only from fixture values and are not guaranteed savings.

## MCP tools

- `list_resources`
- `inspect_resource`
- `scan_waste`
- `create_cleanup_plan`
- `list_cleanup_plans`
- `execute_approved_plan`
- `verify_resource`
- `read_audit_log`

The mock state persists at `.data/cloudguard.json`. Set `CLOUDGUARD_STATE_PATH` to use another local state file.

## Boundaries

This implementation does not connect to TrueForge's proprietary agent API or any live AWS APIs because those contracts were not provided. Before adding a real provider, implement and test a provider adapter, establish identity/authorization for approval, handle concurrent state updates, and add provider-side idempotency and verification. Never reuse the mock adapter as a production cloud control.

## SDK

The server uses the official [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) package `@modelcontextprotocol/server`, with Zod 4 input schemas and the SDK's stdio transport.