# KIT-Assisted Open-Source Codebase Sweep

## Executive summary

A two-week campaign used the KIT to review security-critical paths in a large set of open-source repositories. The assigned queue contained 158 entries representing 157 distinct repository URLs. Five human owners supervised the work. The repositories span more than 70 million estimated lines of code across 11 primary languages. The campaign recorded 85 defects.

The KIT combines intent recovery, security invariant design, focused source analysis, lightweight formal models, proof construction, and targeted validation. It helped reviewers move from a large repository to a small set of security claims, then trace each claim through implementation paths that control authority, assets, process execution, or irreversible state.

The campaign was a scoped security sweep. It did not attempt exhaustive line-by-line audit coverage of every repository. The 70 million line figure describes the size of the portfolio under review. It does not represent the number of lines manually inspected or formally verified.

## Campaign setup and results

The tracking export assigns 158 repository entries to five owners. One URL appears twice, leaving 157 distinct assigned repositories. At the time of export, 147 entries had both a Complete status and a populated result. Eleven assigned entries lacked completion metadata. Thirteen further rows were unassigned and are excluded from the campaign totals in this report.

The repository set covers Python, TypeScript, JavaScript, Go, Rust, Solidity, Java, Kotlin, Scala, C#, and Huff as primary implementation languages. Some repositories combine several of these languages. Portfolio size was estimated from repository-scale metadata. The estimate should be read as more than 70 million lines of source across the assigned set.

Each review began with repository acquisition and subsystem mapping. The reviewer identified security-critical assets and roles, recovered intended behavior from public documentation and tests, and wrote explicit invariants for the selected scope. The KIT then guided call-path analysis, dataflow checks, state-transition review, and proof construction. Small canaries were used where a local check could distinguish a real implementation flaw from a speculative proof gap.

The formal layer used K definitions and reachability claims for relevant language fragments. Most packages were constructed proof artifacts rather than completed machine-checked proofs. This distinction matters. A constructed claim organizes the security argument and exposes missing preconditions. It does not carry the assurance of a successful proof run. The campaign retained 574 evidence files, including 110 K files, to preserve the reasoning behind the findings.

Human work centered on scope selection, setup recovery, evidence review, severity decisions, and report preparation. No normalized timesheet was collected. Person-hour totals therefore cannot be stated reliably. The available trace for one owner records 29 review sessions, 92 continuation cycles, and 56 repository evidence directories over eight calendar days. This supports the description of an AI-heavy review supervised by five people over roughly two weeks. It should not be interpreted as five full-time reviewers working for ten person-weeks.

The campaign aggregate is 85 defects. The export does not provide stable per-defect identifiers or a normalized severity table, so this report does not infer a severity distribution. The four case studies below have pinned review commits and retained KIT evidence packages. Langflow and Mastra include targeted validation canaries. Daytona and Onyx are source-confirmed boundary failures whose full deployment impact remains conditional.

## Case study 1

### Langflow validated one command representation and executed another

Langflow lets authenticated users configure stdio-based MCP servers. The reviewed code allowed a multi-word `command` field for frontend compatibility. It checked only the first token of that field against an executable allowlist. Separate validators rejected dangerous syntax in the `args` array.

The execution path later joined the command and arguments into one string. On Unix it passed that string through `bash -c`, with an equivalent command-shell wrapper on Windows. Syntax placed after the first token in the multi-word `command` field therefore reached a shell without passing through the argument checks. The same syntax was rejected when placed in `args`.

Where MCP server configuration remained unlocked, its default at the reviewed commit, an ordinary authenticated user could reach backend command execution with the service account's privileges. The result was an authenticated remote-code-execution condition. The risk included application data, configured credentials available to the process, and access to adjacent services reachable from the host.

The KIT compared validation grammar with execution grammar. The security claim required every token interpreted by the shell to pass one policy before execution. Field-level review looked sound in isolation. End-to-end dataflow showed that validation tokenized only the start of `command`, while execution interpreted the complete recomposed string. A non-destructive canary confirmed the acceptance difference between the two fields.

This case had an exact reviewed commit in the retained session record. Later upstream security work centralized MCP stdio policy and added checks closer to execution. The analysis here applies to the reviewed commit and does not characterize the current Langflow release.

## Case study 2

### Mastra let default-protected custom routes bypass authentication

Mastra allows developers to register custom API routes that invoke agents, models, tools, workflows, and application code. Its route API states that authentication is enabled by default and can be disabled explicitly with `requiresAuth: false`. Developers using the documented direct server adapters should therefore be able to rely on a default-protected route.

At the reviewed commit, direct Express, Hono, Fastify, and Koa adapter paths did not always derive the custom-route authentication map from the configured routes. The Express adapter ran route authentication only when that map identified the matched path as protected. When the map was absent, the shared middleware also returned early for paths outside the global protected patterns. A custom route outside the default `/api/*` pattern could reach its handler even though the route had never opted out of authentication.

The impact depends on the handler, which is why this boundary is serious. Public examples include completion routes that can invoke models, agents, and tools. An unauthenticated caller could consume provider resources, retrieve application data exposed by the handler, or trigger tool side effects that the developer expected to sit behind Mastra authentication.

The KIT stated the rule in one line before following adapter code. Every registered custom route must authenticate unless it explicitly sets `requiresAuth: false`. It then compared adapters that constructed route-auth metadata with direct adapters that did not. A targeted canary called Mastra's actual core authentication middleware. With no custom-route map, the request advanced and authentication was never called. Adding the expected map entry produced a 401 response. This gave the finding both a source-level path and a controlled implementation check.

## Case study 3

### Daytona could preserve public preview access after a sandbox became private

Daytona provides browser-accessible previews for services running inside sandboxes. Public sandboxes can be reached without a preview credential. Private sandboxes require authentication. The proxy caches each sandbox's public status for one hour to avoid an API lookup on every request.

When an owner changed a sandbox from public to private, the API attempted to delete the proxy's cached public status. A failed deletion was caught and logged while the privacy transition still completed. The proxy continued to trust a cached `true` value and skipped authentication for non-reserved preview ports. Daytona also supported an in-process cache mode that the API-side Redis invalidation service could not clear.

This is a conditional fail-open path. It requires a public-status value to be cached before the transition and invalidation to fail, miss an instance, or be unable to reach the active cache. When those conditions hold, a sandbox reported as private can remain anonymously reachable for as long as the stale value survives. Services inside the sandbox may expose source code, development data, dashboards, or application endpoints that the owner believed had become private.

The KIT modeled privacy change as a state-transition invariant. Once `public=false` commits, no earlier cache state may authorize anonymous preview. The proof construction left a residual path where sandbox state changed while cache state remained `true`. Source review then connected that path to the one-hour cache, swallowed invalidation error, and proxy branch that bypassed authentication. The result is recorded as a likely P1 conditional failure. A full local deployment reproduction was recommended and was not claimed as complete.

## Case study 4

### Onyx treated generated artifacts as readable by every authenticated user

Onyx stores files produced by image generation and its Python execution tool in a shared FileStore. The authenticated download endpoint normally checks ownership, shared-chat access, persona access, or document ACLs before returning bytes. Generated artifacts used the `CHAT_IMAGE_GEN` origin, including outputs produced by the Python tool.

The access function contained a separate branch for that origin. If a matching FileStore record existed, it returned access without linking the file to the requesting user, chat, session, or document permissions. Any authenticated user who obtained the file ID could pass the check and receive the stored bytes.

Generated artifacts are not necessarily harmless images. Code execution can produce reports, CSV files, transformed uploads, and outputs derived from private context. The defect can therefore cross user boundaries and disclose material that belongs to another session. Exploitation requires the target file ID. The retained evidence did not establish that IDs were guessable or exposed through a second flaw, so the report treats the authorization failure as confirmed and its P1 impact as conditional.

The KIT classified generated files as authority-bearing objects and required every download to resolve to an owner or an explicit sharing rule. It traced the returned tool URL into the common download endpoint, then compared each authorization branch. Ownership and ACL branches preserved context. The origin-only branch discarded it. That single exception made the file-access claim fail and identified a narrow repair boundary without requiring a broad review of the storage system.

## Evidence boundaries

These case studies come from retained KIT sessions rather than the campaign CSV summaries. Each evidence package identifies or preserves a pinned source revision. Langflow has a non-destructive validation canary. Mastra has a targeted canary against the actual authentication middleware. Daytona and Onyx have source-confirmed failing paths and constructed proof obligations, with end-to-end reproductions still recommended.

The K packages for all four cases were constructed rather than machine-checked. The studies describe trust-boundary failures and impact without exploit payloads, deployment instructions, or weaponized proof-of-concept code. Each finding should be revalidated against the version and configuration used in a production deployment.

## Conclusion

The sweep used the KIT to turn public security intent into explicit invariants and trace them through large, unrelated codebases. The selected cases found backend command execution, an authentication fail-open, a stale privacy decision, and a cross-user artifact authorization gap.

The campaign shows the KIT's value as a triage and evidence-production system for large review queues. Pinned revisions and focused canaries made the strongest findings suitable for deeper review. Deployment testing, machine-checked proofs, and maintainer remediation remain separate work.
