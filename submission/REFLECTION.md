# Reflection — Anti-pattern Risk Assessment

## Selected Anti-pattern: Small-File Problem

The **small-file problem** poses the highest operational risk to LLM observability at 1B requests/day.

### Why This Risk is Critical

Streaming ingestion naturally creates pathological file layouts. Without OPTIMIZE, **1B requests/day → hundreds of thousands of tiny files**, degrading query performance catastrophically.

**NB2 benchmark proved this:**
- 200 files → 171ms latency
- After OPTIMIZE+Z-ORDER → 28.7ms (**6.0× speedup**)
- File-pruning ratio: **55.0×** — point queries hit 1 of 55 files vs. scanning all

### Operational Impact

- Dashboard SLA violations: tenant-level cost reports breach 5-minute refresh target
- Storage bloat: Parquet footer overhead multiplies per file
- Query inefficiency: file statistics useless with overlapping ranges

### Solution

Run hourly OPTIMIZE+Z-ORDER by (model, user_id) after peak ingestion:
- Target size: keep 50–100 files per layer
- Monitor: alert if file count > 500
- Cost: ~2 min/hour (~0.2% compute)
- Benefit: 6× query speedup → SLA-compliant dashboards

**Conclusion**: Proactive file management is non-negotiable; without it, small-file overhead will cripple production observability within weeks.
