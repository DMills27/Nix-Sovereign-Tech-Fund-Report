Relevant PRs

- [https://github.com/RaitoBezarius/nixos-shim](https://github.com/RaitoBezarius/nixos-shim)
- [https://hydra.newtype.fr/project/nixos-shim](https://hydra.newtype.fr/project/nixos-shim)

Lanzaboote facilitates the signing of custom secure boot components with user-defined keys, such as  Microsoft keys. However, it's important to note that signing on behalf of Microsoft is not possible without access to their private key. A common challenge arises during the installation of NixOS on a new laptop with secure boot, as the initial step involves adding or registering personal keys into the system's firmware, which can be intricate.

Now, a distinction must be made between the two types of boot entries, namely Type 1 and Type 2 entries:

1. **Type 1 Entries:**
   - Type 1 entries are the traditional boot entries that point directly to a kernel image.
   - They are commonly used for booting the Linux kernel and initiating the boot process.
   - Type 1 entries typically include information about the kernel image, initrd (initial ramdisk), kernel parameters, and other necessary details for booting the operating system.

2. **Type 2 Entries:**
   - Type 2 entries are more flexible and allow systemd-boot to chain-load another bootloader.
   - They can be used for booting other bootloaders or operating systems.
   - Type 2 entries point to a specific EFI binary or bootloader, and they are not limited to booting Linux.

These distinctions help systemd-boot manage different types of boot scenarios and provide flexibility in handling various boot configurations on a UEFI-enabled system.

When configuring systemd-boot, administrators can define these Type 1 and Type 2 entries in the bootloader's configuration files, typically located in the `/boot/loader/entries/` directory. Each entry corresponds to a specific boot option, and systemd-boot uses these entries to present a menu to the user during the boot process, allowing them to choose the desired operating system or configuration.

Now, to more effeciently streamline the process having secure boot present from the outset of any NixOS installer, we are actively involved in the NixOS shim project. This initiative entails constructing a NixOS shim, that is, developing a systemd boot second-stage bootloader, and creating a final ISO/image for submission to the shim-review project. Shim-review serves as a community-driven solution endorsed by Microsoft, allowing open-source contributors to request signature validation without incurring the standard exorbitant fee. Users, from a trusted group of maintainers, can open an issue, specifying their signing intentions, and Microsoft signs the shim exclusively. This approach simplifies the Secure Boot process for NixOS installation, requiring management of machine-owned or custom keys rather than the otherwise intricate process of acquiring secure boot keys.
