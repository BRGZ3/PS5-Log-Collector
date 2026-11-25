# PS5 Log Collector

[English](README.md) · [Русский](README.ru.md)

<p align="center">
  <img src="ps5logcollector.png" width="1664" height="833" alt="PS5 Log Collecctor">
</p>

A portable Windows application, plus an experimental native macOS ARM client,
for collecting logs from a jailbroken PlayStation 5 — without Netcat or
command-line tools.

## Highlights

- Connects directly to `klog.elf` / `klogsrv` over raw TCP.
- Provides a separate Game DEBUG session for [`ps5debug-NG` v1.3.0](https://github.com/OpenSourcereR-dev/ps5debug-NG).
- Provides a KStuff Metrics session for metrics-enabled `kstuff.elf` plus `debug-reader.elf`.
- Adds an in-app **SM+** workspace for downloading the saved ShadowMountPlus `debug.log`.
- Keeps the ELF-loader connection open after upload to capture the payload's stdout/stderr stream.
- Detects the foreground game and records structured trap/signal events with thread and register context.
- Optionally sends a user-selected ELF payload to an active console loader on port `9021`.
- Uses the fixed KLOG port `3232`; only the console host is entered in the main row.
- Displays incoming KLOG data in a responsive live viewer.
- Adds an ISO 8601 timestamp with milliseconds and the PC time-zone offset to every line.
- Saves the complete session from the moment the connection was established.
- Clears the visible log view without stopping capture; the complete session remains available to save.
- Preserves partial lines when capture is stopped or the remote side disconnects.
- Switches between English and Russian without restarting.
- Remembers the selected language and last endpoint.
- Ships as one self-contained EXE, with no separate .NET installation.
- Includes a separate self-contained macOS ARM build with KLOG and SM+ only.
- Does not bundle or invoke Netcat or Telnet.

Example output:

```text
[2026-09-13T22:42:15.123+03:00] kernel: example message
```

## Requirements

- A jailbroken PlayStation 5 capable of running `klog.elf` or a compatible `klogsrv` payload.
- Game DEBUG additionally requires `ps5debug-NG` v1.3.0 running on the console.
- The PC and console connected to the same local network.
- Windows 11 x64 is the primary supported target.
- Windows 10 22H2 x64 is best-effort and should be tested with the current release.
- macOS 12+ on Apple silicon is supported by the experimental KLOG + SM+ build.

## Quick start

1. Download the latest Windows EXE from the [Releases page](../../releases/latest).
2. Start the downloaded `PS5LogCollector-<version>-win-x64.exe`.
3. Select **KLOG**.
4. Enter the console IP address.
5. Select **Start KLOG**. The app first connects to an already-running KLOG stream; if it is not active, it sends the built-in `klogsrv-ps5.elf` v0.9 through a compatible loader on port `9021` and connects automatically.
6. If KLOG is already active and you do not want to send a payload, select **Connect only**. To use another ELF, expand **Use another ELF for this run**; the override is confirmed explicitly and resets when the app closes.
7. Use **Stop** to end capture and **Save log** to export the complete session as UTF-8 text.

Saving is also available while capture is active. Cancelling the Save dialog does not interrupt the connection or discard the session.

The Windows client hides the experimental **DEBUG** workspace by default. Enable
**Settings → Experimental DEBUG → Show DEBUG in the navigation** when you need
the KStuff Metrics or Debugger Events tools.

### Settings and Saved Logs

**Settings** lets you choose the default session folder, enable or disable
automatic session saving, switch the interface language, and review the built-in
file-naming rules. Automatic saving is disabled by default and can be enabled
explicitly.

**Saved Logs** indexes captured files from the configured folder. It provides
search, source filters, details, preview, export, and editable notes. Metadata
includes the capture date, duration derived from log timestamps when available,
detected game/PPSA and Title ID, payload and firmware parsed from the log, and
file size. Manual Save actions remain available regardless of the auto-save
setting.

### macOS ARM — KLOG + SM+

1. Download the macOS ARM archive (`PS5LogCollector-<version>-mac-arm64.zip`) from the release assets.
2. Unzip it and open **PS5 Log Collector.app**. The app is self-contained and does not require a separate .NET installation.
3. In **KLOG**, enter the PS5 IP address and select **Start KLOG**. The app first tries an already-running KLOG stream; if it is unavailable, it sends the built-in `klogsrv-ps5.elf` v0.9 through an active ELF loader on port `9021`, then connects to KLOG on `3232` automatically.
4. To use another payload, expand **Use another ELF for this run**, choose an ELF, and confirm the transfer. The choice is temporary and is not saved.
5. Use **SM+** to fetch the saved `debug.log` through the ShadowMountPlus HTTP API, with the optional FTP fallback.
6. **Settings** lets you choose the session folder and enable optional automatic saving (off by default). **Saved Logs** indexes KLOG and SM+ files from that folder for search, preview, metadata, notes, and export.

This build intentionally does not include DEBUG, DECI5, or parallel capture. For the one-button KLOG flow, a compatible ELF loader must already be active on the PS5 and listening on `9021`. If `klogsrv` is already active, use **Connect only**. The macOS artifact is currently unsigned, so Gatekeeper may require an explicit **Open** action on first launch.

If macOS reports a startup failure before the window appears, diagnostics are written to `~/Library/Logs/PS5LogCollector/startup.log`; if the home path is unavailable, use `/tmp/PS5LogCollector/startup.log`.

### Game DEBUG — KStuff Metrics

1. Select **DEBUG → KStuff Metrics** and enter the console IP address.
2. Optionally choose the matching metrics-enabled `kstuff.elf` and send it through the loader on port `9021`.
3. Choose `debug-reader.elf` and select **Start metrics capture**.
4. The app sends the reader, closes only the upload direction, and reads its stdout/stderr over the same `9021` connection.
5. Select **Stop capture** before closing the payload, then save the complete session.

The reader prints periodic snapshots of kstuff metrics, kstuff message/word logs, syscall statistics, and ioctl-com statistics. The matching kstuff build must expose the shared observation area; the application cannot make an ordinary kstuff build produce these metrics.

### Game DEBUG — Debugger Events

Select **Debugger Events** for the existing `ps5debug-NG` workflow: command port `744`, reverse event port `755`, and structured trap/signal packets. This is a separate diagnostic source and is not the same as KStuff Metrics stdout.

KStuff Metrics captures the reader payload's stdout/stderr, not ordinary game stdout. Classic application/SDK stdout is expected to require DECI5 TTY and remains a later transport. KLOG, DEBUG, and SM+ run as separate sessions.

### SM+ — ShadowMountPlus ([API reference](https://github.com/drakmor/ShadowMountPlus/blob/1.7/docs/api.md))

1. Open **SM+** in the left navigation and enter the PS5 IP address.
2. Select **Fetch debug.log**. The app first tries the ShadowMountPlus HTTP API on port `10101`.
3. If the API is unavailable, expand the FTP fallback and use the console FTP port (commonly `1337`) and the saved path `/data/shadowmount/debug.log`.
   The FTP user field is intentionally blank by default; fill it only when the console FTP service requires a username.
4. Review the downloaded file, then use **Save log** to write the original text without adding timestamps or other lines.

This is a one-shot download of the saved file; live ShadowMountPlus monitoring is intentionally not part of this phase. The HTTP API is normally loopback-only, so ShadowMountPlus must be configured to bind its API to the local network (for example, `api_bind_address=0.0.0.0`). The API has no authentication, so expose it only on a trusted LAN. It may return only the tail when the file is larger than its response limit, and the status bar reports that condition.


## Connection troubleshooting

- **Connection refused:** confirm that `klog.elf` is active and listening on the selected port.
- **Timed out:** verify the PS5 IP address, local network connectivity, and firewall rules.
- **No new messages:** KLOG may be quiet until the console produces new kernel output.
- **Connection closed:** restart the payload if necessary, then connect again manually.

The application uses the same raw TCP stream as:

```shell
nc <PS5_IP> 3232
```

Telnet mode is intentionally not implemented because the upstream service exposes a raw socket rather than the Telnet protocol.

## Privacy and safety

KLOG and Game DEBUG files can contain device identifiers, game identifiers, thread names, memory addresses, filesystem paths, network details, or other diagnostic information. Review and redact logs before posting them publicly.

PS5 Log Collector only connects to the host and ports entered by the user. It does not upload logs, include telemetry, or perform a jailbreak. Payload transfer happens only after the user explicitly chooses a local ELF file and selects **Send to console**.


## Architecture

- `src/Ps5LogCollector.Core` — KLOG transport, ShadowMountPlus HTTP/FTP retrieval, same-socket ELF/stdout capture, ps5debug-NG protocol, session lifecycle, event parsing, temporary spool, and full-session export.
- `src/Ps5LogCollector` — WPF/Fluent interface, localized resources, Save dialog, and persisted user preferences.
- `src/Ps5LogCollector.Mac` — Avalonia desktop interface for the macOS ARM KLOG + SM+ build.
- `tests/Ps5LogCollector.Core.Tests` — dependency-free regression suite covering TCP, timestamps, ShadowMount HTTP API parsing, ps5debug-NG framing, event parsing, detach, export, and localization parity.

The live text control retains a bounded recent window for UI performance, while the temporary session spool preserves the complete exportable log.


## Disclaimer

This is an independent diagnostic utility and is not affiliated with, endorsed by, or sponsored by Sony Interactive Entertainment. “PlayStation” and “PS5” are trademarks of their respective owners. Use the application only with systems and software you are authorized to access.
