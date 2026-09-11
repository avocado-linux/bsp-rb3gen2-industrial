# Changelog

All notable changes to avocado-bsp-rb3gen2-industrial are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0]

### Added
- Prepared board support for the RB3 Gen 2 Industrial Mezzanine: WCD9370 audio
  over SoundWire with the LPASS macros, and the ST33HTPM SPI TPM with its GENI
  SPI controller.
- The mezzanine's device tree as
  `overlays/qcs6490-rb3gen2-industrial-mezzanine.dtso`.

### Notes
- **Not verified on hardware.** Every package is checked to exist in the feed;
  none is checked to bind to a real device. See README.
- The codec's driver is `wcd937x`, which covers the WCD9370; the SoundWire half
  is a separate module without which the codec does not probe.
- The overlay is shipped but not applied, as with the vision mezzanine.
