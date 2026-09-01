# Source register

## Campaign data

The campaign totals come from the internal Applied Evidence export used for this paper. The export contains 171 repository rows. Of these, 158 were assigned to an owner and 147 were marked Complete with a result. The assigned set contains 157 distinct repository URLs because Langflow appears twice. Thirteen unassigned rows are excluded.

The supplied campaign aggregate is 85 defects. The export has free-form result summaries rather than one normalized row per defect. It is therefore unsuitable for deriving an independent defect count or severity distribution.

The export records no reviewed commit hashes. Commit status for each case is identified below.

## Case-study anchors

| Case | Repository | Commit | Status |
| --- | --- | --- | --- |
| Mosaik | [flashbots/mosaik](https://github.com/flashbots/mosaik) | [`88b23b790473ba089e48b5e4b188ee3e0be59f3b`](https://github.com/flashbots/mosaik/commit/88b23b790473ba089e48b5e4b188ee3e0be59f3b) | Default-branch re-check anchor. The campaign export does not prove this was the original review commit. |
| Solana Portal | [m0-foundation/solana-portal](https://github.com/m0-foundation/solana-portal) | [`2fe16987b642dbcc95454d8d93b9da4799e40fd6`](https://github.com/m0-foundation/solana-portal/commit/2fe16987b642dbcc95454d8d93b9da4799e40fd6) | Default-branch re-check anchor. The campaign export does not prove this was the original review commit. |
| Moonhatch | [SeismicSystems/moonhatch](https://github.com/SeismicSystems/moonhatch) | [`c1971772b28b451ab2a50ac5736f10a1364b1b49`](https://github.com/SeismicSystems/moonhatch/commit/c1971772b28b451ab2a50ac5736f10a1364b1b49) | Default-branch re-check anchor. The campaign export does not prove this was the original review commit. |
| SONGS Protocol | [SONGS-TOOLS/songs-protocol](https://github.com/SONGS-TOOLS/songs-protocol) | [`2b3d493a24409e0863ac4da52992be0bb3c76323`](https://github.com/SONGS-TOOLS/songs-protocol/commit/2b3d493a24409e0863ac4da52992be0bb3c76323) | Default-branch re-check anchor. The campaign export does not prove this was the original review commit. |
| Langflow | [langflow-ai/langflow](https://github.com/langflow-ai/langflow) | [`7db38d1abbdc9ae764f8e7e28da64cb5104217f6`](https://github.com/langflow-ai/langflow/commit/7db38d1abbdc9ae764f8e7e28da64cb5104217f6) | Exact review commit recovered from retained session metadata. |

The first four default branches remained at the listed commits when re-checked on 1 September 2026.

## Relevant source paths

### Mosaik

- [`src/tee/tdx/ticket.rs`](https://github.com/flashbots/mosaik/blob/88b23b790473ba089e48b5e4b188ee3e0be59f3b/src/tee/tdx/ticket.rs#L187-L203)
- [`tdx-quote` 0.0.5 verification API](https://github.com/entropyxyz/tdx-quote/blob/8eefaca080fdcef279538d26f73041d581c6f490/src/lib.rs#L61-L193)

### Solana Portal

- [`packages/common/src/accounts.rs`](https://github.com/m0-foundation/solana-portal/blob/2fe16987b642dbcc95454d8d93b9da4799e40fd6/packages/common/src/accounts.rs#L8-L73)
- [`programs/portal/src/instructions/receive_message.rs`](https://github.com/m0-foundation/solana-portal/blob/2fe16987b642dbcc95454d8d93b9da4799e40fd6/programs/portal/src/instructions/receive_message.rs#L224-L342)
- [`packages/common/src/receive_metas.rs`](https://github.com/m0-foundation/solana-portal/blob/2fe16987b642dbcc95454d8d93b9da4799e40fd6/packages/common/src/receive_metas.rs#L72-L85)

### Moonhatch

- [`contracts/src/pump/PumpRand.sol`](https://github.com/SeismicSystems/moonhatch/blob/c1971772b28b451ab2a50ac5736f10a1364b1b49/contracts/src/pump/PumpRand.sol#L233-L258)
- [`crates/server/src/http.rs`](https://github.com/SeismicSystems/moonhatch/blob/c1971772b28b451ab2a50ac5736f10a1364b1b49/crates/server/src/http.rs#L94-L109)

### SONGS Protocol

- [`contracts/protocol/NonUpgradable/DistributorWallet.sol`](https://github.com/SONGS-TOOLS/songs-protocol/blob/2b3d493a24409e0863ac4da52992be0bb3c76323/contracts/protocol/NonUpgradable/DistributorWallet.sol#L154-L185)
- [Wrapped-song registration in the same contract](https://github.com/SONGS-TOOLS/songs-protocol/blob/2b3d493a24409e0863ac4da52992be0bb3c76323/contracts/protocol/NonUpgradable/DistributorWallet.sol#L217-L231)

### Langflow

- [`src/backend/base/langflow/api/v2/schemas.py`](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/backend/base/langflow/api/v2/schemas.py#L105-L335)
- [Shell execution in `src/lfx/src/lfx/base/mcp/util.py`](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/lfx/src/lfx/base/mcp/util.py#L1581-L1626)
- [Command reconstruction in the same file](https://github.com/langflow-ai/langflow/blob/7db38d1abbdc9ae764f8e7e28da64cb5104217f6/src/lfx/src/lfx/base/mcp/util.py#L2141-L2201)
- [Later upstream MCP stdio hardening](https://github.com/langflow-ai/langflow/commit/eba285edf1dd4a33bf23a9cb8113c991fcdf3d1d)

## Interpretation limits

A re-check anchor shows that the cited implementation path exists at that revision. It does not establish the exact revision reviewed during the campaign. Repository history can also change after this register is published. The current default branch, release tags, deployed configuration, and remediation status require separate verification.