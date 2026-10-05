FDS Catalog, storage v4 (64 KiB blocks, 1 solid LZMA2 groups of up to 128 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- First release: FDS and QD disk images, FDS BIOS variants, FDS/QD DATs, DB Export and Dump Log, RetroAchievements console 81, Chinese names, and the FDS images found in the RetroAchievements NES folder.
- Source collections (`source_collections`, `v_collection_files`) and RetroAchievements links per file (`v_ra_collection`).
- Source: 747 ZIPs (nointro 720, retroachievements 27), 33.8 MiB (747 ROM files, 82.7 MiB uncompressed). Populated database: 21.1 MiB (62.5% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 703 ROM records, 307 games, 408 releases; DAT versions: 20260517-061737, 20260617-195332, 20260930-033941.
- RetroAchievements: 34 of 38 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 30.4 MiB/s (405 files); single file with a cold cache 0.364 s (ROM) / 0.334 s (TorrentZip) on average.
- Full audit of the populated database: 712 objects, 1 groups, 728 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-FDS/blob/main/README.zh-CN.md)
