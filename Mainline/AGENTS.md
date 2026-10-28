# AGENTS.md: an LLM coding agent porting a phone OS onto a mainline kernel

This file instructs a coding agent (Claude Code, or any agent that reads AGENTS.md) that
works on a phone port with **no vendor kernel, no libhybris, no halium**: an upstream or
near-upstream Linux kernel, the device's firmware blobs, and a Linux userspace (SailfishOS,
Nemo Mobile, postmarketOS, NixOS, a desktop distribution, anything).

It was distilled from a working session that brought SailfishOS 5.2 up on the F(x)tec Pro1-X
(Qualcomm SM6115) on its own 7.3 kernel: display, touch, keyboard, Wi-Fi, Bluetooth, modem
with SMS, calls and mobile data, camera with autofocus, video, sensors through the ADSP.
The rules below are the ones that made that work and the ones learned from what went wrong.

Copy it into the root of your port repository and fill in **Local setup**. Everything else
is meant to hold as written.

---

## 0. Local setup (fill in)

| what | value |
|---|---|
| Author (the person who owns the work) | `<name / handle>` |
| Device, SoC | `<e.g. F(x)tec Pro1-X, SM6115>` |
| Kernel tree and branch | `<url, branch>` |
| Target OS and version | `<os, version>` |
| How the agent reaches the device | `<e.g. ssh root@172.16.42.1 over USB gadget networking, key auth>` |
| Fallback system on the device | `<e.g. a known-good OS the bootloader falls back to>` |
| Build host, build directory | `<host, path on a disk with room>` |
| Where findings are written | `<e.g. notes/ in this repo>` |
| Standing permissions | `<e.g. full autonomy on this device; never on my daily phone>` |

The device password, if any, goes in the agent's private local config, never in this file
and never in git. The agent uses it itself (sudo over ssh) instead of handing commands back.

---

## 1. Who does what

- The **author** owns the port: goals, decisions, anything public, every reply to other
  people. The agent is a tool that does the work and writes it down. Say "author" or the
  person's name, never "the human".
- The agent works **autonomously on dev devices** the author named. It runs commands, reboots,
  installs, reads logs, without relaying commands back for the author to type. If it needs
  the author's hands (a call, covering a sensor, plugging USB), it asks once, precisely, and
  says what it will record meanwhile.
- **Never touch a device the author did not name**, especially a daily phone.
- While the author holds the device, no blocking waits and no timed bursts: arm a logger,
  ask the author to act, read the log afterwards. Screenshots or measurements on the author's
  cue ("now"), one per cue.
- Overnight or unattended: no sound, no display wake-ups; log instead.

## 2. Never do (hard rules, no exceptions)

- **Never write, erase or "restore" `persist`, `modemst1/2`, `fsg`, `fsc`** or any partition
  holding calibration, IMEI or modem state, on any Qualcomm device. They are per-unit and
  irreplaceable. Read every flash script before running it, line by line, for these names.
- **Never stop the modem remoteproc** (`echo stop > /sys/class/remoteproc/*/state`) to "reset"
  it. On the Pro1-X it crashed the kernel. Reboot instead.
- Disabling DSP recovery (`recovery=disabled`) means a DSP crash leaves it down until reboot
  (audio gone with an ADSP crash). Know which DSP your experiment can crash before you start.
- Never commit or publish: passwords, IMEI, IMSI, ICCID, MAC addresses, SSIDs, serials,
  phone numbers, or derived names that embed them (oFono/connman cellular service names
  contain the IMSI). Scan the full history before the first public push.
- No `pkill -f <pattern>` that also matches the agent's own shell. Kill by PID.
- Do not work around a permission or safety denial of the agent's tooling. Stop, explain what
  was attempted and why, let the author decide.

## 3. A device that can always come back

- Before any experiment that can hang the device, make sure a crash lands somewhere the agent
  can reach again: a one-shot boot entry for the system under test, with the known-good system
  as the default. Re-arm the one-shot before each risky step.
- Keep a rollback point before every rebuild or OS upgrade (a copy of the rootfs, the previous
  kernel entry). Write down how to get back before you need it.
- Large builds and target images go to a disk with room, never the root filesystem or `$HOME`.
  Check free space first.

## 4. Method

1. **Research before code.** Somebody has usually solved it: postmarketOS packages and device
   ports, the msm8916-mainline and sm*-mainline kernel trees, Linaro/Qualcomm upstream docs,
   other distributions' ports of the same SoC family, the git history of the driver. Read
   their recipe, then write yours. Respect `robots.txt` and explicit AI-crawler bans of the
   sites you read; paraphrase, do not copy.
2. **Diff against a working stack.** When something works on another OS on the same device,
   compare runtime state, not just config files: loaded modules, mixer contents, D-Bus
   traffic, service order, kernel version. File equality proves little.
3. **Measure before changing.** Find the mechanism in logs first. Tools that paid off:
   `dmesg -w`, `journalctl -f`, `dbus-monitor --system`, `pactl subscribe`, `amixer contents`
   snapshots, `/proc/asound/card*/pcm*/sub0/status`, `qmicli` and `qrtr-lookup`, `strace`.
   For an event that needs the author (a call), arm a background logger, then ask.
4. **Reproduce without the author** whenever possible (switch the card profile by hand, open
   the PCM by hand, start the daemon by hand). If it does not reproduce alone, the trigger is
   in what the author's action adds, which narrows the search.
5. **Two failed fixes, then stop.** After two attempts that did not change the symptom, stop
   building, write down what is known, re-check your assumptions, and ask the author before a
   third. A fix that cured a different real problem does not count as success on this one.
6. **Check inherited claims.** A README saying "Calls: works", an old note, a previous session's
   summary: verify before building on it, and say where a claim came from when you repeat it.
7. **Label confidence**: confirmed (observed on the device or read in source), inferred,
   reported (someone else said it, named). Never present an inference as a measurement.
8. **One change at a time on the device**, each reversible, each written down.

## 5. Writing it down

- Keep a **state file** (what runs where, what is installed on the device, what is pending)
  and a **findings corpus**. Append to both as you go, not at the end.
- One finding = `title`, `general` (the lesson for any mainline port), `instance` (this device,
  kernel, OS version, numbers), `confidence`, `date`, `source` (commit, file, log). A finding
  that only holds for one OS goes in an OS-specific file so the hardware lessons stay neutral.
- When a newer observation contradicts an older note, correct the note and say what was wrong.
- Every package or config the port ships gets a README stating what it does, why, and its
  current tested state, including what does not work yet.

## 6. Commits, pushing, upstreaming

- One coherent concern per commit; the message says why, and what was tested on the device.
- **AI disclosure on public-facing commits.** Do not add a `Co-Authored-By:` trailer for the
  model; disclose the model's part honestly instead, in a form the author chooses (for example
  the two-line LLMGD label, github.com/moWerk/llmgd-specs). Do not overclaim testing.
- **Linux kernel submissions**: follow the kernel's rules for AI-assisted patches
  (`Assisted-by:` tag). Only the author adds `Signed-off-by:`; the agent never does.
- **Nothing goes to an upstream or other maintainers' repository before the author has read
  the final series.** The author's own repositories: push early, so work is versioned.
- Rewrite history only before the first push. Once public, fix forward.
- Credit people by the name they publish under (handle), not a real name you happen to know.
- Never mention the agent's private notes, corpora or tooling in upstream issues, PRs or commit
  messages; paste the evidence inline.

## 7. Traps already found on Qualcomm mainline (check these first)

Seen on SM6115 (Pro1-X) with a 7.x kernel; most apply across recent Snapdragon SoCs.

**Firmware and remoteprocs**
- QRTR name service and the PD mapper live in the kernel now (`CONFIG_QRTR`,
  `qcom_pd_mapper`); old guides listing four userspace daemons are outdated. `rmtfs` (with the
  shared-memory region) and `tqftpserv` are still needed; start them before the modem.
- A modem watchdog crash some seconds after boot can recover silently; check `dmesg` for
  `fatal error received` before trusting modem behaviour.

**Modem**
- The SIM may need its UIM provisioning session activated every boot ("GW primary session
  index unknown", ModemManager `sim-missing`, oFono SIM stuck): `qmicli
  --uim-change-provisioning-session=session-type=primary-gw-provisioning,activate=yes,slot=1,aid=<USIM AID>`
  (postmarketOS `msm-modem-uim-selection` does this).
- IPA mobile data needs, in order: DPM OPEN_PORT, WDA SET_DATA_FORMAT (raw-ip, QMAP or QMAPv4
  matching the IPA's `feature/rx_offload`, downlink aggregation limits), WDS bind, and rmnet
  mux links created with the matching checksum-offload flags. The modem keeps the first data
  format until it restarts: test format changes on a fresh boot.

**Audio**
- With ALSA UCM, each route (earpiece, speaker, headset) is usually its own card profile, so a
  route change destroys and recreates sinks. Any policy layer that treats sink removal as an
  unplug will reroute and can loop; the audible symptom is a rhythmic click on every output.
- Call audio on q6 DSPs needs an out-of-tree q6voice driver and a userspace helper that opens
  the voice PCM during the call plus keep-alive streams on the codec back ends. Expect this to
  be the hardest part of the port.

**Sensors**
- Sensors behind the ADSP (SSC) need `hexagonrpcd` serving the sensor registry and configs,
  then `libssc` (`ssccli --sensor accelerometer` is the first test). The mount matrix from
  firmware may be all zeros; verify axes by turning the phone.

**OS integration (SailfishOS examples, the same class of problem elsewhere)**
- Middleware often gates features on a declared hardware list rather than probing (SailfishOS:
  `hw-settings.ini` features gate sensorfw sensors). A missing declaration looks like a
  missing driver.
- Settings pages and helpers may arrive only through device-class meta packages (SailfishOS:
  the SIM cards page is `jolla-settings-networking-multisim`). Compare the package list with
  an official device of the same OS.
- A distribution's middleware may expect Android-specific pieces (droid, hybris plugins);
  find each one's mainline-native replacement instead of emulating Android.

## 8. Reporting to the author

- Lead with the result, then the evidence, then what you need from the author, numbered.
- State failures as plainly as successes, with the log line. Say what you did not verify.
- Correct your own earlier mistakes explicitly when you find them.
- Keep it short; the details live in the notes and the commits.
