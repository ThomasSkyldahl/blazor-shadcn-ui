# Changelog

All notable changes to this project will be documented in this file.

## 2026-5-6 — Filter, CurrencyInput & Chart Bug Fixes

> **Release: `v4.0.9`**  
> **Fixes apply to `NeoUI.Blazor`.** No breaking changes.

---

### 🐛 Fix — `Filter`: `DateTimeOffset` parsing in `FilterValue`

`Value` in `FilterValue` was parsed as `DateTimeOffset` without consideration for time zone context. This caused date values without an offset (e.g. from JSON deserialization) to be treated as UTC, leading to incorrect comparisons.

**Fix:** 
- Added `Text.ToDateTimeOffset()` helper on the C# side for consistent conversion.
- This method attempts to parse the value as `DateTimeOffset` first, falling back to `DateTime` and `DateOnly` without an offset.
- `TryConvertToDateTimeOffset()` in `FilterExtensions` also updated to use the new parsing logic.

---

### 🐛 Fix — `CurrencyInput`: empty string handling in `OnChange`

`OnChange` in `CurrencyInput` was bypassing the `TryParse` attempt when the input was an empty string (`""`). This caused `Decimal.TryParse` to receive `null`, leading to a format exception. The correct emptying behaviour (setting `Value` to `null`) was also not triggered.

**Fix:** 
- Changed the `string.IsNullOrEmpty(Value)` check to `Value?.Length == 0` to distinguish between `null` and empty string.
- This allows the parser to handle empty strings correctly and set the value to `null` as intended.

---

### 🐛 Fix — `Chart`: invalid URL for external images in data labels

Data label formatter for pie and bar charts was not encoding URLs for external images, leading to broken image links. The formatter now encodes the entire URL using `Uri.EscapeDataString()`.

---

## 2026-4-24 to 2026-5-5 — DataTable & Filter Enhancements

> **Releases: `v4.0.4` → `v4.0.8`**  
> **Affects `NeoUI.Blazor`.** Additive — no breaking changes.

---

### ✨ Enhancement — `Filter`: `DateTimeOffset` support in `FilterExtensions` and `FilterValue`

**Release: `v4.0.8`**

`FilterExtensions` now correctly handles `DateTimeOffset` filtering conditions:

- **Equals/NotEquals:** Whole-day UTC range matching (same as `DateTime`)
- **GT/LT/GTE/LTE:** Exact comparison
- **Between/NotBetween:** Exact comparison on both bounds
- **Implementation:** Added early-exit branch for `DateTimeOffset` before `Convert.ChangeType` (which cannot handle `DateTimeOffset` as it is not `IConvertible`)
- **Helper:** `TryConvertToDateTimeOffset` accepts `DateTimeOffset`, `DateTime`, `DateOnly`, and `string`

`FilterValue.ToDateOnly` now handles `DateTimeOffset` input so date picker chips correctly display `DateTimeOffset` condition values (e.g. from presets).

---

### ✨ Enhancement — `DataTable`: `ToolbarClass` parameter

**Release: `v4.0.7`**

Added `ToolbarClass` parameter to `DataTable` and `DataTableToolbar` to allow overriding toolbar container padding/styling without replacing the entire toolbar template.

```razor
<DataTable Items="users" ToolbarClass="px-0 py-2">
    ...
</DataTable>
```

---

### 🐛 Fix — `SelectionIndicator`: reposition after collapsible expand/collapse

**Release: `v4.0.5`**

When a `SidebarCollapsibleGroup` expands or collapses, its `grid-template-rows` transition shifts items below it. The `MutationObserver` fires at attribute-change time (before layout settles), causing the indicator to land at a stale Y coordinate.

Added `transitionend`/`transitioncancel` listeners on the container, filtered to layout-affecting properties (`grid-template-rows`, `height`, `max-height`). After any such transition, `positionIndicator` is called to snap to the correct final position. Events from the indicator itself are filtered out to prevent self-recursion. Hover state is also protected from interference.

---

### ✨ Enhancement — `DataTable`: `GlobalSearchValue` two-way binding parameter

**Release: `v4.0.4`**

Added `GlobalSearchValue` parameter with `GlobalSearchValueChanged` callback to enable programmatic control and synchronization of the global search input. Complements the existing `OnGlobalSearch` event callback.

```razor
<DataTable Items="users" @bind-GlobalSearchValue="searchQuery">
    ...
</DataTable>

