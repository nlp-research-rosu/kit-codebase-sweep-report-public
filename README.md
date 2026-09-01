# KIT-Assisted Open-Source Codebase Sweep

This repository contains the technical report for an approximately two-week, KIT-assisted security review campaign across 158 assigned open-source repository entries.

The report covers campaign scale, review method, human supervision, evidence limits, and four case studies backed by pinned review commits and retained KIT sessions.

- [Read the report](REPORT.md)
- [Review the source anchors](SOURCES.md)

## Detailed case studies

- [Langflow MCP command validation bypass](case-studies/langflow/ANALYSIS.md)
- [Mastra default-auth custom route bypass](case-studies/mastra/ANALYSIS.md)
- [Daytona stale public-preview authorization](case-studies/daytona/ANALYSIS.md)
- [Onyx generated-artifact authorization gap](case-studies/onyx/ANALYSIS.md)

## Evidence provenance

- The approximately two-week duration is based on David Bucur's recollection. It is an estimate rather than a timestamped campaign metric.
- All four case studies come from David's KIT runs because their agent sessions, source revisions, and evidence packages were retained. For the other owners, only the campaign table was available. Their results contribute to the campaign totals but are not used as technical case studies.

This repository includes the report, detailed technical analyses, and public source references. It excludes the internal campaign table, agent transcripts, scratch files, and proof-of-concept payloads. Each case study describes the code at the commit listed in `SOURCES.md` and does not claim that current releases remain vulnerable.
