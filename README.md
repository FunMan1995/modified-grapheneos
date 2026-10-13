# Modified GrapheneOS

Unofficial derivative of [GrapheneOS](https://grapheneos.org/) for the Pixel 9 Pro XL (`komodo`). This repository records the image that is installed on that phone. It is an independent build. It is not an official GrapheneOS release, and it is not affiliated with the GrapheneOS project.

The full OTA is about 1.6 GiB. GitHub rejects files that large, so the zip stays on the phone in `Download/GrapheneOS-OTA/`. Checksums for that folder are in `SHA256SUMS`.

## Installed build

| | |
| --- | --- |
| Device | Pixel 9 Pro XL (`komodo`) |
| Fingerprint | `google/komodo/komodo:17/CP2A.260805.005/2026091000:user/release-keys` |
| Display id | `2026091000` |
| Android | 17 (SDK 37) |
| Security patch | 2026-09-01 |
| Base | GrapheneOS stable `2026091000` |
| Bootloader | Locked (`ro.boot.flash.locked=1`, vbmeta device state `locked`) |
| Verified boot | `yellow` (custom AVB key) |
| OTA type | Virtual A/B |

Archive name: `komodo-ota_update-2026091000-magisk-custota-noclient.zip`

The zip is an avbroot-signed payload. `ota/metadata.txt` and `ota/payload_properties.txt` are the small metadata members. `ota/*.csig` is the Custota signature over that zip. The private signing key is not in this repository.

## Changes from stock GrapheneOS stable

These are present in the system image on this phone:

- **Magisk** baked into `/system_ext/bin/magisk`. `magisk -c` reports `189e867f:MAGISK:R (31000)`, which is [pixincreate/Magisk@189e867f](https://github.com/pixincreate/Magisk/commit/189e867f). The matching fork is [FunMan1995/Magisk](https://github.com/FunMan1995/Magisk). That fork has no GitHub release, and its `master` (`a0f4091`, 2026-08-27) is an older snapshot of the same GrapheneOS changes. The phone uses `Magisk-v31.0-3.apk` from the parent line. The Magisk app package is not installed in the image. Install that APK as a normal app update and cancel any prompt to patch `boot`.
- **Custota 6.4** (`com.chiller3.custota`, versionCode 394240) as a privileged system app at `/system/priv-app/com.chiller3.custota`. It replaces the stock updater certificate set, so the stock updater cannot install official GrapheneOS OTAs. Feed file: `custota/komodo.json`.
- **OEMUnlockOnBoot 1.4** (`com.chiller3.oemunlockonboot`, revision `997dc0f620abd0da2021c2fa64139ef0250de53d`). Init starts it from `/system/etc/init/oemunlockonboot.rc` with classpath `/system/bin/oemunlockonboot.apk`. That keeps OEM unlocking enabled, which this setup needs with a locked bootloader and root.

The phone also has Magisk's built-in systemless hosts module enabled under `/data/adb/modules/hosts`. That module lives in data, not in the OTA zip.

## Custota

`custota/komodo.json` points at the zip and `.csig` beside it. In Custota, set the OTA source to the local folder `Download/GrapheneOS-OTA`, or serve that folder as the update source.

```json
{
  "version": 2,
  "full": {
    "location_ota": "komodo-ota_update-2026091000-magisk-custota-noclient.zip",
    "location_csig": "komodo-ota_update-2026091000-magisk-custota-noclient.zip.csig"
  }
}
```

## Install

This phone is already locked and running the image. Do not use Magisk Direct Install. Do not unlock the bootloader unless a wipe is acceptable.

On a device that already trusts this AVB key:

1. Install `Magisk-v31.0-3.apk` as a normal app update. Cancel any prompt to patch `boot`.
2. In Custota, set the OTA source to `Download/GrapheneOS-OTA`, or sideload the zip from recovery.
3. Leave OEM unlocking enabled.

The first boot after the OTA can take several minutes. Wait for the virtual A/B snapshot merge before another OTA.

First-time custom-key setup, only when `avb_custom_key` is not already this key. Unlocking and locking wipes the device.

```text
fastboot flashing unlock
avbroot ota extract --input komodo-ota_update-2026091000-magisk-custota-noclient.zip --directory extracted --fastboot
export ANDROID_PRODUCT_OUT=extracted
fastboot flashall --skip-reboot
fastboot reboot-bootloader
fastboot erase avb_custom_key
fastboot flash avb_custom_key avb_pkmd.bin
fastboot reboot recovery
# adb sideload the patched zip
fastboot flashing lock
```

`avb_pkmd.bin` is the public AVB key. Keep the private key offline.

## Forks

Every public fork on [FunMan1995](https://github.com/FunMan1995) was checked against this image.

| Fork | Parent | In this image |
| --- | --- | --- |
| [Magisk](https://github.com/FunMan1995/Magisk) | [pixincreate/Magisk](https://github.com/pixincreate/Magisk) | Yes. The system binary is parent commit [`189e867f`](https://github.com/pixincreate/Magisk/commit/189e867f) (`189e867f:MAGISK:R (31000)`, 2026-09-15). Fork `master` is [`a0f4091`](https://github.com/FunMan1995/Magisk/commit/a0f4091898e0) (2026-08-27) and has no release. Those six fork commits are the GrapheneOS changes: point URLs at pixincreate, CI, Zygisk on GrapheneOS, secure app spawning, and hiding direct install on a locked bootloader. Parent `189e867f` is 67 commits ahead of that snapshot, with the same changes already rebased. The OTA folder's manager APK is `Magisk-v31.0-3.apk` from that parent line. |
| [Mango](https://github.com/FunMan1995/Mango) | [Hexadecinull/Mango](https://github.com/Hexadecinull/Mango) | No. AArch32 bridge, separate from this OTA. |
| [armada](https://github.com/FunMan1995/armada) | [armada-os/armada](https://github.com/armada-os/armada) | No. ARM handheld distro. |
| [dragnwash-modframework](https://github.com/FunMan1995/dragnwash-modframework) | [TomXV/dragnwash-modframework](https://github.com/TomXV/dragnwash-modframework) | No. Drag'n Wash mod API. |
| [MeMedia](https://github.com/FunMan1995/MeMedia) | [aniyomiorg/aniyomi](https://github.com/aniyomiorg/aniyomi) | No. Manga and anime app. |
| [fansly-downloader-ng](https://github.com/FunMan1995/fansly-downloader-ng) | [prof79/fansly-downloader-ng](https://github.com/prof79/fansly-downloader-ng) | No. Downloader. |

Custota, OEMUnlockOnBoot, and avbroot are upstream [chenxiaolong](https://github.com/chenxiaolong) projects. There is no fork of those under FunMan1995.

## Upstream

- GrapheneOS stable `2026091000`: https://grapheneos.org/releases
- GrapheneOS manifest tag: https://github.com/GrapheneOS/platform_manifest
- Magisk fork used by this build: https://github.com/FunMan1995/Magisk
- Magisk parent commit in the image: https://github.com/pixincreate/Magisk/commit/189e867f
- Custota: https://github.com/chenxiaolong/Custota
- OEMUnlockOnBoot: https://github.com/chenxiaolong/OEMUnlockOnBoot
- avbroot: https://github.com/chenxiaolong/avbroot

The image is a derivative of those projects. Their licenses cover the corresponding code. This repository holds the feed, the checksums, the Custota signature, and these notes.
