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

- **Magisk** from the [pixincreate/Magisk](https://github.com/pixincreate/Magisk) GrapheneOS fork, baked into `/system_ext/bin/magisk`. `magisk -c` reports `189e867f:MAGISK:R (31000)`, which is [commit 189e867f](https://github.com/pixincreate/Magisk/commit/189e867f). The Magisk app package is not installed. `Magisk-v31.0-3.apk` in the phone's OTA folder is the manager, installed as a normal app update. Cancel any prompt to patch `boot`.
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

## Upstream

- GrapheneOS stable `2026091000`: https://grapheneos.org/releases
- GrapheneOS manifest tag: https://github.com/GrapheneOS/platform_manifest
- Magisk fork: https://github.com/pixincreate/Magisk
- Custota: https://github.com/chenxiaolong/Custota
- OEMUnlockOnBoot: https://github.com/chenxiaolong/OEMUnlockOnBoot
- avbroot: https://github.com/chenxiaolong/avbroot

The image is a derivative of those projects. Their licenses cover the corresponding code. This repository holds the feed, the checksums, the Custota signature, and these notes.
