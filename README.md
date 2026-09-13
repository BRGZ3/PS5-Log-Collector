# PS5 Log Collector

[English](README.md) · [Русский](README.ru.md)

A portable Windows application for viewing, timestamping, and saving kernel logs from a jailbroken PlayStation 5 — without Netcat or command-line tools.

> **Project status:** KLOG collection is available in `v0.1.1`. DEBUG support is planned for a future release.

## Highlights

- Connects directly to `klog.elf` / `klogsrv` over raw TCP.
- Uses port `3232` by default, with an editable host and port.
- Displays incoming KLOG data in a responsive live viewer.
- Adds an ISO 8601 timestamp with milliseconds and the PC time-zone offset to every line.
- Saves the complete session from the moment the connection was established.
- Preserves partial lines when capture is stopped or the remote side disconnects.
- Switches between English and Russian without restarting.
- Remembers the selected language and last endpoint.
- Ships as a self-contained ZIP: extract and run, with no separate .NET installation.
- Does not bundle or invoke Netcat or Telnet.

Example output:

```text
[2026-09-13T22:42:15.123+03:00] kernel: example message
```

## Requirements

- A jailbroken PlayStation 5 capable of running `klog.elf` or a compatible `klogsrv` payload.
- The PC and console connected to the same local network.
- Windows 11 x64 is the primary supported target.
- Windows 10 22H2 x64 is best-effort and should be tested with the current release.

## Quick start

1. Download the latest Windows ZIP from the [Releases page](../../releases/latest).
2. Extract the archive to any folder.
3. Start `PS5LogCollector.exe`.
4. Select **KLOG**.
5. Activate `klog.elf` on the PS5.
6. Enter the console IP address. Keep port `3232` unless the payload uses a custom port.
7. Select **Connect**.
8. Use **Stop** to end capture and **Save log** to export the complete session as UTF-8 text.

Saving is also available while capture is active. Cancelling the Save dialog does not interrupt the connection or discard the session.

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

KLOG files can contain device identifiers, filesystem paths, network details, or other diagnostic information. Review and redact logs before posting them publicly.

PS5 Log Collector only connects to the host and port entered by the user. It does not upload logs, include telemetry, modify the console, or deliver jailbreak payloads.

## Build from source

Prerequisites:

- .NET 10 SDK
- Windows Desktop targeting pack

```powershell
dotnet build Ps5LogCollector.slnx -c Release
dotnet run --project tests/Ps5LogCollector.Core.Tests -c Release
```

Create the self-contained Windows x64 ZIP:

```powershell
./scripts/publish-windows.ps1
```

## Architecture

- `src/Ps5LogCollector.Core` — endpoint validation, raw TCP transport, session lifecycle, timestamp framing, UTF-8 decoding, temporary spool, and full-session export.
- `src/Ps5LogCollector` — WPF/Fluent interface, localized resources, Save dialog, and persisted user preferences.
- `tests/Ps5LogCollector.Core.Tests` — dependency-free regression suite covering TCP, timestamps, cancellation, export, and localization parity.

The live text control retains a bounded recent window for UI performance, while the temporary session spool preserves the complete exportable log.


## Disclaimer

This is an independent diagnostic utility and is not affiliated with, endorsed by, or sponsored by Sony Interactive Entertainment. “PlayStation” and “PS5” are trademarks of their respective owners. Use the application only with systems and software you are authorized to access.
