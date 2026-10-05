# RetroBoxDB FDS

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for Nintendo Family Computer Disk System. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 747 source ZIPs, 33.8 MiB (No-Intro 720, RetroAchievements sets 27); 747 ROM files, 82.7 MiB uncompressed |
| Stored size | populated database 21.1 MiB; public Catalog 9.5 MiB (no ROM data) |
| Ratio | 62.4% of the source ZIPs, 25.5% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 64 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 128 MiB (128 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. Whole-set export (747 ROM files in storage order, each group decoded once): 20.0 MiB/s, 6 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 0.364 s, TorrentZip 0.334 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.FDS.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-FDS/releases/latest/download/RetroBoxDB.FDS.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for all seven platforms |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-fds-games.csv) / [summary](reports/ra-fds.json), [build report](reports/fds-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

Change against 32 MiB groups on real data (whole collection with 64 KiB blocks, 77 MiB (one group from 128 MiB)): 64 MiB −1.15%, 128 MiB −4.75%, 256 MiB −4.75%; 128 MiB chosen by the rule.

- Disk images in two No-Intro formats of the same disks: FDS (65,500-byte sides holding the blocks without CRCs or gaps) and QD (65,536-byte sides with a CRC after each block), plus the FDS BIOS variants. Both formats and the BIOS files of both folders are imported; identical files are stored once.
- The whole platform is about 77 MiB after block deduplication, so one 128 MiB group holds it; with one group every block size compresses to about 9.9 MiB and 64 KiB blocks (one per side) need the least metadata. Blocks restart at an optional 16-byte fwNES header and at every side.
- Disk information block of every side: manufacturer code, three-letter game code, game type, revision, side and disk number, disk type, BCD manufacturing date (read as Showa years 50–64 or Heisei years below 50; the raw BCD is kept), country code, and the file amount from block 2.
- Two DAT formats: the FDS DAT creates games and releases; QD entries join the FDS release of the same name. The DB Export lists both formats; the Dump Log covers FDS images.
- RetroAchievements lists FDS games under its own console (ID 81). The RA NES folder contains FDS disk images; they are imported here, not into the NES database. RA hashes drop a 16-byte fwNES header when present.

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 703 / 307 / 408 |
| DAT coverage per version | 20260517-061737: 405/407; 20260617-195332: 295/296; 20260930-033941: 294/295 |
| Local ROMs in no DAT | 9 |
| ROM files of the RetroAchievements set | in a No-Intro DAT 18, RA only 8, hash not in the latest RA snapshot 1 ([list](reports/ra-fds-collection-unknown.csv)); RA games still without a local ROM: [gap list](reports/ra-fds-missing.csv) |
| No-Intro DB Export + Dump Log 20260930-033941 | 408 archives, 1,437 file identities, 8 documented hardware assertions; Dump Log Verified 7 |
| RetroAchievements (console 81) | 38 games with achievements: 34 with a local ROM (47 ROMs), 0 DAT only, 1 DB file only, 3 without a No-Intro counterpart |
| Chinese names | 404 of 404 rows translated (296 unique); 692 local ROMs have a Chinese name |
| Populated-database audit | 712 objects, 1 groups, 728 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.FDS.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.FDS.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.FDS.sqlite --discover --ra --catalog RetroBoxDB.FDS.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
