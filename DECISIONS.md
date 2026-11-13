# DECISIONS

MLED-1 decisions with rationale. IDs use the MLED- prefix to stay distinct from M-1's D-numbers.

## Decided

### MLED-D1. Hardware: ESP32-PICO-MINI-02-N8R2
ESP32-PICO-V3-02 (rev v3.1), 8MB embedded flash, 2MB embedded PSRAM, USB-UART bridge
(port shows as /dev/cu.usbserial-*). Env `apollo_mled1` extends upstream `esp32dev_8M`
(8MB layout tools/WLED_ESP32_8MB.csv, DIO flash). PSRAM is not enabled yet.
Flash at 460800 baud or lower; 921600 fails on this bridge ("Packet content transfer stopped").
Confirmed on hardware 2026-11-13: the full_install image flashes correctly at 460800 after a
full erase. A Chrome tab using Web Serial can hold the port ("Resource busy"); close it first.

### MLED-D2. LED data pin GPIO13 via DATA_PINS
`-D DATA_PINS=13` (guarded in cfg.cpp). LEDPIN no longer exists in 16.x and
DEFAULT_LED_PIN is unguarded, so neither works. GPIO13 is a normal GPIO on ESP32;
it would be a flash pin on ESP32-C3, so the chip must not change family.

### MLED-D3. Setup hotspot "Apollo MLED-1 Hotspot", open (Justin, 2026-11-13)
`-D WLED_AP_SSID='"Apollo MLED-1 Hotspot"' -D WLED_AP_PASS='""'`. Empty password gives an
open AP; a 1-7 character password would make softAP fail (no AP at all). No MAC suffix:
two unconfigured units in range share one name (same tradeoff as M-1 D26).
set.cpp's factory-reset message now names WLED_AP_SSID instead of the literal "WLED-AP".

### MLED-D4. Identity strings
WLED_RELEASE_NAME `Apollo_MLED-1` (never change: OTA validation rejects other names, which
also keeps generic WLED binaries off units unless the user overrides). WLED_REPO
`ApolloAutomation/MLED-1`, WLED_PRODUCT_NAME and SERVERNAME `Apollo MLED-1`.
WLED_BRAND stays `WLED`.

### MLED-D5. Standalone private repo, not a GitHub fork
GitHub forks of public repos are always public. Seeded with upstream history to v16.0.1
(commit 29b389df) so upstream releases merge as normal 3-way merges. Only `main` is pushed;
no upstream branches or tags.

### MLED-D6. Upstream GitHub automation removed
Deleted upstream build, nightly, release, stale, test, usermods, wled-ci and pr-merge
workflows, FUNDING.yml, the release ini template, and the WLED issue templates. Replaced
by apollo-mled1-build.yml and an Apollo issue-template config.

### MLED-D7. Content-creator units, 2026-11-13
First units were flashed from a local build whose firmware files (platformio.ini env,
wled00/set.cpp) are identical to this repo's first Apollo commit. Recipients get the
matching source archive (EUPL-1.2 Art. 5) because the repo is still private.

### MLED-D8. Libraries pinned; factory image built in CI
`[env:apollo_mled1]` restates the inherited esp32dev_8M lib_deps in the same order, with the
three moving references pinned to what the first builds resolved: AnimatedGIF 1.4.7 (was
^1.4.7), ESPAsyncWebServer 39b830c8 (was tag v2.4.2), QuickESPNow 2985a4a6 (was branch
optional-debug). Upstream maintainers can no longer change Apollo builds without a merge.
On every upstream sync, diff the inherited list (`pio project config`) and re-pin.
CI also merges the factory image (MLED-1_full_install.bin, bootloader 0x1000, partitions
0x8000, boot_app0 0xe000, app 0x10000; DIO, 40m, 8MB) and uploads it with SHA256SUMS.txt.

## Open decisions (before the first retail release)

- LED defaults: LED type, colour order, default pixel count, and ABL milliamps from the PSU
  rating (currently stock: WS2812B, 30 LEDs, 850 mA).
- AudioReactive: compiled in (inherited from esp32dev_8M). It links arduinoFFT (GPL-3.0).
  Drop it if the board has no microphone, or keep it and add the GPL written offer.
- Release tag scheme: Home Assistant (python-wled) treats any tag containing "b" as a beta
  and expects x.y.z (optional v prefix and -suffix), so M-1's m1-bN style does not fit.
  The tag must equal the package.json version, and the OTA asset should be named
  WLED_<ver>_Apollo_MLED-1.bin. Add a tag ruleset once the scheme is chosen.
- WLED's in-device Auto update reads api.github.com releases for WLED_REPO (404 while the
  repo is private) and downloads through download.wled.me, which appears to serve only the
  wled/WLED fork network (probed 2026-11-13). Patch update.htm to Apollo's Pages, or hide
  the button.
- Captive portal: keep upstream behaviour (popup guides setup) or port M-1 D22.
- apBehavior and a recommended AP password or settings PIN after setup.
- mDNS prefix (needs M-1's WLED_MDNS_PREFIX patch) and PSRAM enablement.
- Name clearance for "MLED-1"; EU RED / EN 18031, UK PSTI, CA SB-327 review of the open AP.
- Attribution: About-page line in settings_sec.htm, THIRD_PARTY_NOTICES per release,
  Timezone library (CC BY-SA 3.0) licence question for counsel.
- Factory QC script: read /json/cfg and assert pin 13, SSID and open AP without saving
  settings; log MAC, firmware SHA-256 and result per unit.
- Launch: make repo public, Pages manifest, installer devices.json entry, FlashScripts
  pinned URL, Firmware Vault archive per release.
