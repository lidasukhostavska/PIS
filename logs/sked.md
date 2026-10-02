## SKED Dialog Summary

- **Identified Uncertainties:** Addressed unresolved requirements from Lab 1 regarding `Stock Status Rules` thresholds, `Expiration Date` handling, and overall system boundaries.
- **Decisions Made:**
  - Defined strict quantity thresholds: `Out of Stock` (0), `Running Low` (1-10), `Available` (11+).
  - Defined Expiration Date logic: Products receive the `Expired` flag at 00:00 the day after expiration. Expired products are hidden from the Guest Public Catalog but remain visible to Admins with a warning.
  - Excluded out-of-scope features: Explicitly rejected shopping carts, checkout, payment processing, user registration, and multi-location inventory.
- **Corrected Assumptions:** Clarified that the `Expired` state is an independent flag, not a replacement for the primary `Stock Status`.