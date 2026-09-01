# Source register

## Campaign data

The campaign totals come from the internal Applied Evidence export used for this paper. The export contains 171 repository rows. Of these, 158 were assigned to an owner and 147 were marked Complete with a result. The assigned set contains 157 distinct repository URLs because Langflow appears twice. Thirteen unassigned rows are excluded.

The supplied campaign aggregate is 85 defects. The export has free-form result summaries rather than one normalized row per defect. It is therefore unsuitable for deriving an independent defect count or severity distribution.

The export records no reviewed commit hashes. For that reason, the report limits its technical case study to Langflow, whose exact reviewed revision appears in retained session metadata and detailed review artifacts.

## Langflow review anchor

| Repository | Commit | Provenance |
| --- | --- | --- |
| [langflow-ai/langflow](https://github.com/langflow-ai/langflow) | [`7db38d1abbdc9ae764f8e7e28da64cb5104217f6`](https://github.com/langflow-ai/langflow/commit/7db38d1abbdc9ae764f8e7e28da64cb5104217f6) | Exact review commit recorded by the retained Langflow sessions and evidence package. |

## Relevant source paths

- [`src/backend/base/langflow/api/v2/schemas.py`](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/backend/base/langflow/api/v2/schemas.py#L105-L335)
- [Shell execution in `src/lfx/src/lfx/base/mcp/util.py`](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/lfx/src/lfx/base/mcp/util.py#L1581-L1626)
- [Command reconstruction in the same file](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/lfx/src/lfx/base/mcp/util.py#L2141-L2201)
- [`mcp_servers_locked` default](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/lfx/src/lfx/services/settings/groups/mcp.py#L65-L69)
- [Later upstream MCP stdio hardening](https://github.com/langflow-ai/langflow/commit/eba285edf1dd4a33bf23a9cb8113c991fcdf3d1d)

## Evidence limits

The Langflow case has source, commit, session, and non-destructive validation evidence. The retained reviews also reported other P1 candidates at the same commit, but several were based on static review of deployment examples without runtime confirmation. They are excluded from this paper pending targeted revalidation.

Repository history and deployment configuration can change after this register is published. Current releases and remediation status require separate verification.
