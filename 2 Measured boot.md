The Lanzaboot project [https://github.com/nix-community/lanzaboote] implemented a secure boot feature in NixOS however its implementation is somewhat unsatisfactory in the sense that, in its current state, there are drawbacks, which hinder it being integrated as an upstream feature, such as:

Lanzaboot is mainly implemented using a specification “modification” of the Unified Kernel Image (UKI). A Unified Kernel Image (UKI) is a single, unified file that includes everything a computer needs to start and run a Linux distribution. It combines a bootloader, the Linux kernel, some initial resources, and more into a single file. The most important of these initial resources is the .initrd file, which we will go into greater detail shortly. [Go into greater detail about this “specification hack”]
https://github.com/uapi-group/specifications/blob/main/specs/unified_kernel_image.md  
The way in which Lanzaboot is applied requires one to first install NixOS via an insecure boot then install Lanzaboot to have secure boot in effect. This is undesirable and ideally we should have boot security by default when installing NixOS for the first time.This video goes into greater details of this process: Secure Boot On NixOS - Lanzaboote

Relevant PRs
boot: load addons from systemd-boot Type 1 entries systemd/systemd#28057
systemd-stub: support loading .initrd via addons systemd/systemd#28070
In order to overcome this specification “modification,” upstream features, like load addons and supporting the loading of .initrd via addons, are integral to transitioning away from the current specification hack. .initrd stands for "initial ramdisk" and it is a scheme for loading a temporary root file system into memory during the boot process of the Linux kernel. The primary purpose of initrd is to provide kernel modules and necessary drivers that are required to mount the actual root file system, which may be on a device such as a hard drive or a network file system.We use the term “addons” to describe having the kernel and .initrd separately aligned. 

https://www.reddit.com/r/systemd/comments/rgbn3z/eli5_whats_the_difference_between_systemdstub_and

With addons, one can separate the command line or create variations of an image. Each addon can be signed independently. The responsibility of handling this separation falls on the systemd stub, which is similar to Lanzaboote is a specialised bootloader designed for a specific kernel and image ID. A systemd-stub, is distinct from systemd-boot, which is a versatile bootloader for any generation. A systemd-stub is a purpose-built bootloader designed to boot only a specific kernel and initrd. As we align with the Lanzaboote style, systemd stub will be extended to support separated initrds and kernels. Additionally, work is being done on establishing a connection between systemd-boot and systemd-stub by creating a UEFI protocol for communication with the addons they recognize.

[Go into greater how these upstream features are implemented]


While secure boot provides a greater degree of security, it is still susceptible to attacks such as rootkits, which can conceal their presence or the presence of other undesirable software on a computer, or Direct Memory Access (DMA) attacks, which can inject code into memory during the boot process. We can gain a greater level of security through an extra layer of security called Measured Boot. 

Unlike Secure Boot, Measured Boot has a hardware dependency – Trusted Platform Module 2.0 (TPM2). The TPM is typically a dedicated microcontroller integrated into a computer's motherboard or as a separate hardware component that provides a secure enclave for storing sensitive information, performing cryptographic operations, and enhancing overall system security.

Measured Boot records a hash of each system component onto the Trusted Platform Module 2.0 (TPM2) as the system boots. This entails measuring each component, using memory locations called Platform Configuration Registers, from the firmware to the boot start drivers, which are then hashed. These measurements are then securely stored in the TPM2, and a log is produced, facilitating remote verification of the client's boot state.

