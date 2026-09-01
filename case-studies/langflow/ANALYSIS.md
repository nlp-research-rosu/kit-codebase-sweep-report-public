# Langflow MCP command validation bypass

This is a confirmed P1. An authenticated user who could configure MCP stdio servers could reach backend command execution in the reviewed default configuration.

The reviewed revision is [`7db38d1abbdc9ae764f8e7e28da64cb5104217f6`](https://github.com/langflow-ai/langflow/commit/7db38d1abbdc9ae764f8e7e28da64cb5104217f6).

The evidence includes a pinned source trace, constructed K claims, and a non-destructive local canary against the validation and launch path.

## 1. System boundary

Langflow lets users connect stdio-based Model Context Protocol servers. A server configuration names an executable in `command` and supplies additional values in `args`. The reviewed schema treated its executable list and dangerous-token list as security controls. Tests also established that multi-word commands were a supported compatibility feature.

The security boundary sits between stored user configuration and backend process creation. A normal user could manage MCP servers unless an operator enabled `mcp_servers_locked`. That setting [defaulted to `False`](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/lfx/src/lfx/services/settings/groups/mcp.py#L65-L69) at the reviewed revision.

Every token that could affect process execution had to pass the same policy before Langflow created a subprocess.

## 2. Root cause

The schema split one logical command line across two fields and applied different policies to them.

[`MCPServerConfig`](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/backend/base/langflow/api/v2/schemas.py#L105-L335) extracted the first token from `command` and checked that token against the executable allowlist. It returned the original multi-word string after that check. The dangerous flag and metacharacter checks applied to the separate `args` list.

The execution path later erased that distinction. [`update_tools`](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/lfx/src/lfx/base/mcp/util.py#L2141-L2201) split the full `command` value into tokens, appended `args`, and reconstructed one command string. [`MCPStdioClient`](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/lfx/src/lfx/base/mcp/util.py#L1581-L1626) then passed that string through a command shell.

This created two grammars for the same process launch:

- The validation grammar treated the first `command` token as the executable and inspected dangerous values only in `args`.
- The execution grammar interpreted every token from both fields as one shell-mediated command.

A dangerous execution flag rejected in `args` could therefore survive when placed after the allowlisted executable inside `command`.

## 3. Reachability and impact

The path required an active authenticated account with permission to create or update an MCP stdio server. The default unlocked setting gave that ability to ordinary users. Triggering MCP server inspection or tool loading caused the backend to process the stored configuration.

Successful use of the bypass ran code with the Langflow backend process account. That process can hold database connection details, application secrets, provider credentials, mounted configuration, and network access granted to the deployment. The exact data at risk depends on the operator's configuration. The result was a path from low-privileged application access to backend process execution.

Locking MCP server management to trusted administrators reduced attacker reachability. It did not repair the validation mismatch.

## 4. How the KIT found it

The KIT began with public intent rather than the field validators in isolation. The evidence ledger recorded four facts:

1. Langflow described the executable list as a security allowlist.
2. Its dangerous-token policy rejected code-execution flags.
3. Multi-word `command` values were intentionally supported.
4. The launch path executed tokens from both `command` and `args`.

Those facts produced the claim `VALIDATE-COMMAND-DANGEROUS-REJECT`. It required the policy to reject a dangerous token regardless of which input field carried it. A second claim, `LAUNCH-USES-VALIDATED-TOKENS`, tied the accepted token stream to the stream used at launch.

The candidate implementation could not satisfy the first claim. Its acceptance condition depended on an allowed first token and a clean `args` list. It had no condition for dangerous suffix tokens already present in `command`. The resulting counterexample was structural. The validator accepted a token stream that the launcher later interpreted with greater authority.

The K package captured this difference in a reduced command model. It was constructed and was not machine-checked. The proof obstacle came from the source trace and did not depend on a successful `kprove` run.

## 5. Validation record

The retained audit ran a non-destructive local canary against the reviewed validation and command-construction behavior. It compared equivalent dangerous flags placed in `args` and in the suffix of `command`. The first representation was rejected. The second was accepted.

A harmless marker confirmed that the accepted representation reached process execution through the same Unix launch pattern. The public analysis omits the payload and local transcript because neither is needed to establish the boundary failure.

This evidence supports the P1 classification for default multi-user deployments. It is stronger than a source-only hypothesis because the canary confirmed both the validation asymmetry and execution reachability.

## 6. Repair direction and limits

The executable and its arguments should remain structured through validation and launch. Langflow can enforce that property by accepting only an executable in `command`, validating the complete normalized argument vector, and passing both directly to a subprocess API without a command shell.

If compatibility requires multi-word command input, Langflow must normalize it before policy checks and apply one policy to every executable token. Regression tests should present the same restricted flag through each accepted representation.

This case describes the pinned revision. [Later upstream work](https://github.com/langflow-ai/langflow/commit/eba285edf1dd4a33bf23a9cb8113c991fcdf3d1d) centralized MCP stdio policy closer to execution. This report makes no claim about current Langflow releases.
