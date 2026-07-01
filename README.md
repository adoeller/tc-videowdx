# MediaWDX

A [Total Commander](https://www.ghisler.com/) content plugin (`.wdx`) that exposes video and audio metadata as sortable, searchable columns — duration, resolution, codec, bitrate, aspect ratio, language codes, and more.

![Settings](Settings.png)
![Dup](Dup.png)

---

## Features

- **26 metadata fields** extracted from video and audio files
- **SQLite cache** — each file is read once; TC columns repaint instantly from cache
- **Two extraction backends**: MediaInfo (recommended) or ffprobe
- **Two cache key strategies**: by file path or by partial hash (detects renames/moves)
- **Duplicate finder** — finds files with an identical content hash (requires hash strategy)
- **GUI maintenance tool** (`VideoWdxTool64.exe`) for scanning, statistics, and settings
- **Localized UI**: English and German; auto-detects OS language
- Ships as both 32-bit (`.wdx`) and 64-bit (`.wdx64`) — TC loads the matching bitness

---

## Installation

1. In Total Commander, open **Configuration → Plugins → Content plugins (.wdx)**.
2. Click **Add** and select `VideoWDX.wdx` (or `.wdx64`) from the plugin folder — or drag the `pluginst.inf` file onto the TC window for automatic installation.
3. Restart Total Commander.
4. Add columns via **Show → Define column sets** or the custom column dialog.

Place `VideoWDX.ini` next to the plugin DLL. If it is absent, the plugin writes a default one on first load.

---

## Fields

Fields are organized into three groups. Enable or disable each one in the Settings tab of the GUI tool or directly in `VideoWDX.ini`.

### General (enabled by default)

| Field | Description | Units |
|-------|-------------|-------|
| Duration | Playback length | hh:mm:ss.fff · hh:mm:ss · s · ms |
| Container | Container format (e.g. Matroska, MPEG-4, AVI) | — |

### General (disabled by default)

| Field | Description |
|-------|-------------|
| OverallBitrate | Combined stream bitrate (Bps / KBps / MBps) |
| Album | Tag: album name |
| Title | Tag: title |
| Performer | Tag: performer |
| Artist | Tag: artist |
| TrackNumber | Tag: track number |

### Video (enabled by default)

| Field | Description | Units |
|-------|-------------|-------|
| VideoBitrate | Video stream bitrate | Bps · KBps · MBps |
| Width | Frame width | px |
| Height | Frame height | px |
| VideoCodec | Codec name (e.g. AVC, HEVC, AV1) | — |
| DisplayAspectRatio | Display aspect ratio (e.g. 16:9, 2.39:1) | — |

### Video (disabled by default)

| Field | Description | Units |
|-------|-------------|-------|
| VideoFPS | Frame rate | fps |
| VideoFrameCount | Total number of frames | — |
| VideoRotation | Rotation metadata | deg |

### Audio (enabled by default)

| Field | Description | Units |
|-------|-------------|-------|
| AudioBitrate | Audio stream bitrate | Bps · KBps · MBps |
| AudioCodec | Codec name (e.g. AAC, AC-3, FLAC, Opus) | — |

### Audio (disabled by default)

| Field | Description | Units |
|-------|-------------|-------|
| AudioStreamCount | Number of audio tracks | — |
| AudioFormat | Low-level audio format | — |
| AudioBrMode | Bitrate mode (CBR / VBR) | — |
| AudioSamplingRate | Sampling frequency | Hz |
| AudioChannels | Number of channels | — |
| AudioFmtEndianness | Byte order (Little / Big) | — |
| AudioFmtSign | Sample sign (Signed / Unsigned) | — |
| AudioLangCode | ISO 639 language code of the first audio track | — |

---

## GUI Tool — VideoWdxTool64.exe

Open `VideoWdxTool64.exe` (in the plugin folder) to manage the cache and adjust settings. The tool connects to the same SQLite database as the plugin.

### Overview tab

Displays cache statistics:

- Total entries, database size, oldest and newest scan timestamp
- Hash duplicate count (only meaningful with hash key strategy)
- Top 15 containers and video codecs
- Top 15 audio languages
- Top 10 longest, shortest, and largest files

Buttons:

| Button | Action |
|--------|--------|
| Refresh | Re-read statistics from the database |
| Cleanup | Delete cache entries whose file no longer exists on disk |
| Compact | Run `VACUUM` to shrink the database file |
| Clear database… | Delete **all** cache entries (prompts for confirmation) |

### Rescan tab

Maintains a list of folders to be scanned in bulk (e.g. a media library).

| Button | Action |
|--------|--------|
| Add folder… | Add a directory to the scan list |
| Remove folder | Remove the selected directory |
| Re-scan existing | Re-extract metadata for every file already in the database |
| Scan directories | Scan all listed folders — add new files, update changed ones |
| Add new from directories | Scan listed folders but skip paths already present in the database |
| Cancel | Abort the running scan |

Progress is shown at the bottom with a file counter.

### Settings tab

All changes are saved to `VideoWDX.ini`. Plugin-affecting settings (fields, backend, strategy) take effect after restarting Total Commander. The language selection takes effect after restarting the tool.

| Setting | Description |
|---------|-------------|
| **Language** | UI language: Auto (OS), English, German |
| **Key strategy** | `path` — cache key is the full file path (fast, no file reading on cache hit). `hash` — cache key is a partial content hash (survives rename/move; required for duplicate detection). |
| **Backend** | `mediainfo` (recommended) or `ffprobe` |
| **Backend path** | Path to `MediaInfo.dll` or `ffprobe.exe`. A bare filename is resolved relative to the plugin folder. |
| **Database path** | Path to `cache.db`. Leave empty to use the default location (`%APPDATA%\TC_VideoWDX\cache.db`). |
| **Parse speed** | `fast` — read container header only (sufficient for all fields, very quick). `accurate` — scan the whole file (more precise bitrates, significantly slower). |
| **Video / Audio extensions** | Comma-separated list of uppercase extensions the plugin should process. Files with other extensions are ignored without opening. |
| **Saved fields** | Per-field enable/disable switches. Disabled fields do not appear in TC's column list. |

### Duplicates tab

> **Only available when the key strategy is set to `hash`.**

Finds files in the database whose partial content hash is identical.

| Button | Action |
|--------|--------|
| Find duplicates | Query the database for hash-matched groups |
| Select newer | In each group, select the file with the more recent modification date |
| Unselect all | Clear all checkboxes |
| Delete selected | Permanently delete the checked files (prompts for confirmation) |

Files are shown grouped; the group header displays the shared file size and duration as additional context.

---

## Command-line usage — VideoWdxTool64.exe

The tool can be driven from the command line for use in scheduled tasks or scripts. When any argument is recognized, the tool runs silently (Rescan tab is shown), performs the requested action, and exits automatically when finished.

```
VideoWdxTool64.exe [/add <folder>] [/rescan | /fastscan]
```

Both `/` and `--` prefixes are accepted (e.g. `--rescan` is equivalent to `/rescan`).

### Parameters

| Parameter | Description |
|-----------|-------------|
| `/add <folder>` | Add `<folder>` to the configured scan folder list and persist it to `VideoWDX.ini`. If the folder is already in the list it is not duplicated. Can be combined with a scan parameter. |
| `/rescan` | Scan all configured folders and add or update every media file found (equivalent to the **Scan directories** button). The tool closes when the scan finishes. |
| `/fastscan` | Scan all configured folders but skip files whose path is already in the database (equivalent to **Add new from directories**). Useful for nightly jobs where only newly arrived files should be indexed. The tool closes when the scan finishes. |

If neither `/rescan` nor `/fastscan` is given the tool closes immediately after processing `/add` (if present).

### Examples

Index only new files in the already-configured folders:
```
VideoWdxTool64.exe /fastscan
```

Add a new folder to the list, then do a full scan of all folders:
```
VideoWdxTool64.exe /add "D:\Movies\New Arrivals" /rescan
```

Just register a folder without scanning (e.g. from a setup script):
```
VideoWdxTool64.exe /add "D:\Music"
```

Force re-extraction of all cached entries (useful after a backend or field change):
```
VideoWdxTool64.exe /rescan
```

### Scheduled task (Windows Task Scheduler)

Create a daily task to pick up new media files:

- **Program:** `C:\Program Files\totalcmd\plugins\wdx\VideoWDX\VideoWdxTool64.exe`
- **Arguments:** `/fastscan`
- **Start in:** *(leave empty or set to the plugin folder)*

The tool exits with no UI interaction once the scan is complete.

---

## Configuration file — VideoWDX.ini

The INI file is read by both the plugin DLL and the GUI tool. It lives next to the plugin DLL. A default file is created automatically if absent.

```ini
[General]
Language           =          ; '' = auto, 'en', 'de'
KeyStrategy        = path     ; path | hash
Backend            = mediainfo ; mediainfo | ffprobe
BackendPath        = MediaInfo.dll
DbPath             =          ; empty = %APPDATA%\TC_VideoWDX\cache.db
ParseSpeed         = fast     ; fast | accurate
VideoExtensions    = ASF,AVC,AVI,D2V,DAT,DIVX,FLV,M2TS,M4V,MKV,MOV,MP4,...
AudioExtensions    = AC3,AIFF,APE,FLAC,M4A,MKA,MP3,WAV,WMA,...

[Fields]
Duration           = 1        ; 1 = enabled, 0 = hidden
VideoBitrate       = 1
AudioBitrate       = 1
Width              = 1
Height             = 1
Container          = 1
VideoCodec         = 1
AudioCodec         = 1
DisplayAspectRatio = 1
OverallBitrate     = 0
; ... (all 26 fields)

[RescanFolders]
; Written automatically by the GUI tool.
1 = D:\Movies
2 = D:\Series
```

---

## Cache key strategies

### Path strategy (default)

The file path is the cache key. A cached entry is reused as long as the path does not change. Fast — no file content is read on a cache hit.

**Use this when** files stay in their folders and you don't need duplicate detection.

### Hash strategy

A 64-bit partial hash (xxHash64 of file size + first 64 KB + last 64 KB) is the cache key. A renamed or moved file is recognized from cache without re-reading all metadata. Both the original and the copy/moved location are stored as separate rows, enabling the Duplicates tab.

**Use this when** you frequently rename or reorganize files, or want to find duplicate videos.

> **Note:** Every newly encountered file must be read partially (128 KB maximum) to compute the hash. On a cold cache this is slightly slower than the path strategy, but the hash is computed only once per file.

---

## Performance notes

- Metadata is read once and cached in SQLite; subsequent TC repaints are instant.
- The `fast` parse speed reads only the container header — this is sufficient for all fields and is the recommended default.
- `accurate` scans the entire file and may be noticeably slower on large files or slow drives.
- The plugin skips files whose extension is not in the VideoExtensions / AudioExtensions lists, so TC browsing unrelated folders has no overhead.

---

## Requirements

- Total Commander 9.0 or later (32-bit or 64-bit)
- Windows 7 or later
- **MediaInfo backend** (recommended): `MediaInfo.dll` (or `MediaInfo_x64.dll`) in the plugin folder or on `PATH`
- **ffprobe backend** (alternative): `ffprobe.exe` from [FFmpeg](https://ffmpeg.org/) on `PATH` or specified via `BackendPath`

---

## File layout

```
VideoWDX\
  VideoWDX.wdx          32-bit plugin
  VideoWDX.wdx64        64-bit plugin
  VideoWDX.ini          configuration (created automatically if absent)
  VideoWdxTool64.exe    GUI maintenance tool (64-bit)
  MediaInfo.dll         MediaInfo backend (place here or on PATH)
  pluginst.inf          TC auto-install hint
  locale\
    VideoWdxTool.de.po    German UI translation
```
