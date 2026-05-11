---
applyTo:
  - "**/gitpcm*/**"
  - "**/PCM/**"
  - "**/Pcm*.cs"
  - "**/*Pcm*.cs"
  - "**/gitpcm*.sql"
---

# PCM Template-Override Pattern

When reviewing changes in PCM (Product Content Management) code:

**AP-04 🟡 Template-Override violation.** Lion's PCM has a base-template + override pattern: `gitpcm00` is the base template, `gitpcm20` is the override layer. Flag changes that:

- Modify `gitpcm00` business fields without corresponding `gitpcm20` override logic
- Modify `gitpcm20` (override) but ignore the base spec in `gitpcm00`

Exception: SELECT-only reads, master-data maintenance paths (e.g. `MasterData*`/`Template*Maintenance*`).

Phrasing example: "🟡 Should-fix — This change touches the PCM master template without the matching override path. Per-batch customizations may silently break. Consider adding the corresponding `gitpcm20` override logic, or note in the PR description why the override step is intentionally skipped."

Keep severity at 🟡 unless the change also crosses a 🔴 anti-pattern.
