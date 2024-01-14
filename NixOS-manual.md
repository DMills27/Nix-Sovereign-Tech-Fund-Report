-- This would go in the Development section under "Building Specific Parts of NixOS"

# Understanding Enhanced Security Features in NixOS

NixOS differs from more traditional Linux distributions in a number of ways due to its declarative nature. Perhaps, the most visible of these differences is the inclusion of NixOS generations in the bootloader upon starting one's machine. Whereas with other Linux distributions one would see a choice of one or more operating systems to boot from, NixOS allows one to access the various (older) stages of its operating system every time its configuration file is changed and the system is rebuilt.

## Secure boot

However, even before getting to this point one might ask how secure is the process leading up to the stage of the bootloader becoming active and thereafter?

By default, NixOS, like other Linux distros, requires a security feature called secure boot to be disabled, upon initial installation, which leaves it seceptible to attacks such as rootkits and bootkits, that can gain unauthorised access to a system during the boot process. The secure boot process ensures that only signed and trusted code is executed during the system startup, guarding against unauthorized or malicious software.

Initally, the Lanzaboote project provided a tool that facilitates the signing of custom secure boot components with user-defined keys, such as  Microsoft keys. However, it's important to note that signing on behalf of Microsoft is not possible without access to their private key. A common challenge arises during the installation of NixOS on a new laptop with secure boot, as the initial step involves adding or registering personal keys into the system's firmware, which can be intricate. the signing of custom secure boot components with user-defined keys, such as  Microsoft keys. However, it's important to note that signing on behalf of Microsoft is not possible without access to their private key. A common challenge arises during the installation of NixOS on a new laptop with secure boot, as the initial step involves adding or registering personal keys into the system's firmware, which can be intricate.

In order for a NixOS installer to work with secure boot enabled, by default, we need the bootloader (e.g., GRUB or Shim) and the kernel to be signed with cryptographic keys that are recognised by the system's Unified Extensible Firmware Interface (UEFI). More specifically, we need a shim, which is a first-stage boot loader, which, in turn, is a program that is executed by a computer's hardware during the initial boot process such as the UEFI, that embeds a self-signed certificate from a trusted entity know as the Certificate Authority (CA). Microsoft  signs shim binaries, which ensures that they can be booted on all machines with a pre-loaded Microsoft certificate. 

Note that this is an improvement over the Lanzaboote tool which required one to first install NixOS via an insecure boot then install Lanzaboot to have secure boot in effect. This is undesirable and ideally we should have boot security by default when installing NixOS for the first time.



These differences might lead one to question: what are the implications for the security in this context? For instance, 

It is natural to ask how does this affect the kind of security features that would be need to be added on an operating system like NixOS vis-à-vis


