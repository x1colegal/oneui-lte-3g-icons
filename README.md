# One UI EUX LTE and 3G Icons

A Magisk module that changes Samsung One UI's mobile-network labels on devices using the **EUX** CSC/OMC profile.

## Result

| Network type | Status bar icon |
|---|---|
| UMTS / HSDPA / HSUPA / HSPA / HSPA+ | 3G |
| LTE | LTE |
| LTE with Carrier Aggregation | LTE+ |

This module only changes the icon mapping shown by SystemUI. It does **not** change the modem, radio technology, network speed, carrier aggregation, or connectivity behavior.

## Requirements

- Samsung device running One UI based on Android 16 / API 36 or newer
- Active sales code: `EUX`
- Active OMC path: `/optics/configs/carriers/single/EUX/conf`
- Magisk

The installer checks these requirements and aborts when the active configuration is incompatible.

## Installation

1. Download the latest module ZIP from the Releases page.
2. Open Magisk.
3. Go to **Modules** → **Install from storage**.
4. Select the ZIP.
5. Reboot.

If `/optics` was manually modified before installing this module, restore it to its original state first. This keeps the modification fully systemless and ensures that uninstalling the module restores the stock behavior.

## Verification

After rebooting, run:

```sh
su -c 'dumpsys activity service com.android.systemui/.SystemUIService' |
grep -iE \
'USE_HSPA_DATA_ICON|USE_LTE_INSTEAD_OF_4G|USE_LTE_CA_ICON|ICON_BRANDING_FROM_CARRIER_FEATURE'
```

Expected values:

```text
USE_HSPA_DATA_ICON=false
USE_LTE_INSTEAD_OF_4G=true
USE_LTE_CA_ICON=true
ICON_BRANDING_FROM_CARRIER_FEATURE=KOO
```

The output may appear twice on dual-SIM devices.

The module boot log is stored at:

```text
/data/local/tmp/oneui_eux_lte_3g_icons.log
```

You can also press the module's **Action** button in Magisk to display its status and boot log.

## How it works

Samsung SystemUI uses carrier-specific feature values to decide how mobile-network technologies are represented in the status bar.

This module supplies the following tested CarrierFeature values for EUX:

```text
CarrierFeature_SystemUI_ConfigOpBrandingForIndicatorIcon=KOO
CarrierFeature_SystemUI_ConfigOverrideDataIcon=LTE
```

During `post-fs-data`, the module:

1. Verifies the active sales code and OMC path.
2. Copies the active EUX `conf/system` directory into a temporary runtime directory inside the module.
3. Adds the supplied `customer_carrier_feature.json`.
4. Applies the expected ownership, permissions, and SELinux context.
5. Bind-mounts the runtime directory over the original EUX carrier-feature directory as read-only.

The real `/optics` partition is not modified by the module.

## Why `KOO` is used

On the tested One UI SystemUI implementation:

- The default EUX branding enables separate `H` and `H+` icons.
- Some alternative branding values disable those icons but remap HSPA technologies to `4G`.
- `KOO` disables the separate HSPA icons without triggering that HSPA-to-4G remapping.
- The independent `LTE` override keeps actual LTE displayed as `LTE` and LTE carrier aggregation as `LTE+`.

This behavior was determined by inspecting and testing Samsung's SystemUI carrier-icon logic on the target firmware.

## Uninstallation

Remove the module from Magisk and reboot.

Because the module uses a bind mount and does not write to `/optics`, stock behavior returns automatically after reboot, provided the partition was not manually modified separately.

## Troubleshooting

### Module installs but icons do not change

Check the boot log:

```sh
su -c 'cat /data/local/tmp/oneui_eux_lte_3g_icons.log'
```

Confirm the active configuration:

```sh
getprop ro.csc.sales_code
getprop persist.sys.omc_path
```

Expected output:

```text
EUX
/optics/configs/carriers/single/EUX/conf
```

### Module reports an inactive or unexpected configuration

This module is intentionally restricted to EUX. Do not rename another CSC to EUX or bypass the checks blindly: carrier-feature behavior differs across firmware and regional packages.

### HSPA+ still appears as 4G

Confirm that the active branding is `KOO`, not `OYA`, `OYB`, or `VID`:

```sh
su -c 'dumpsys activity service com.android.systemui/.SystemUIService' |
grep -i 'ICON_BRANDING_FROM_CARRIER_FEATURE'
```

## Module structure

```text
.
├── action.sh
├── customize.sh
├── module.prop
├── payload/
│   └── customer_carrier_feature.json
├── post-fs-data.sh
├── service.sh
└── skip_mount
```

## Compatibility notice

This module was built around the carrier-icon implementation present in the tested Samsung One UI 8 / Android 16 EUX firmware. Samsung may change SystemUI internals, CarrierFeature handling, file locations, or accepted values in future updates.

A firmware update may therefore require a new module version even when the CSC remains EUX.

## Disclaimer

Use this module at your own risk. Root modifications can cause boot problems, UI crashes, failed updates, or unexpected carrier-specific behavior. Keep a working Magisk recovery/removal method available before installing experimental modules.

## Compatibility

This module has been tested on a Samsung Galaxy S24 (Exynos) running One UI 8.5 - UPDATE: Also working on OneUI 9

It may also work on older or newer One UI versions, provided that the device uses the same Samsung CarrierFeature and SystemUI logic. Compatibility with other devices, chipsets, CSCs, or firmware versions is not guaranteed. on older OneUIs, you will have to change the module to disable the Android API restrictions. on newer ones, you CAN try flash, but working is NOT guaranteed. Samsung CAN change the paramters on SystemUI or Carrier Config Binary JSON.

## Author

Created by **x1co**.
