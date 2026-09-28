# Fast OFM licensing boundary

Fast OFM is a source-available research project. Original Fast OFM material is
available for noncommercial use; commercial use requires a separate written
license from the copyright holder.

## Repository boundary

| Repository or component | License | Commercial use |
| --- | --- | --- |
| `FAST-OFM/fast-ofm` original documentation | CC-BY-NC-SA-4.0 | Not granted |
| `FAST-OFM/controller` original software and firmware | PolyForm-Noncommercial-1.0.0 | Not granted |
| `FAST-OFM/controller` original documentation and specifications | CC-BY-NC-SA-4.0 | Not granted |
| Klipper-derived patches and integration source | GPL-3.0-only | Permitted by GPL |
| `FAST-OFM/hardware` original design documentation | CC-BY-NC-SA-4.0 | Not granted |
| `FAST-OFM/hardware` software utilities | PolyForm-Noncommercial-1.0.0 | Not granted |
| `FAST-OFM/openflexure-wsi`, including Fast OFM modifications in that combined work | GPL-3.0-only | Permitted by GPL |
| `FAST-OFM/fast-ofm-core` independently authored runtime and tests | PolyForm-Noncommercial-1.0.0 | Not granted |
| `FAST-OFM/fast-ofm-core` neutral protocol schemas | Apache-2.0 | Permitted by Apache-2.0 |
| `FAST-OFM/fast-ofm-stitching-openflexure` implementation and tests | LGPL-3.0-only | Permitted by LGPL |

The GPL-covered OpenFlexure and Klipper portions cannot carry a noncommercial
restriction. Their presence does not relicense separately distributed,
independently authored Fast OFM material. Conversely, the noncommercial
licenses do not override the GPL for a combined or derivative GPL-covered work.

The GPL server communicates with the noncommercial core as a separate program
using versioned JSON and verified file artifacts. The core invokes the
OpenFlexure-dependent stitching worker as a separately installed, replaceable
executable. Repository separation alone is not treated as the boundary; the
process architecture is tested independently.

`REUSE.toml` files and repository-specific `LICENSING.md` documents are the
machine-readable and human-readable authorities for individual files.

## Commercial licenses

No commercial permission is implied by source availability. A commercial
license for original Fast OFM copyright owned by **Alexander Fridman** must be
granted separately in writing. Neither Alexander Fridman nor Fast
OFM can grant third-party rights.

## Earlier release

The immutable `v0.2.0-prototype.1` snapshots retain the licenses under which
they were issued. This boundary applies from `v0.2.0-prototype.2` onward and
does not attempt to revoke rights already granted for earlier copies.
