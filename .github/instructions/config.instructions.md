---
applyTo:
  - "**/appsettings*.json"
  - "**/web.config"
  - "**/*.config"
  - "**/featureflags*"
  - "**/feature-flags*"
---

# Implicit Switch and Feature Flag Changes

When reviewing changes to config or feature-flag files:

**AP-05 🟡 Implicit switch changed without explanation.** Lion has known implicit production switches (e.g. `ToVector=0` in the search system) where flipping the value silently changes production behavior. Flag changes that:

- Modify boolean constants or feature flags like `ToVector`, `IsEnabled`, `UseV2` without a rationale in the commit message or PR description
- Modify switch values in `appsettings.json` / `web.config` while the same PR also contains business-logic changes (mixed concerns)

Exception: pure-config PRs where the commit message clearly states "enable/disable feature X", or the changed switch has a corresponding unit test that asserts the new value.

Phrasing example: "🟡 Should-fix — This PR changes an implicit switch (e.g. `ToVector`) alongside business logic. Without a rationale, production behavior may shift unexpectedly. Consider adding a short rationale in the PR description, or splitting the config change into a separate PR."

If the switch is in a known-sensitive area (search ranking, payment routing, tour-finalization), suggest the reviewer also flag this for the owning squad before merge.
