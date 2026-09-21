# Nothing Phone (2) / Pong — Linux 5.10.270 — Baseline

Kernel sources for the Android 17 / LineageOS 24.0 project.
Changelog organized on 2026-09-21. Code revision before documentation changes: `d9b84e9900f957c94156f190859dde4593e4308f`.

## Repository branches

| Branch | Purpose |
| --- | --- |
| [`lineage-24.0`](https://github.com/Szmazwidi/android_kernel_nothing_sm8475/tree/lineage-24.0) | Existing fork baseline, preserved during cleanup. Already includes local Pong fixes; this is not an unmodified LineageOS upstream tree. |
| [`pong-a17-5.10.270-baseline`](https://github.com/Szmazwidi/android_kernel_nothing_sm8475/tree/pong-a17-5.10.270-baseline) | Reference branch: Linux 5.10.270, Android Common integration and earlier PM/UFS fixes. |
| [`pong-a17-5.10.270-experimental`](https://github.com/Szmazwidi/android_kernel_nothing_sm8475/tree/pong-a17-5.10.270-experimental) | Baseline plus accumulated experimental fixes and backports. |

## Baseline changelog — compared with `lineage-24.0`

### Linux and Android Common

- Updated the kernel from 5.10.257 to **5.10.270**, integrating Android Common 5.10.269 followed by Linux stable 5.10.270.
- Included upstream fixes for memory management, filesystems, networking, Bluetooth and error handling. Not all changes apply to Pong hardware.
- Adapted the integration to existing Android and Qualcomm interfaces, including MHI, platform shutdown, TCP and QRTR packet length validation.

### Storage and suspend/resume

- Fixed interactions between SCSI/UFS power management and error recovery, including a deadlock between PM and the SCSI error handler.
- Added system suspend/resume tracking and fixes for power mode transitions and START STOP UNIT timeouts.
- Added UFS multi-clear support and fixed a race between interrupt handling and controller reset.
- Adapted multi-clear to local locking (`hba->host->host_lock`) and atomic request completion.
- Preserved earlier Pong changes, including CNSS2/QCA6490 power sequencing and thermal frequency table initialization.

This baseline is the project's reference point, not an unmodified Linux 5.10.270 tag.

## Validation and limitations

- The earlier SCSI/UFS PM stack was tested on Pong: 98 successful suspend entries in the reported interval, with no observed resume failures or UFS/SCSI errors. This did not test forced timeouts or every recovery path.
- Those results apply to an earlier integration stage, not automatically to every subsequent commit or the entire experimental branch.
- Branch cleanup checked history and documentation changes only; no new build or device test was performed. The baseline name does not imply complete hardware validation.

## Provenance

Linux stable, Android Common, LineageOS and local development history is preserved.
Backports originate from Linux upstream, Qualcomm, arter97 and other projects
identified in the commit messages. Authors, source SHAs and adaptation notes are
retained; branch cleanup does not squash or rewrite that history.

The original Android Common contribution guidelines are preserved in
[`README.android-common.md`](README.android-common.md). General Linux documentation
remains in [`README`](README) and `Documentation/`.
