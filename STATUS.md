# Fast OFM capability status

Target release: `v0.3.0-prototype.1`
Status date: 2026-09-29  
Publisher: Alexander Fridman  
Product status: research prototype; not clinically validated

This is the canonical maturity statement for the release. Public READMEs,
release notes and articles must not claim more than this file records.

## Status vocabulary

- `prototype-verified` — exercised on the accepted physical prototype.
- `offline-verified` — replayed or benchmarked without claiming current
  hardware use.
- `test-verified` — covered by automated tests but not accepted on hardware.
- `experimental-disabled` — source exists, but it was disabled in the accepted
  prototype run.
- `roadmap` — design direction, not a release capability.

## What the prototype demonstrably does

| Package | Capability | Status | Evidence boundary |
| --- | --- | --- | --- |
| Motion | Drive the OpenFlexure XYZ stage through Moonraker/Klipper on an MKS Robin Mini V2.0 | `prototype-verified` | Accepted physical scan; commanded position only, with no encoder or verified homing |
| Motion | Apply configured travel, speed, acceleration and Z-segment limits and wait for motion completion | `prototype-verified` | Runtime integration and focused automated tests |
| Illumination | Set RED/GREEN/WHITE brightness with an Arduino-compatible controller and gate channels from the MKS board | `prototype-verified` | Arduino v4 runtime plus observed live gate assignments |
| RG focus | Calibrate and measure red/green focus displacement, apply bounded Z correction and reject low-quality measurements | `prototype-verified` | Accepted calibration/focus workflow and automated tests |
| RG focus | Recover useful tissue patches outside the central crop and fall back to WHITE autofocus when RG cannot produce a valid result | `prototype-verified` | Runtime implementation and focused regression tests; accepted reference scan happened to require zero WHITE fallbacks |
| Focus map | Build sparse focus anchors and predict Z between measured locations | `prototype-verified` | Accepted scan saved 30 RG-focused and 67 predicted-Z fields |
| Scanning | Execute bounded tissue-aware stop-and-shoot snake/spiral acquisition with telemetry | `prototype-verified` | Reference run visited 132 fields and saved 97 in 572 seconds |
| WSI output | Produce pyramidal OME-BigTIFF suitable for QuPath review | `prototype-verified` | Reference output opened successfully in QuPath |
| Stitching | Reuse a bounded FFT spectrum cache and apply bounded render acceleration | `offline-verified` | Saved-scan benchmark/replay only |
| Stitching | Correlate and place disconnected scan regions using stage coordinates and local component alignment | `offline-verified` | Saved-scan replay and automated tests only |
| Operator UI | Control the integrated prototype's stage, illumination, focus and scan operations | `prototype-verified` | Accepted prototype workflow |

## Accepted reference run

The current evidence run is
`pathologist-spiral-15mm-4k-simrg-20260909_0001`:

- 132 fields visited;
- 97 fields saved and 35 background fields skipped;
- 30 saved fields used RG autofocus;
- 67 saved fields used predicted Z;
- zero WHITE-autofocus fallbacks in this particular run;
- acquisition time: 572 seconds;
- adaptive planner limit: 7.5 mm from the scan centre;
- retained field-centre span: 8.808 × 13.200 mm;
- calibrated axis-aligned retained-field bounding footprint: approximately
  10.108 × 14.179 mm (not a claim of contiguous rectangular coverage);
- camera: Raspberry Pi HQ Camera / Sony IMX477;
- tile capture: 4056 × 3040 JPEG, quality 95, chroma subsampling disabled;
- acquisition: host-orchestrated stop-and-shoot, without hardware camera trigger.

The `15mm` text in the historical scan identifier denotes the planner's 15 mm
diameter limit; it is not a claim that a complete 15 × 15 mm rectangle was
scanned. These figures describe one prototype run. They are not throughput,
diagnostic accuracy or reliability guarantees.

## Measured benchmarks and extrapolations

The following timings deliberately separate completed measurements from
engineering estimates:

| Result | Time | Classification |
| --- | ---: | --- |
| Accepted adaptive-spiral acquisition; retained-field bounding footprint approximately 10.108 × 14.179 mm | 572 s / 9:32 | measured on the physical prototype |
| Separate 6 × 14 stop-and-shoot acquisition; 84 addressed positions | 294.6 s / 4:54.6 | measured on the physical prototype |
| Planned 14 × 18 route; bounding footprint approximately 15.613 × 15.004 mm | 883.8 s / about 14.7 min | extrapolated from the 6 × 14 run by position count |
| Optimized OME-only stitch of a separate 91-tile 4K dataset | 150.28 s / 2:30.28 | measured on the x86 server with a warm correlation cache, 3 workers and a 4 GiB cache budget |
| Planned approximately 15 × 15 mm stitch, warm cache | 346.4 s / 5:46.4 | extrapolated by the 2.3051 footprint-area ratio |
| Planned approximately 15 × 15 mm stitch, first cold run | about 486 s / 8:06 | rough extrapolation adding measured correlation cost scaled from 158 to 472 expected pairs |

The 91-tile benchmark input footprint was approximately 10.1081 × 10.0543 mm
and used a SATA Samsung 850, not an NVMe device. The acquisition and stitching
estimates must not be added and presented as a measured end-to-end result.

## Important limitations

- The software and hardware are a research prototype and are not clinically
  validated or intended for diagnostic use.
- The accepted stage reference used an operator-defined local zero. Homing and
  encoder feedback were not verified.
- The saved camera images came from a rolling-shutter IMX477 workflow, not the
  planned 4K global-shutter camera.
- Motion, lighting, camera capture and focus updates were coordinated by the
  host in stop-and-shoot mode.
- The accepted scan did not exercise WHITE fallback even though the fallback
  path exists and has focused test coverage.
- The stitching accelerations and disconnected-region support currently have
  offline evidence, not a separate physical acceptance run.
- The accepted runtime profile uses observed WHITE gate `PB14`; the older
  `PB13` record remains only as a clearly labelled, unverified legacy candidate.
- The accepted illumination reference is deployed Arduino configuration v4;
  later source candidates are separated and make no release claim.
- The owner records the measured scan as Raspberry Pi 4 and the current
  deployment as Raspberry Pi 5. Exact board, OS image and kernel identity and a
  matched Pi 5 timing must still be recaptured. Clean ARM64 container
  validation does not replace or imply Raspberry Pi OS validation.

## Experimental and roadmap work

| Capability | Status | Release treatment |
| --- | --- | --- |
| MCU scanner/frame synchronization prototype | `experimental-disabled` | Source may be retained in a clearly isolated, disabled-by-default area; it must not be described as part of the accepted scan |
| 4K global-shutter camera with hardware trigger | `roadmap` | Architecture and hardware-selection work only |
| Continuous-motion capture coordinated on the MKS/Klipper timeline, with a companion MCU fallback | `roadmap` | Architecture only; a greater-than-2× throughput gain is a target hypothesis, not a measured release claim |
| Real-time focus-surface updates during continuous serpentine motion | `roadmap` | Research plan derived from the current sparse focus map |

## Traceability

- Frozen source identities and runtime facts: `RELEASE_MANIFEST.yaml`
- Detailed feature/source/test mapping: `FEATURE_INDEX.md`
- Source-only validation and deferred gates: `VALIDATION.md`
- OpenFlexure provenance and modification boundaries:
  [`UPSTREAM.md`](https://github.com/FAST-OFM/openflexure-wsi/blob/v0.3.0-prototype.1/UPSTREAM.md)
  and [`MODIFICATIONS.md`](https://github.com/FAST-OFM/openflexure-wsi/blob/v0.3.0-prototype.1/MODIFICATIONS.md)

If evidence changes, update this file and the release manifest together before
changing any public claim.
