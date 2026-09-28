# Reproducing the prototype checkpoint

Reproduction is staged so that image processing can be evaluated without
hardware and hardware is not energized before configuration is understood.

## 1. Inspect evidence boundaries

Read `STATUS.md`, `DISCLAIMER.md` and `RELEASE_MANIFEST.yaml`. Do not substitute
candidate pins, future synchronization code or roadmap hardware for the
accepted stop-and-shoot configuration.

## 2. Offline software replay

1. Install the documented Python and Node versions in an isolated environment.
2. Run the `openflexure-wsi` Python and web tests.
3. Use the sanitized motion-profile fixture in
   `openflexure-wsi/tests/fixtures/prototype_motion_profile.json` for controller
   and scan-planning tests. It contains no specimen imagery.
4. Run RG-focus, focus-map, OME-BigTIFF and stitching tests against their
   synthetic fixtures. A privacy-reviewed specimen-image replay dataset is not
   part of this source-only release candidate.
5. For a locally supplied saved scan, stitch with the documented CPU and RAM
   limits and verify the resulting OME-BigTIFF in QuPath. Do not present that
   local data as a redistributable release fixture until its privacy review is
   recorded.

## 3. Controller tests without outputs

1. Run the `controller` pure-Python simulator and protocol tests.
2. Build the Arduino source for the named board without uploading it.
3. Review the exact MKS/Klipper base and example configuration.
4. Confirm experimental scanner-sync features are disabled by default.

## 4. Bounded hardware bring-up

Follow the `hardware` repository's safe order:

1. identify exact board revisions and power domains;
2. verify current limits, polarity and output reset states;
3. validate communication with motors and illumination disabled;
4. check illumination gates into a safe measurement load;
5. establish a local mechanical reference and conservative software bounds;
6. enable one bounded subsystem at a time;
7. run a small focus/capture smoke test before any specimen scan.

The accepted prototype has no verified physical homing or encoder feedback.
Never assume its software coordinates survived a restart or motor release.

## Release gate

The private source release candidate can be tagged after clean installation,
synthetic/offline tests, source/privacy scanning and reproducible firmware
builds pass. A visibility change, specimen-data publication, or hardware
acceptance claim additionally requires a privacy-reviewed replay dataset and a
bounded owner-run hardware smoke test with its exact artifacts recorded.
