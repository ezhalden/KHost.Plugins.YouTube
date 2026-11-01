<img src=".github/preview.png" alt="YouTube Search: a KHost plugin. Find a karaoke video, hit Enqueue, and it lands in the singer's queue." width="100%">

[![Release](https://img.shields.io/github/v/release/ezhalden/KHost.Plugins.YouTube?style=for-the-badge&color=F0452B&labelColor=141414)](https://github.com/ezhalden/KHost.Plugins.YouTube/releases/latest)
[![KHost plugin API 8](https://img.shields.io/badge/KHost_plugin_API-8-FFD23F?style=for-the-badge&labelColor=141414)](#build-it-yourself)
[![.NET 10](https://img.shields.io/badge/.NET-10-FFD23F?style=for-the-badge&labelColor=141414)](https://dotnet.microsoft.com/)
[![Windows, macOS, Linux](https://img.shields.io/badge/runs_on-Win_%7C_Mac_%7C_Linux-FFD23F?style=for-the-badge&labelColor=141414)](#install)
[![yt-dlp](https://img.shields.io/badge/powered_by-yt--dlp-FFD23F?style=for-the-badge&labelColor=141414)](https://github.com/yt-dlp/yt-dlp)

# YouTube Search for KHost

Search YouTube right from KHost's **Song Search** panel. Pick a video, hit **Enqueue**, and it
downloads into your library and joins the queue for whoever's singing. That's it.

No API key, no Google account, nothing to set up first. Searches go through
[yt-dlp](https://github.com/yt-dlp/yt-dlp), and the plugin finds or fetches it on its own.

## What you get

- **Results with the video's real title.** The artist is a best guess from that title. The
  channel shows as "Published by", since the uploader usually isn't the performer.
- **Enqueue** downloads the video and queues it for the selected singer.
- **Open on YouTube** (under Enqueue) opens the video in your browser.
- **Settings apply as soon as you save them.** No restart needed.

## Install

**From KHost:** open **Plugins → Available**, install **YouTube Search**, restart KHost.

**By hand:** unzip a [release](https://github.com/ezhalden/KHost.Plugins.YouTube/releases/latest)
into its own folder under KHost's `plugins/` directory, enable it on the Plugins page, restart.

One build covers Windows, macOS and Linux.

> [!TIP]
> On a Mac, install yt-dlp yourself (`brew install yt-dlp`). Searches drop from 10+ seconds to
> about one. [Here's why.](#macos-searches-are-slow)

## Settings

| Setting | Default | What it does |
|---|---|---|
| Max Results | `10` | How many results each search shows. |
| yt-dlp path | blank | Use a specific yt-dlp. Blank means "find one, or fetch one". |
| Keep the downloaded yt-dlp up to date | on | Runs `yt-dlp -U` once per launch, only on the copy this plugin downloaded. |
| Ephemeral downloads | off | Deletes each video when KHost closes. The song stays in the library and downloads again next time it's queued, so you'll need internet then. |

## Where yt-dlp comes from

The plugin checks three places, in this order:

1. **Your `yt-dlp path` setting.** If it's set and wrong, you get an error. It won't quietly
   download a second copy behind your back.
2. **`yt-dlp` on your `PATH`.**
3. **Its own copy**, downloaded from yt-dlp's latest GitHub release into `cache/tools/` next to
   KHost, and checked against the published SHA-512 before it ever runs. Covers Windows
   (x64/arm64/x86), macOS (universal) and Linux (x64/arm64, glibc or musl, plus 32-bit ARM).

The plugin only ever updates the copy it downloaded. A yt-dlp you installed is yours to update.

### ffmpeg

yt-dlp uses ffmpeg to merge separate video and audio streams. This plugin doesn't ship or look
for ffmpeg; yt-dlp uses whatever it finds (usually on `PATH`). Without it, yt-dlp falls back to a
single pre-merged MP4 when YouTube offers one.

## Troubleshooting

### Searches suddenly fail

YouTube changes things now and then, and yt-dlp is quick to ship fixes. Update it:

```bash
yt-dlp -U            # a standalone yt-dlp updates itself
brew upgrade yt-dlp  # or use whatever installed it
```

That's also why yt-dlp isn't baked into the plugin: every YouTube change would need a new plugin
release.

### macOS: searches are slow

yt-dlp's macOS download is a big unsigned single-file app, and macOS re-scans it every time it
starts. yt-dlp starts fresh for every search, so you pay for that scan on every search. A package
manager install doesn't have the problem, and the plugin picks it up from `PATH` on its own:

```bash
brew install yt-dlp            # macOS
winget install yt-dlp.yt-dlp   # Windows
sudo apt install yt-dlp        # Debian/Ubuntu (or: pipx install yt-dlp)
```

Then leave **yt-dlp path** blank, or set it to whatever `which yt-dlp` prints.

<details>
<summary>The numbers (Apple silicon, yt-dlp 2026.08.19)</summary>

| | Downloaded copy | `brew install yt-dlp` |
|---|---|---|
| `yt-dlp --version` (no network) | 7.7s warm, 20.8–26.8s cold | **0.12–0.14s** |
| Search, 10 results | 9.0s warm, 16.9–20.4s cold | **1.08–1.16s** |
| Search through the plugin | 22.7s | **1.16s** |

"Cold" is what you get the first time in a while, which tends to be the start of the night.
Windows and Linux weren't measured, but Windows Defender scans executables too, so installing
yt-dlp yourself is the better bet everywhere.

</details>

## Build it yourself

You need the .NET 10 SDK plus the `KHost.Abstractions` and `KHost.Common` 0.61.0 packages. Those
aren't on nuget.org, so build them into a local feed with KHost's `./build/pack-contracts.sh` and
add the feed once:

```bash
dotnet nuget add source ~/.nuget/khost-local -n khost-local
```

Then:

```bash
dotnet build KHost.Plugins.YouTube.slnx
dotnet test tests/KHost.Plugins.YouTube.Tests
```

---

<sub>Not affiliated with or endorsed by YouTube or Google.</sub>
