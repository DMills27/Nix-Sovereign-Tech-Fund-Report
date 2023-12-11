## Bootspec V2

Relevant PRs

- [[RFC 0165] Bootspec v2 rfcs#165](https://github.com/NixOS/rfcs/pull/165)

Bootspec is a data format, in the form of a JSON document, that serves the purpose of concisely expressing metadata about a NixOS generation from a boot perspective. Bootspec’s main objective is to tackle the fragility and maintenance challenges stemming from having an independent extension of features by bootloaders, which often leads to the creation of dissimilar public APIs across nixpkgs.

Bootspec serves as the principal input for bootloader backends such as systemd-boot and GRUB, facilitating the generation of files in `/boot/loader/entries/` and `grub.cfg` directories. This involves utilizing a stable, comprehensive, and machine-parsable definition of a NixOS Generation, functioning as an intermediate representation (IR) bridging the gap between the NixOS system definition and the tools responsible for bootloader management.

The bootspec paradigm creates a strict reliance on data from the bootspec configuration, excluding additional data beyond user preferences, such as timeout settings. This will ultimately foster well-defined public APIs, discourage ad-hoc measures taken when booting up, and prevents the incorporation of extraneous data within the generation boot data, which is concealed from the user.

Bootspec V1, the first iteration of Bootspec, addressed various issues but exhibited gaps, such as the lack of information about device trees, which are data structures used to describe the hardware configuration of a system, particularly in embedded systems and open architecture platforms. Bootspec V2 addresses this by reevaluating the initrd secret feature, which has proven insecure and problematic (see: [GitHub issue #157989](https://github.com/NixOS/nixpkgs/issues/157989)). In Bootspec V2, the goal is to replace initrd secrets with systemd credentials, significantly enhancing security. Users can encrypt these credentials to TPM2, store them in /boot (even on the boot partition), and then have TPM2 unlock them before booting. This approach ensures compatibility with a broader range of systems and promotes uniformity within NixOS.

At its core, bootspec ensures a more focused approach. Bootloader backends will minimize reliance on extraneous files and directories, prioritizing information sourced directly from the bootspec." Which essentially goes from the "user preferences" to the bootspec's generation to becoming "bootable for the current generation."

This refined approach not only elevates readability, comprehension, and predictability but also fosters innovation in bootloader development, all while preventing the accrual of unnecessary technical debt. Furthermore, the shift to systemd credentials ensures compatibility with a broader range of systems, promoting uniformity within NixOS.
