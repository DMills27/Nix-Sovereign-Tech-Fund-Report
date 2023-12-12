## Bootspec V2

Relevant PRs

- [[RFC 0165] Bootspec v2 rfcs#165](https://github.com/NixOS/rfcs/pull/165)


Bootspec refers to a JSON document that contains metadata related to the boot process of a NixOS generation, which functions as an intermediate bridge (IR) between the NixOS system definition and the tools tasked with managing the bootloader.
Some of these tools include:

* Multiple Profiles: which allows users to maintain different configurations or sets of packages. Each profile represents a distinct environment with its own package set, dependencies, and configurations. This enables users to switch between different system states or use cases easily.

* Generation Limits: In the context of NixOS, a generation limit refers to the maximum number of past generations (system configurations) that are retained.

* Initrd Secrets: Initrd (initial ramdisk) secrets are cryptographic keys required during the initial stages of the system boot process. These secrets are used to unlock encrypted partitions or perform other security-related tasks before the main operating system is fully loaded.

* Specialisation: Specialisation in NixOS involves tailoring the system configuration to specific use cases or requirements, such as development, server hosting, or desktop usage.

* Custom Labels: Custom labels refer to user-defined labels or tags associated with NixOS configurations. These labels can be used to identify and organize different system states, profiles, or generations.


The main motivation behind bootspec is to support a uniform collection of features and design decisions, such as those previously listed, which aren't often supported partially due to the complexity of implementing the features in a particular bootloader such as those on a RaspberryPI or Extlinux. When they are supported it is often through globbing directories, that is, by specifying sets of filenames with wildcard characters then utilizing these wildcards to match and locate files according to a predefined pattern. This approach of globbing directories poses several challenges, the main of which is the inherent complexity and significant effort when  introducing new bootloader features or enhancing older ones. 
Bootspe, thus, addresses the challenges of feature extension in bootloaders, reducing the fragility and maintenance issues associated with disparate public APIs in nixpkgs. 

Bootspec V1, the first iteration of Bootspec, addressed various issues but exhibited gaps, such as the lack of information about device trees, which are data structures used to describe the hardware configuration of a system, particularly in embedded systems and open architecture platforms. Bootspec V2 addresses this by reevaluating the initrd secret feature, which has proven insecure and problematic since it starts to omit a cyptographic key file when upgrading the version of NixOS. In Bootspec V2, the goal is to replace initrd secrets with systemd-creds, which securely stores and retrieves credentials used by systemd units, to significantly enhance security in the NixOS boot process. Users can encrypt these credentials to TPM2, store them in /boot (even on the boot partition), and then have TPM2 unlock them before booting. This approach ensures compatibility with a broader range of systems (such as remote desktops, server farms, etc).

Bootspec serves as the principal input for bootloader backends such as systemd-boot and GRUB, facilitating the generation of files in `/boot/loader/entries/` and `grub.cfg` directories. 

 Bootspec will ultimately foster well-defined public APIs, discourage ad-hoc measures taken when booting up, and prevents the incorporation of extraneous data within the generation boot data, which is concealed from the user.

