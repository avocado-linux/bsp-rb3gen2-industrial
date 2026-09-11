---
language: yaml
targets:
  - rb3gen2
topics:
  - bsp
  - audio
  - tpm
  - device-tree
---

# RB3 Gen 2 Industrial Mezzanine

Support for the industrial mezzanine of the Qualcomm Robotics RB3 Gen 2.

**This is prep, not a verified BSP.** Nobody here has had this mezzanine on a
board. Everything below is derived from what upstream's
`qcs6490-rb3gen2-industrial-mezzanine.dtso` describes, and every package is
checked to exist in the rb3gen2 feed — but none is checked to bind to a real
device. Treat it as a starting point that will need a pass on hardware.

## What the mezzanine brings

Read off the overlay's targets:

| Overlay target | What it is |
|---|---|
| `qcom,wcd9370-codec` | audio codec, on SoundWire |
| `&lpass_rx_macro`, `&lpass_tx_macro` | LPASS audio macros |
| `&spi11` + `st,st33htpm-spi` | **discrete TPM** (ST33HTPM) |
| `&remoteproc_wpss` | WPSS remoteproc |
| `&pcie0`, `&pcie0_phy`, `&pcie0_port` | PCIe, another `pci1179,0623` bridge |

## The TPM is the interesting part

`&spi11` carries an `st,st33htpm-spi` / `tcg,tpm_tis-spi`. A discrete TPM on the
mezzanine means measured boot and a TPM-sealed encrypted `/var` are possible on
*this* variant and not on a core kit alone — the same capability the SLB9670
provides on the EXMP-Q911, where `AVOCADO_SECURITY_CAPABILITIES` gains `tpm2`
only when `MACHINE_FEATURES` carries `carrier-tpm-slb9670`.

That is a machine-level decision, not something an extension can assert, so
wiring it up means a `MACHINE_FEATURES` gate for this mezzanine plus the
initramfs firmware the SPI bus needs before the TPM is reachable — on the Q911
that was `qupv3fw.elf` via `INITRAMFS_IMAGE_EXTRA_INSTALL`, because `/var` is
unsealed in the initramfs, long before any extension is merged. Expect the same
here. Nothing in this extension attempts it.

## Naming, checked not guessed

The overlay says `qcom,wcd9370-codec`, but there is no `wcd9370` module: the
`wcd937x` driver covers the WCD9370 and WCD9375. Same shape as the vision
mezzanine's IMX577 being driven by `imx412`. The SoundWire half
(`wcd937x-sdw`) is a separate module and the codec does not probe without it.

## The overlay is shipped but not applied

Identical situation to the vision mezzanine — the Qualcomm flow flashes one dtb
and has no overlay-application step, and declaring `device_tree_overlays:` pulls
in `avocado-dtc-overlay-deliver`, which this feed does not carry for this
target. See `bsp-rb3gen2-vision/README.md` for the full reasoning and the
`fdtoverlay` evidence. Until that is closed, none of the hardware above is
reachable regardless of which packages are installed.

## Deliberately absent

`&remoteproc_wpss` and the PCIe block are named by the overlay but get no
packages here. WPSS is the wifi remoteproc and the core kit already carries the
WCN6750 firmware; the PCIe bridge is another `pci1179,0623`, i.e. the same
Toshiba part family as the core kit's QPS615, whose driver already ships in
`avocado-bsp-rb3gen2`. Adding either speculatively would repeat the mistake
that put six packages in the first rb3gen2 BSP that did not exist.
