# Contributing to Fast OFM

Fast OFM is a research prototype assembled as six implementation repositories.
Before filing
or implementing a change, identify the relevant feature in `FEATURE_INDEX.md`
and its minimum dependency set.

## Change boundaries

- Keep the four headline work packages independently reviewable: motion,
  illumination, RG focus, and stitching/WSI output.
- Preserve the runtime/license boundary between the GPL OpenFlexure adapter,
  PolyForm Noncommercial core process and replaceable LGPL stitching worker.
- Do not describe roadmap work as implemented or clinically validated.
- Preserve OpenFlexure, Klipper, and other upstream notices and licenses.
- Never add patient/specimen identifiers, machine credentials, private network
  details, or laboratory task journals.
- Keep experimental camera-trigger and scanner-sync paths disabled by default.

Code changes belong in the repository that owns the implementation. Update the
feature index and status matrix here when a change affects maturity,
dependencies, evidence, or public claims. Include the exact validation command
and result in every pull request.

Use GitHub Security Advisories instead of a normal issue for sensitive
vulnerabilities; see `SECURITY.md`.
