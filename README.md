# sailfish-rag

Curated SailfishOS and mainline-porting knowledge, kept as structured JSON so
an LLM can retrieve it instead of guessing. This is the public subset of a
larger private working corpus, synced at stop points rather than continuously.

Two folders, because they answer two different questions and a mixed retrieval
would confidently return the wrong one:

- **`SailfishOS/`** is the platform: how the UI is meant to look and behave,
  what a ported app runs into, and what SailfishOS itself needs on a device.
- **`Mainline/`** is the hardware below any OS: kernel, device tree, firmware,
  modem, audio, camera, sensors, power, and the method that found each trap. It
  is written OS-neutral, with the concrete instance beside the general lesson,
  so a port of another system onto the same class of phone can read it.

## How to use it

Point a session at the folder that matches the task, not at the repository. Each
folder has an `AGENTS.md` written to be copied into the root of your own
project, so an agent carries the instructions with it.

- Writing, porting or refactoring an app: `SailfishOS/AGENTS.md` first. It is
  the app-side working practice condensed into one file, and it points at the
  JSON for the detail.
- Refactoring a UI that works but does not belong on the platform: read
  `SailfishOS/design_guide.json`, then check every theme icon name against
  `SailfishOS/system_icons.json` before it is written into QML.
- Porting an app: `SailfishOS/app_development.json`.
- Bringing a phone up on a mainline kernel: `Mainline/AGENTS.md` first, then
  the chapter for the subsystem in front of you, then `Mainline/hazards.json`
  before anything that can brick a device.

## SailfishOS

| file | what it holds |
|---|---|
| `AGENTS.md` | instructions for an LLM coding agent writing, porting or refactoring a SailfishOS app. The practice from [PRACTICE.md](https://github.com/moWerk/llmgd-specs/blob/main/PRACTICE.md) plus the platform and app findings from this folder, condensed into one file to copy into a project root and fill in at section 0 |
| `design_guide.json` | the platform's design and UX specification in 25 sections: principles, the five system views, gestures, navigation, Theme as the single style source, the colour signal, Jolla's own list of porting pitfalls, 85 Silica component names, app and system icons, ambience, Jolla's Definition of Done, one credited community guide, and a derived refactor checklist |
| `system_icons.json` | 717 platform theme icon names by size class, so a session can check that a name exists instead of inventing one. All 717 resolved on a 5.1.0.11 device |
| `app_development.json` | what porting a QML app actually took: Qt 5.6 gaps, the logging trap, the project skeleton, cover, icons, Harbour store rules, tall-screen layout, audio capture, sensors through the Camera permission |
| `mainline_porting.json` | SailfishOS on a mainline kernel: what the ecosystem already provides, and the ngfd, PulseAudio, mce and oFono traps with their fixes |
| `pro1x_device_notes.json` | the F(x)tec Pro1-X as a SailfishOS device: the hybris port that exists, installing it from stock Android, the upgrade path, and the state of mainline kernel support for the Pro1 and the Pro1-X |

## Mainline

This corpus came out of porting SailfishOS 5.2 onto the F(x)tec Pro1-X
(Qualcomm SM6115) on its own 7.3 kernel, with no vendor kernel, no libhybris
and no halium. 270 labelled entries
across fourteen chapter files: 224 confirmed, 30 reported by a named source, 16
inferred.

| file | holds |
|---|---|
| `AGENTS.md` | instructions for an LLM coding agent doing a mainline port. Written to be copied into the root of a port repository and filled in at section 0 |
| `method.json` | how to work: research first, reference ports, diff against a working stack, measure before changing, keep a rollback |
| `hazards.json` | what crashed, froze or could brick a device, and the never-do list |
| `boot.json` | bootloader, extlinux, initramfs, dual boot, rootfs layout, recovery after a crash |
| `kernel.json` | building and configuring a kernel outside a distribution build system, device tree, modules, driver bugs met |
| `firmware_remoteproc.json` | firmware files, remoteprocs (modem, adsp, cdsp), rmtfs, tqftpserv, recovery settings |
| `modem.json` | QRTR and QMI services, SIM provisioning, the data path (IPA, rmnet, QMAP), voice |
| `audio.json` | DSP audio (q6), codecs, UCM2, the sound server and call audio |
| `camera.json` | camss, sensor and lens drivers, libcamera pipeline and software ISP, autofocus, frame rates |
| `sensors.json` | sensors behind a DSP (SSC), hexagonrpc, libssc, the framework adaptors |
| `display_input_power.json` | GPU and display, keyboards and touch, haptics, battery and charger, USB gadget networking |
| `packaging.json` | building userspace for the target OS: SDK, toolchains, packages the OS lacks, version locks across upgrades |
| `userspace_sailfishos.json` | what SailfishOS middleware expects on mainline and what had to change. Another OS gets its own file, never a mix |
| `devices.json` | per-device status matrix with the date of the newest evidence |
| `upstreaming.json` | sign-off and AI-disclosure rules, kernel and device tree submission rules, what to send first, the port's per-patch backlog |
| `references.json` | 69 mainlining guides and Qualcomm documents outside postmarketOS, with owner and licence, lessons paraphrased rather than quoted, and the gaps this corpus can fill |

A `Mainline` entry is `title`, `general` (the lesson for any mainline port),
`instance` (this device, kernel, OS version, numbers), `confidence`, `date` and
`source`.

## Confidence labels

Every state claim carries the release and the date it was measured on.

- `confirmed`: observed on a device or read in the source by the session that
  wrote it.
- `reported`: someone else said it, and they are named.
- `handover`: taken from another session's handover and not re-checked.
- `vendor_doc`: read from the vendor's own documentation, with the page and the
  date it was fetched. Never measured.
- `community_doc`: the same from a named third party, paraphrased.
- `forum_summary`, `inferred`, `recalled`: weakest, and labelled as such.

`vendor_doc` and `community_doc` are deliberately weaker than `confirmed`. A
documentation page states what the platform intends, which is not the same as
what a release does. Findings that turned out wrong are kept with the
correction attached, because the dead ends are the part that saves the next
person time.

## What is held back

The private corpus also holds device identity, per-unit battery history, build
host configuration, machine and infrastructure records, and raw session
transcripts. None of that is here. Publication runs off an allowlist: a file
reaches this repository because somebody listed it, never by appearing in a
folder. Build host names and paths are substituted on every sync, and the sync
refuses to write when a pattern gate matches a known-private literal.

Some `source` fields point at files that stay private. They are kept as written
rather than blanked, so a claim can still be traced if you ask.

This tree is generated. Hand edits here are lost on the next sync, so
corrections go through an issue.

## Sources and credit

The design material is Jolla's: the design and Silica pages on sailfishos.org,
the documentation on docs.sailfishos.org, the Sailfish Icon Reference, the icon
story and template, and the 2015 design blog post by Martin Schüle. One section
is Michal Szczepaniak's app design guide on the VerdandiTeam blog, credited at
the point of use. The mainline material credits the upstream developers and
porters whose work it rests on, by the name they publish under.

Nothing here reproduces Jolla's prose or code blocks. Facts are restated in our
own words, and API, property and icon names are kept verbatim because there is
no other way to state an API. The documentation portion on docs.sailfishos.org
is CC BY-NC-SA 4.0 (the MIT licence GitHub reports for that repository covers
its Jekyll theme, not the documentation); the design and Silica pages state no
licence at all, only "Jolla Mobile Ltd (c) 2025"; the icon template archive is
CC BY 3.0.

## Related repositories

- [moWerk/sailfishos-pro1x-mainline](https://github.com/moWerk/sailfishos-pro1x-mainline):
  the port this corpus came out of, configs and scripts.
- [moWerk/jolla-tablet-2026](https://github.com/moWerk/jolla-tablet-2026):
  Jolla Tablet fixes on 4.6.0.15.
- [moWerk/asteroid-rag](https://github.com/moWerk/asteroid-rag): the same idea
  for AsteroidOS. Separate on purpose, for the same reason the two folders here
  are separate.

## Disclosure

This corpus is written by LLM sessions, steered and corrected by moWerk, and it
says so rather than leaving it to be guessed. Commits carry a two line
[LLMGD](https://github.com/moWerk/llmgd-specs) disclosure. Where a finding was
measured on hardware it says so and names the release; where it was not, the
label says that too.

## Licence

Copyright (C) 2026 Timo Könnecke (moWerk) <mo@mowerk.net>

`SPDX-License-Identifier: GPL-3.0-or-later`

This corpus is free: you can redistribute it and/or modify it under the terms
of the GNU General Public License as published by the Free Software
Foundation, either version 3 of the License, or (at your option) any later
version. See [LICENSE](LICENSE). It is distributed in the hope that it will be
useful, but WITHOUT ANY WARRANTY, without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

The licence covers this compilation and its wording. Individual facts about
SailfishOS, about Qualcomm hardware and about the Linux kernel are not anyone's
to license, and where a finding is someone else's it is named at the point of
use.

If the licence is the wrong fit for how you want to use this, open an issue.
moWerk is willing to change it on request if a real problem comes up with it.
