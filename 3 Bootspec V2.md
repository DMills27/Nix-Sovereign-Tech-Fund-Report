Relevant PRs

- [[RFC 0165] Bootspec v2 rfcs#165](https://github.com/NixOS/rfcs/pull/165)

Bootspec is a set of precomputed or stored information which includes information about the system's hardware, software configuration, and potentially other relevant parameters. These facts play a central role in configuring bootloaders like systemd-boot and GRUB. These bootloaders use the information to create the necessary configuration files for the boot process, ensuring that the system boots correctly based on its current state and configuration.

Work has been done on standardising the functionality of bootloaders in Nix packages. The challenge lies in the absence of a consistent file format to articulate instructions like "install this generation with your custom bootloader logic." Currently, we rely on custom tools and scripts to convey information to the bootloader.

Bootspec aims to provide a guide for a generation to generate bootloader files. Bootspec encompasses details about init, initrd, secrets, kernel, kernel parameters, the generation system's name, and the top level itself. It supports specialisation and more, aiming to eliminate divergent features among different bootloader installers.

While Bootspec V1 addresses numerous aspects, there are notable gaps, such as the absence of information about device trees. Device trees are data structures used to describe the hardware configuration of a system, particularly in embedded systems and open architecture platforms. Unlike x86 systems that utilise Advanced Configuration and Power Interface (ACPI) for multiple hardware support, AArch64, ARM64, and similar embedded ecosystems rely on device trees to outline the physical devices a motherboard should possess.

One crucial aspect we are reevaluating is the initrd secret feature, which has proven to be insecure, prone to implementation issues, and problematic (see: [https://github.com/NixOS/nixpkgs/issues/157989](https://github.com/NixOS/nixpkgs/issues/157989)). In Bootspec V2, our objective is to replace initrd secrets with systemd credentials. Users can encrypt these credentials to TPM2, store them in /boot (even on the boot partition), and then have TPM2 unlock them before booting. This approach significantly enhances the security of initrd secrets.

Furthermore, the shift to systemd credentials ensures compatibility with a broader range of systems, promoting uniformity within NixOS.
