-- This would go in the Development section under "Building Specific Parts of NixOS" in the NixOS  manual.

# Understanding Enhanced Security Features in NixOS

-- Write a better intro that details the various security features being discussed and how they interrelate to one another.

Increasing security within NixOS and Nixpkgs is vital for wider adoption. The Nix ecosystem faces unique challenges due its declarative nature which sets it apart from other Linux distributions, and requires more nuanced and novel approaches to deal with issues related to security, in particular. Perhaps, the most visible of these differences is the inclusion of NixOS generations in the bootloader upon starting one's machine, which allows one to choose from various (older) stages of the operating system every time the configuration file is changed and the system is rebuilt. This might lead one to ask questions such as:
- Does the inclusion of generations increase the number of potential vulnerabilties that a potential attacker can exploit? 
- What mechanisms are in place to ensure potential vulnerabilties from an existing generation don't propagate to newer ones as they are generated? 

However, even before getting to this point one might inquire:
how secure is the process leading up to the bootloader becoming active and thereafter? Moreover, what processes are in place to ensure the accuracy, consistency and reliabilty of data being sent between Stage 1 and Stage 2 of the boot process, that is, what are the so-called Integrity Checks in place? Here Stage 1 and Stage 2 refer to the processes for booting the Linux kernel, which comprise of establishing details such as the kernel image, initrd (initial ramdisk), and kernel parameters for initiating the operating system (Stage 1), as well as, booting other bootloaders or operating systems, by pointing to a specific Extensible Firmware Interface (EFI) binary or bootloader, and are not necessarily limited to booting Linux operating systems (Stage 2). Furthermore, the dynamic nature of the interpreted languages that NixOS sometimes uses (Python, Perl and Bash) in the background allows for runtime flexibility but also introduces potential security risks, as the interpreter must handle and execute code in real-time.  We detail the mechanisms in place to deal with the aforementioned issues and more in the following sub-sections.  


## Secure Boot

Orginally, NixOS, like other Linux distributions, required a security feature called Secure Boot to be disabled, upon initial installation, which left it susceptible to attacks such as bootkits, that can gain unauthorised access to a system during the boot process. The Secure Boot process ensures that only signed and trusted code is executed during the system startup, guarding against unauthorized or malicious software.

Initally, the Lanzaboote project was developed to ameliorate this issue by providing a tool that facilitates the signing of custom Secure Boot components with user-defined keys, such as Microsoft keys. However, its implementation was unsatisfactory in the sense it required one to first install NixOS via an insecure boot then install Lanzaboote to have secure boot in effect when NixOS is subsequently booted. This was undesirable as ideally we should have boot security by default when installing NixOS for the first time.

This led to the development of a feature that would enable UEFI secure boot by default for NixOS installer images. This consists of two aspects:
 1. Constructing a shim for NixOS, which is a first-stage boot loader, which, in turn, is a program that is executed by a computer's hardware during the initial boot process such as the UEFI, that embeds a self-signed certificate from a trusted entity known as the Certificate Authority (CA). Microsoft signs shim binaries, which ensures that they can be booted on all machines with a pre-loaded Microsoft certificate.
2. Then creating an ISO from the NixOS-shim that is then submitted for review by the shim-review committee which is a community-driven solution endorsed by Microsoft, allowing open-source contributors to request signature validation. 

## Measured Boot
While Secure Boot provides a greater degree of security, it is still susceptible to attacks such as rootkits, which can conceal their presence or the presence of other undesirable software on a computer, or Direct Memory Access (DMA) attacks, which can inject code into memory during the boot process. We can gain a greater level of security through an extra layer of security called Measured Boot. 

Measured Boot records a hash of each system component onto the Trusted Platform Module 2.0 (TPM2) as the system boots. This entails measuring each component, using memory locations called Platform Configuration Registers, from the firmware to the boot start drivers, which are then hashed. These measurements are then securely stored in the TPM2, and a log is produced, facilitating remote verification of the client's boot state.

Unlike Secure Boot, Measured Boot has a hardware dependency – Trusted Platform Module 2.0 (TPM2). The TPM is typically a dedicated microcontroller integrated into a computer's motherboard or as a separate hardware component that provides a secure means for storing sensitive information, performing cryptographic operations, and enhancing overall system security.

## Bootspec V2

Bootspec refers to a JSON document that contains metadata related to the boot process of a NixOS generation, which functions as an intermediate bridge (IR) between the NixOS system definition and the tools tasked with managing the bootloader.
Some of these tools include:

* Multiple Profiles: which allows users to maintain different configurations or sets of packages. Each profile represents a distinct environment with its own package set, dependencies, and configurations. This enables users to switch between different system states or use cases easily.

* Generation Limits: In the context of NixOS, a generation limit refers to the maximum number of past generations (system configurations) that are retained.

* Initrd Secrets: Initrd (initial ramdisk) secrets are cryptographic keys required during the initial stages of the system boot process. These secrets are used to unlock encrypted partitions or perform other security-related tasks before the main operating system is fully loaded.

* Specialisation: Specialisation in NixOS involves tailoring the system configuration to specific use cases or requirements, such as development, server hosting, or desktop usage.

* Custom Labels: Custom labels refer to user-defined labels or tags associated with NixOS configurations. These labels can be used to identify and organize different system states, profiles, or generations.


The main motivation behind Bootspec, in general, is to support a uniform collection of features and design decisions, such as those previously listed, which aren't often supported partially due to the complexity of implementing such features in a particular bootloader, such as those on lower end devices, such as a RaspberryPI or Extlinux. When supported, it is often through globbing directories, that is, by specifying sets of filenames with wildcard characters then utilizing these wildcards to match and locate files according to a predefined pattern. This approach poses several challenges; the main of which is the inherent complexity and significant effort when  introducing new bootloader features or enhancing older ones. Bootspec, thus, addresses the challenges of feature extension in bootloaders, reducing the fragility and maintenance issues associated with disparate public APIs in nixpkgs. 

Bootspec V1, the first iteration of Bootspec, dealt with a number of issues but exhibited gaps, such as the lack of information about device trees, which are data structures used to describe the hardware configuration of a system, particularly in embedded systems and open architecture platforms. Bootspec V2 addresses this by reevaluating the initrd secret feature, which has proven insecure and problematic since it starts to omit a cryptographic key file when upgrading the version of NixOS. In Bootspec V2, the goal is to replace initrd secrets with systemd-creds, which securely stores and retrieves credentials used by systemd units, to significantly enhance security in the NixOS boot process. Users can encrypt these credentials to TPM2, store them in /boot (even on the boot partition), and then have TPM2 unlock them before booting. This approach ensures compatibility with a broader range of systems (such as remote desktops, server farms, etc).

## Portable executable binaries

Portable Executable (PE) binaries refer to executable files that adhere to the PE file format. The PE format is a file structure used for executables, object code, DLLs (Dynamic Link Libraries), and other types of files typically used in Windows operating systems. PE files have a structured layout that includes different sections, each serving a specific purpose. The two primary types of sections are the "code section" and the "data section":

- **Code Section**: The code section contains the actual executable instructions or machine code that the computer's processor executes. The code section is marked as executable and read-only to ensure that the program's instructions are not modified during runtime.

- **Data Section**: The data section, on the other hand, stores various types of data that the program uses during its execution. This data can include global variables, constants, static data structures, and other information that needs to be stored in the executable file. Unlike the code section, the data section may have read and write permissions, allowing the program to modify its own data during runtime.

Of particular interest is the data section which includes critical information such as digital signatures, checksums, configuration settings, or other data necessary for the proper functioning and security of the executable. Read and write support for the data section is crucial for verifying and updating this information as needed. By having separate sections for code and data, PE files follow a modular structure that allows for efficient organization and management of the program's components. This separation also aligns with security principles, enabling control over the permissions associated with code and data to prevent unintended modifications and maintain the integrity of the executable.

However, making changes to the internal structure of a Portable Executable (PE) file, can be quite challenging. As it often involves rewriting entire structures, such as headers or data sections, within the file. Additionally, adjusting these structures may require recalculating the positions of various components, such as section headers, due to space constraints. To tackle this complexity rather than directly modifying the PE structures, we propose the strategy of introducing a "meta-writer structure". This meta-writer structure doesn't belong to the original PE file but serves as a higher-level guide. It keeps track of essential information needed to recreate the existing PE file while also incorporating details about any new sections being introduced. Essentially, it acts as a helpful tool for managing the intricacies of modifying the PE file without the need for a complete overhaul of its internal structures. Moreover, the dynamic nature of this process necessitates reading and rewriting PE binaries, requiring a library designed for this purpose. Currently, the only available library is written in Go, while Lanzaboote is implemented in Rust. One of our objectives is to develop a secure, memory-efficient PE library to address this limitation.

The existing library for writing PE binaries, Objcopy, was replaced with one that provided Rust-based functionality. Goblin, the Rust-based PE library, supports PE write support. The subsequent challenge involved signing, which in turn involves storing certificates and private keys, within the data section, securely. To achieve this, the industry-standard Public-Key Cryptography Standard #11 (PKCS#11) protocol was employed, allowing secure token storage (e.g., TPM2, YubiKey) and defining an API for signing operations.

The PKCS#11 standard, though effective, is known for its complexity. To simplify the signing process, the Goblin Signing subcrate for Rust has been introduced. It not only streamlines the signing procedure but also adheres to Microsoft's signing scheme, a hierarchical code signing structure based on multiple certificates and certificate authorities. This intricate signing scheme requires careful handling, especially regarding file hashing to prevent vulnerabilities. Goblin Signing, as part of the upstream features, addresses these complexities, ensuring a robust and secure signing process in the UEFI and PE context.

## A/B Schema in NixOS
A/B schema, in this context, refers to general methodologies for facilitating primary and secondary booting, which involves the existence of two distinct boot partitions. During an upgrade, updated content is written to the primary partition. In the event of a boot failure, the system automatically switches to the secondary partition, executing a rollback of the unsuccessful update. This conventional approach safeguards embedded systems from disruptions during upgrades by maintaining two separate boot partitions that are not simultaneously upgraded.

 Within NixOS and, more specifically leveraging, systemd-boot, a mechanism has been implemented that generalises this concept to the NixOS generation system to monitor boot occurrences and initiate automatic fallbacks. This feature is referred to as "Automatic Boot Assessment" and it works as follows: During the initial installation of boot entries, they are designated as "indeterminate" and assigned a specific number of boot attempts. With each boot attempt, this count is decremented. If the operating system deems a boot successful, the counter is removed, and the entry is labeled as "good." Conversely, if the designated number of boots is exhausted without a successful boot, the entry is marked as "bad."

The feature can be enabled by toogling the `boot.loader.systemd-boot.bootCounting` option to `true`.

The criterion for a successful boot of an entry is determined by the presence of the `boot-complete.target` value in the NixOS configuration file, which serves as a synchronisation point, that is, a reference point where certain critical tasks or dependencies necessary for the system to function properly have been satisfied. It is the user's responsibility to schedule all required services, called "units" in systemd jargon, to ensure the machine is considered successfully booted before reaching this synchronisation point. To illustrate this, if a machine runs `nsd`, an authoritative DNS server, and the goal is to define a "good" entry as one where this DNS server starts successfully. A configuration for that NixOS machine could look like that:

```
boot.loader.systemd-boot.bootCounting.enable = true;
services.nsd.enable = true;
/* rest of nsd configuration omitted */

systemd.services.nsd = {
  before = [ "boot-complete.target" ];
  wantedBy = [ "boot-complete.target" ];
  unitConfig.FailureAction = "reboot";
};
```
 
With the boot-counting feature enabled, systemd-boot will attempt the boot entries in the same sequence as they appear in the boot menu. Consequently, the specialisations of a particular generation will be tried immediately after that generation. It's important to note that marking a generation as bad doesn't automatically designate its specialisations as bad, as these specialisations might still boot successfully.

 It's crucial that this feature is used with the proper judgement in order to avoid any potential data integrity issues. Rolling back to previous generations can be risky, especially if certain services exhibit undefined behavior when faced with unrecognized data migrations from future versions of themselves. While not directly related to boot security, this feature is integral to the autonomy of systems on NixOS, impacting both boot functionality and overall security. Its presence is crucial, as it significantly influences users' willingness to engage in boot processes and system upgrades.

## Integrity checks in NixOS

When integrating Secure Boot with NixOS one encounters certain limitations during the transition to Stage 2. This challenge arises from the necessity of implementing a mechanism to verify the operating system closure stored in the Nix store partition or file system. Currently, it is common to simply rely on disk encryption, such as Linux Unified Key Setup (LUKS), to keep the Nix store safe from tampering, but this is not always desirable, e.g. for devices that should boot unattended. Integrity checks are most useful where ensuring that critical files, block devices, and executables remain unaltered and trustworthy such as in environments where the protection of system integrity is a high priority, as in the case of certain embedded systems, servers, or devices that store sensitive data.

Various kernel tools can be considered for system integrity purposes, each with tradeoffs. For instance, dm-verity uses a Merkle tree to validate blocks, but its read-only nature and the challenge of simultaneous updates lead to space-consuming disk images and slow processes. Similarly, fs-verity, which operates at the file level, lacks the ability to verify file positions, posing a security risk. Other tools like Integrity Measurement Architecture (IMA) and Extended Verification Module (EVM) share the benefits and drawbacks with fs-verity, working at the file level without verifying file locations. IMA, however, is less flexible due to rigid policies, making it more suitable for auditing than comprehensive boot verification.

Ultimately, the choosen design uses Nix’s own signature verification system (i.e. `nix-store verify`) to verify the system closure before transitioning to stage two. This was chosen for its relative simplicity, without compromising security. It’s also extremely easy to build and deploy, given that it’s already a feature of Nix itself.

During the system's boot process, after the operating system's file  has been mounted, the Nix command-line interface is employed to systematically validate each path within the system closure, ensuring that it possesses a valid Nix signature. This verification process serves the purpose of establishing trust between the initial stage (stage 1) and the subsequent operating system (stage 2) that it is poised to transition to. Importantly, this trust is solidified before permitting the execution of any code from stage 2. The inclusion of this mechanism introduces new NixOS options, namely 

- `boot.initrd.verify.enable`
- `boot.initrd.verify.sigsNeeded`
- `boot.initrd.verify.trustedPublicKeys`

which can be used to configure which public keys stage 1 should trust, as well as how many of those keys need to have signed every individual path for that path to be trusted.

These options control whether the system should sign its own closures when installing the boot loader. This is not done by default, with the assumption that the system closure is built and signed by a trusted builder:

- `boot.initrd.verify.signing.enable`
- `boot.initrd.verify.signing.keyFile`


The implementation for these options are reflected in these files:
- `nixos/modules/system/boot/initrd-verify.nix`  
- `nixos/modules/system/boot/systemd/initrd.nix`

If you enable this, ensure the key file is only accessible when absolutely necessary. An example of how this can be used is as follows:

First, generate a signing key via the command:

`nix-store --generate-binary-cache-key key-name secret-key-file public-key-file`

In a NixOS module, configure stage 2 verification.

```
{
  boot.initrd.verify = {
    enable = true;
    sigsNeeded = 1;
    trustedPublicKeys = [(builtins.readFile ./public-key-file)];
  };
}
```
For autonomous deployment, securely store the secret-key file in an encrypted location and set up the `signing.enable` and `signing.keyFile` options. It's advisable to restrict access to this file only when deploying updates.

Otherwise, store this key on the system that will build the system closure, and configure its `/etc/nix/nix.conf` settings with `secret-key-files` pointing to the secret. When deploying, make sure to copy the closure’s signatures as well.

This approach allows one to continue using an ordinary Nix store file system, meaning no new disk images need to be constructed, and the system can be used like an ordinary NixOS system. New generations can be added without large storage requirements for every single one, because it’s just an ordinary Nix store. While it does verify stage 2, it does so by delaying boot to read and verify the entire closure. On a fast computer, this takes 5-10 seconds. On a slower system (e.g. a raspberry pi using an SD card for the OS), this can take a few minutes.

## Interpreter-less Nix
