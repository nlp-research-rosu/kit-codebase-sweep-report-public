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

The campaign aggregate is 85 defects. The export does not provide stable per-defect identifiers or a normalized severity table, so this report does not infer a severity distribution. The case study below is limited to Langflow because its exact reviewed commit and retained evidence package were available for independent inspection.

## Case study

### Langflow validated one command representation and executed another

Langflow lets authenticated users configure stdio-based MCP servers. The reviewed code allowed a multi-word `command` field for frontend compatibility. It checked only the first token of that field against an executable allowlist. Separate validators rejected dangerous syntax in the `args` array.

The execution path later joined the command and arguments into one string. On Unix it passed that string through `bash -c`, with an equivalent command-shell wrapper on Windows. Syntax placed after the first token in the multi-word `command` field therefore reached a shell without passing through the argument checks. The same syntax was rejected when placed in `args`.

Where MCP server configuration remained unlocked, its default at the reviewed commit, an ordinary authenticated user could reach backend command execution with the service account's privileges. The result was an authenticated remote-code-execution condition. The risk included application data, configured credentials available to the process, and access to adjacent services reachable from the host.

The KIT compared validation grammar with execution grammar. The security claim required every token interpreted by the shell to pass one policy before execution. Field-level review looked sound in isolation. End-to-end dataflow showed that validation tokenized only the start of `command`, while execution interpreted the complete recomposed string. A non-destructive canary confirmed the acceptance difference between the two fields.

This case had an exact reviewed commit in the retained session record. Later upstream security work centralized MCP stdio policy and added checks closer to execution. The analysis here applies to the reviewed commit and does not characterize the current Langflow release.

## Evidence boundaries

The campaign CSV records repository URLs and finding summaries without commit hashes. The exact Langflow commit was recovered from retained session metadata and matched the commit in the detailed review artifacts. The case study is based on that pinned source, the KIT evidence package, and a non-destructive validation canary.

The case study describes the trust-boundary failure and impact without exploit payloads, deployment instructions, or weaponized proof-of-concept code. The finding should still be revalidated against the version used in any production deployment. Later upstream work changed this security boundary.

## Conclusion

The sweep used the KIT to turn public security intent into explicit invariants and trace those invariants through large, unrelated codebases. In the documented Langflow case, validation and execution interpreted the same command under different grammars, which exposed authenticated backend command execution.

The campaign shows the KIT's value as a triage and evidence-production system for large review queues. Exact revision capture, targeted runtime validation, and maintainer remediation remain essential parts of a complete security process.
