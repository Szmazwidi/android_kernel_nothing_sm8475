# Nothing Phone (2) / Pong — Linux 5.10.270 — Experimental

Kernel sources for the Android 17 / LineageOS 24.0 project.
Changelog organized on 2026-09-21. Code revision before documentation changes: `e457b869750d1dd52995c61b07d7d1418e448b83`.

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

## Experimental changelog — additions to baseline

### Memory, zRAM and compression

- Updated **LZ4 to 1.10.0**, with kernel API adaptations and exports for the modular zRAM backend.
- Backported newer **zsmalloc and zRAM** implementations: locking and object mapping changes, dedicated compression backends, algorithm parameters, dictionaries and recompression priorities.
- Enabled the LZ4 backend and selected LZ4 as the default zRAM algorithm in the Pong configuration. Android startup settings may override this; check the active algorithm on the device.
- Added experimental **Kcompressd** to offload eligible swap work from kswapd to a worker, with a bounded queue, page reference management, worker shutdown and fallback handling. Enabled in the Pong configuration.
- Adapted Kcompressd FIFO storage to Linux 5.10, preserving the subsequent local fix.
- Reworked synchronous swap I/O: read/write helpers, on-stack bio for synchronous reads, removal of the old `rw_page` API, mpage and zswap adaptations, and complete bio resource cleanup.
- Marked synchronous zRAM I/O for detection by swapon and Kcompressd eligibility checks.
- Improved bio vector iteration and copying helpers.

### GPU and synchronization

- KGSL: safer GMU array indexing, ioctl data size validation and atomic memory descriptor state.
- Fixed GPUREADONLY reporting in debugfs and type confusion in the LPAC path.
- Used the kernel sorting implementation when merging dma-fences.

### FastRPC / DSP

- Validated page ranges passed to the DSP and fixed a use-after-free in asynchronous performance counters.
- Separated overlap handling for ION and non-ION buffers; the local descriptor classification adaptation also handles fd 0.

### Networking

- Added **BBRv3** with its TCP sampling, ECN, retransmission and TSO dependencies, plus PLB support.
- BBR is available as `bbr`; **CUBIC remains the default**, and PLB is disabled by default.
- Kept the new callback optional for BPF TCP struct_ops programs.
- CNSS2: added a failsafe reboot timer to the recovery path. This preserves the earlier Wi-Fi power sequencing fix.

### Power management, USB and input

- Added shared-state locking in Qualcomm LPM/cluster LPM governors and further validation of cluster idle state selection.
- Handled spurious RPMh completion interrupts and corrected PMIC warm reset counting.
- USB: fixed a DWC3 wakeup deadlock, checked pointers during suspend, corrected xHCI TD invalidation and fixed MIDI bind retries.
- Goodix: added double-tap gesture support with a local adaptation preserving existing wake gestures; reduced verbose touch and haptics logging.
- GENI I2C: added memory barriers after register writes and corrected handling of spurious interrupts.

### Other fixes

- Prevented division by zero in UFS statistics.
- Converted the UID statistics lock to rt_mutex and added rwlock contention detection.
- Added a hash table for global clock lookups and required helper API adaptations.

## Dependencies outside this repository

The complete Android source set also includes the separate repository
`Szmazwidi/android_kernel_nothing_sm8475-modules`.
The matching local integration commit is `0a41fca1b13a0baaf42ccaccb08187f5e5c0cf5c`:

- DSI ISR race fix;
- Wi-Fi qcacmn: scatterlist and HTC/HIF bounds checks;
- chainmask table protection and allocation error propagation.

These changes **are not contained in this kernel repository**. Cleaning up its
branches does not publish the modules repository automatically. The SHA identifies
the local integration state and does not guarantee that it is available on GitHub.

## Validation and limitations

- The latest BBRv3, Kcompressd and swap/Wi-Fi/fences packages have undergone static review; a full build and runtime testing of this final combination remain pending.
- No compilation or flashing was performed during branch cleanup.
- Earlier PM and package tests do not validate the current HEAD.
- Future build validation should cover boot, Wi-Fi, memory pressure, swap/writeback, suspend/resume and TCP with both CUBIC and BBR.
- TCP structure and memory/I/O API changes require compatible modules. Compatibility with older binary modules is not assumed.
- No measurements yet establish improved FPS, battery life or a performance benefit from Kcompressd.

## Provenance

Linux stable, Android Common, LineageOS and local development history is preserved.
Backports originate from Linux upstream, Qualcomm, arter97 and other projects
identified in the commit messages. Authors, source SHAs and adaptation notes are
retained; branch cleanup does not squash or rewrite that history.

The original Android Common contribution guidelines are preserved in
[`README.android-common.md`](README.android-common.md). General Linux documentation
remains in [`README`](README) and `Documentation/`.
