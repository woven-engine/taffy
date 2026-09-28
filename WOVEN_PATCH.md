# Woven layout patch

Based on Taffy v0.10.1, upstream commit
`639f0acdd20f914f847abffbeaa903d66cae4882`.

The patch preserves the public API, floating-point comparisons, layout output,
and cache first-match ordering:

- Size-cache lookup visits only insertion slots that can match the query's
  known dimensions and available-space classes. Queries with two known
  dimensions retain the original complete search order.
- Flex size-only queries return once both container dimensions are determined.
  A definite cross size permits a return after main-size determination;
  otherwise line cross-size calculation and stretch still execute. Child
  stretching, margins and alignment continue to execute for actual layout.
- Cache access wrappers and the lookup itself force cross-crate inlining so
  constant query modes and known-dimension branches can be eliminated at callers.
- Flex item collection reserves the container's bounded child count after the
  first in-flow child is found, avoiding repeated vector growth and copies.
  Hidden/absolute-only containers retain the no-allocation path. Temporary
  capacity includes any filtered siblings; item order and layout stay unchanged.
  Already resolved padding and border values are reused within each item.

Regression tests compare cache lookup with the original implementation across
replacement and clearing, and compare size-only and complete layout results
across direction, wrapping, box sizing, padding and cross-size constraints.
The existing layout fixture suite remains unchanged.

This patch is internal to the native UI layout dependency. Woven's retained
layout run condition cannot modify these private cache and flex calculations.
Keep the fork until an upstream compatible release supplies equivalent behavior;
update the immutable dependency and its fork lock together.
