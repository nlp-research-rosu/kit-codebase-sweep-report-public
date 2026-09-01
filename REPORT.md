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

The campaign aggregate is 85 defects. The export does not provide stable per-defect identifiers or a normalized severity table, so this report does not infer a severity distribution. The five cases below were selected from findings recorded as P1 because they show distinct failure classes and direct security impact.

## Case study 1

### Mosaik accepted TDX quotes without establishing hardware provenance

Mosaik uses Intel TDX attestation tickets to control access to protected peer groups and streams. An accepted ticket should establish that a peer ran in a genuine TDX environment with the required measurements. That guarantee depends on two separate checks. The quote must be internally signed, and the signing key must chain to a trusted platform certificate.

At the re-check anchor, `TdxTicket::verify_quote` called `Quote::from_bytes` and then compared the quote report data with Mosaik-specific ticket data. The `tdx-quote` parser verified the quote body against an attestation key embedded in the same quote. Mosaik did not call the dependency's `verify` or `verify_with_pck` functions, which establish the link to a platform certification key.

The accepted object was therefore self-consistent without being rooted in trusted hardware. A party outside a TDX environment could choose an attestation key, produce matching quote contents, and bind those contents to its own peer identity. Measurement checks later in the validator would operate on attacker-selected quote fields. TDX-gated membership could then admit a peer with no valid hardware provenance.

The KIT separated parsing validity from trust validity. It expressed the acceptance rule as a provenance invariant and followed the verifier call graph into the dependency. The call graph ended after internal signature verification and report-data binding. The missing certificate-chain step made the invariant unprovable and the dependency source confirmed the gap.

The impact reaches every feature that treats a TDX ticket as an admission credential. A forged ticket can defeat the intended hardware boundary and expose protected group state or stream participation. The campaign also recorded follow-on consensus risks in Mosaik, which increased the consequence of unauthorized membership. Those secondary findings are outside this case study.

## Case study 2

### Solana Portal minted bridge assets to an unconstrained account

Solana Portal receives cross-chain messages through bridge adapters. A valid token-transfer message causes the Portal program to mint principal `$M`, then either wrap it into a whitelisted extension token or retain it for a later claim. Relayers supply a positional list of accounts used by this flow.

The account parser validated the recipient, the recipient token account, the swap program, and several program-derived addresses. It returned `authority_m_token_account` as a generic account without checking that it was the expected associated token account owned by the Portal authority. The receive instruction then used that account as the destination of a mint operation.

For a destination token outside the whitelist, the later wrap call does not run. The newly minted `$M` remains in the supplied account while the Portal increments `unclaimed_m_balance` as if the funds remained in its custody. A relayer handling a legitimately authenticated transfer can therefore redirect the mint to a compatible token account under different control. The message is consumed, the accounting liability is created, and the Portal's custody account does not receive the corresponding asset.

This creates immediate asset loss or unauthorized issuance and a delayed solvency failure. A later claim against `unclaimed_m_balance` expects backing that was never deposited into the Portal account. The whitelisted branch has an additional CPI that limits this specific path, which is why the non-whitelisted storage branch is security-critical.

The KIT modeled the bridge flow as an asset-conservation claim. Every increase in the unclaimed liability must be matched by principal held in the canonical Portal account. It then traced all caller-supplied accounts from parsing to the mint CPI and compared the constraints with sibling instructions. Those instructions explicitly bind equivalent accounts to the Portal authority. The missing binding in `TokenTransferPayloadAccounts` was the single condition needed to violate the claim.

## Case study 3

### Moonhatch allowed premature use of commingled graduation funds

Moonhatch's `PumpRand` contract holds ETH from purchases across many coins. A coin is intended to graduate after its deposits reach one ETH. Graduation mints liquidity tokens, marks the coin transferable, and sends one ETH to a DEX router.

The on-chain `deployGraduated` function was public and lacked an authorization check or a check of `graduated[coinId]`. It always sent the fixed one ETH graduation amount from the contract's global balance. A caller could invoke it for a partially funded coin once that coin had a nonzero token supply. Funds collected for other coins would cover the difference.

The effect crosses accounting domains. One coin receives DEX liquidity funded by deposits associated with other coins. The contract balance can then fall below the sum needed for pending refunds and future graduations. The selected token is also moved into an irreversible lifecycle state before the `PumpRand` bookkeeping marks it as graduated, leaving contract and token state inconsistent.

The server's deployment handler checked the graduation flag before calling the client. That check did not protect the contract because any account could call the on-chain function directly. The effective authorization boundary was the public contract entry point.

The KIT first wrote a custody invariant that related the contract balance to per-coin deposits and graduation obligations. It then enumerated every function able to reduce the shared balance and every irreversible lifecycle transition. `deployGraduated` spent a fixed amount without proving either that the selected coin had earned it or that the caller was trusted. This combined a missing state guard with commingled custody, producing a system-wide insolvency risk.

## Case study 4

### SONGS Protocol trusted an unregistered revenue-claim contract

SONGS Protocol's `DistributorWallet` pools stablecoin revenue for several wrapped songs. Distribution epochs store per-song amounts by index. A claimant supplies a wrapped-song address, the wallet obtains a token-management contract from it, and share balances determine the payout.

`claimEpochEarnings` did not establish that the supplied wrapped-song address belonged to `managedWrappedSongs`. Solidity mappings return zero for an unknown key, so `wsRedeemIndexList[_wrappedSong]` selected the first song's distribution slot when the address had never been registered. The function also trusted the supplied contract to identify the token manager used for balance and total-share queries.

A contract outside the protocol registry could report share values that assign the entire indexed distribution to its caller. The wallet would transfer pooled stablecoin and mark the claim against the unregistered address. Distinct unregistered addresses create distinct claim keys, so the same indexed pool can be targeted repeatedly until available funds are exhausted.

The KIT expressed the claim path as a membership and index-integrity property. Any address used to select an epoch amount must equal the managed song stored at that index. It checked where the mapping was written, observed the zero-value behavior for absent keys, and followed all external calls made before transfer. The path contained no registry proof and no reverse check against the managed array.

The direct impact is theft of pooled stablecoin belonging to legitimate artists and token holders. Pooling expands the loss beyond one malformed song entry because the transfer draws from a wallet-wide balance.

## Case study 5

### Langflow validated one command representation and executed another

Langflow lets authenticated users configure stdio-based MCP servers. The reviewed code allowed a multi-word `command` field for frontend compatibility. It checked only the first token of that field against an executable allowlist. Separate validators rejected dangerous syntax in the `args` array.

The execution path later joined the command and arguments into one string. On Unix it passed that string through `bash -c`, with an equivalent command-shell wrapper on Windows. Syntax placed after the first token in the multi-word `command` field therefore reached a shell without passing through the argument checks. The same syntax was rejected when placed in `args`.

Where MCP server configuration remained unlocked, its default at the reviewed commit, an ordinary authenticated user could reach backend command execution with the service account's privileges. The result was an authenticated remote-code-execution condition. The risk included application data, configured credentials available to the process, and access to adjacent services reachable from the host.

The KIT compared validation grammar with execution grammar. The security claim required every token interpreted by the shell to pass one policy before execution. Field-level review looked sound in isolation. End-to-end dataflow showed that validation tokenized only the start of `command`, while execution interpreted the complete recomposed string. A non-destructive canary confirmed the acceptance difference between the two fields.

This case had an exact reviewed commit in the retained session record. Later upstream security work centralized MCP stdio policy and added checks closer to execution. The analysis here applies to the reviewed commit and does not characterize the current Langflow release.

## Evidence boundaries

The campaign CSV records repository URLs and finding summaries without commit hashes. The exact Langflow commit was recovered from the retained review session. For the other four cases, this report uses default-branch commits that were unchanged from before the campaign as source re-check anchors. Those commits reproduce the cited code paths. They are not asserted to be the original review revisions.

The case studies describe flawed trust boundaries and impact without exploit payloads, deployment instructions, or weaponized proof-of-concept code. Findings should still be revalidated against the version used in any production deployment. Except for the later Langflow hardening noted above, remediation status was not established during preparation of this report.

## Conclusion

The sweep found that the most serious defects were narrow mismatches between an intended security rule and the exact value trusted at runtime. A quote was parsed without proving provenance. A bridge liability was credited without proving custody. A lifecycle transition spent shared funds without proving eligibility. A revenue claim selected an index without proving membership. A command was validated under a different grammar from the one used for execution.

The KIT made these mismatches easier to review across unrelated languages and domains by turning public intent into explicit invariants, then tracing each invariant to code and a focused validation step. The results support its use as a triage and evidence-production system for large review queues. Machine-checked proofs, deployment-specific testing, and maintainer remediation remain separate work.