# Source register

## Campaign data

The campaign totals come from the internal Applied Evidence export used for this paper. The export contains 171 repository rows. Of these, 158 were assigned to an owner and 147 were marked Complete with a result. The assigned set contains 157 distinct repository URLs because Langflow appears twice. Thirteen unassigned rows are excluded.

The supplied campaign aggregate is 85 defects. The export has free-form result summaries rather than one normalized row per defect. It is therefore unsuitable for deriving an independent defect count or severity distribution.

The export records no reviewed commit hashes. The four case studies in the report use commits recovered from retained KIT sessions and their evidence packages.

## Review anchors

| Case | Repository | Commit | Evidence status |
| --- | --- | --- | --- |
| Langflow | [langflow-ai/langflow](https://github.com/langflow-ai/langflow) | [`7db38d1abbdc9ae764f8e7e28da64cb5104217f6`](https://github.com/langflow-ai/langflow/commit/7db38d1abbdc9ae764f8e7e28da64cb5104217f6) | Exact review commit, source trace, and non-destructive validation canary. |
| Mastra | [mastra-ai/mastra](https://github.com/mastra-ai/mastra) | [`7a0d62f7ad60937ecc0f31993f1eab7281287fc5`](https://github.com/mastra-ai/mastra/commit/7a0d62f7ad60937ecc0f31993f1eab7281287fc5) | Exact review commit, source trace, and targeted middleware canary. |
| Daytona | [daytonaio/daytona](https://github.com/daytonaio/daytona) | [`4ee2c6365b851cbc7073ca4cea2f9ebba0caf832`](https://github.com/daytonaio/daytona/commit/4ee2c6365b851cbc7073ca4cea2f9ebba0caf832) | Exact review commit and source-confirmed conditional fail-open. No full deployment reproduction. |
| Onyx | [onyx-dot-app/onyx](https://github.com/onyx-dot-app/onyx) | [`c424edb5118743443b3438c7e4895d0c962a794a`](https://github.com/onyx-dot-app/onyx/commit/c424edb5118743443b3438c7e4895d0c962a794a) | Exact review commit and source-confirmed authorization failure. No two-user runtime reproduction. |

## Langflow source paths

- [`src/backend/base/langflow/api/v2/schemas.py`](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/backend/base/langflow/api/v2/schemas.py#L105-L335)
- [Shell execution in `src/lfx/src/lfx/base/mcp/util.py`](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/lfx/src/lfx/base/mcp/util.py#L1581-L1626)
- [Command reconstruction in the same file](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/lfx/src/lfx/base/mcp/util.py#L2141-L2201)
- [`mcp_servers_locked` default](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/lfx/src/lfx/services/settings/groups/mcp.py#L65-L69)
- [Later upstream MCP stdio hardening](https://github.com/langflow-ai/langflow/commit/eba285edf1dd4a33bf23a9cb8113c991fcdf3d1d)

## Mastra source paths

- [Custom route default in `packages/core/src/server/index.ts`](https://github.com/mastra-ai/mastra/blob/7a0d62f7ad60937ecc0f31993f1eab7281287fc5/packages/core/src/server/index.ts#L58-L117)
- [Early pass in `coreAuthMiddleware`](https://github.com/mastra-ai/mastra/blob/7a0d62f7ad60937ecc0f31993f1eab7281287fc5/packages/server/src/server/auth/helpers.ts#L347-L381)
- [Express custom-route authentication decision](https://github.com/mastra-ai/mastra/blob/7a0d62f7ad60937ecc0f31993f1eab7281287fc5/server-adapters/express/src/index.ts#L618-L650)

## Daytona source paths

- [Public-status decision in `get_sandbox_target.go`](https://github.com/daytonaio/daytona/blob/4ee2c6365b851cbc7073ca4cea2f9ebba0caf832/apps/proxy/pkg/proxy/get_sandbox_target.go#L60-L78)
- [One-hour public-status cache](https://github.com/daytonaio/daytona/blob/4ee2c6365b851cbc7073ca4cea2f9ebba0caf832/apps/proxy/pkg/proxy/get_sandbox_target.go#L163-L200)
- [Best-effort cache invalidation](https://github.com/daytonaio/daytona/blob/4ee2c6365b851cbc7073ca4cea2f9ebba0caf832/apps/api/src/sandbox/services/proxy-cache-invalidation.service.ts#L27-L50)
- [Redis and in-process cache modes](https://github.com/daytonaio/daytona/blob/4ee2c6365b851cbc7073ca4cea2f9ebba0caf832/apps/proxy/pkg/proxy/proxy.go#L98-L128)

## Onyx source paths

- [`user_can_access_chat_file`](https://github.com/onyx-dot-app/onyx/blob/c424edb5118743443b3438c7e4895d0c962a794a/backend/onyx/access/access.py#L213-L273)
- [Authenticated file download endpoint](https://github.com/onyx-dot-app/onyx/blob/c424edb5118743443b3438c7e4895d0c962a794a/backend/onyx/server/query_and_chat/chat_backend.py#L893-L930)
- [Python-tool artifact classification](https://github.com/onyx-dot-app/onyx/blob/c424edb5118743443b3438c7e4895d0c962a794a/backend/onyx/tools/tool_implementations/python/python_tool.py#L304-L315)

## Evidence limits

All four cases have retained source, commit, session, and KIT evidence packages. Langflow and Mastra have focused validation canaries. Daytona and Onyx have confirmed source paths with impact that depends on deployment state or an additional attacker prerequisite. Their case studies preserve those qualifications.

The K proof packages were constructed and were not machine-checked. Repository history and deployment configuration can change after publication. Current releases and remediation status require separate verification.
