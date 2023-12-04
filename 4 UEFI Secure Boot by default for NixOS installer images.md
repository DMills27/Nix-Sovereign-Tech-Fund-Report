Relevant PRs

- [https://github.com/RaitoBezarius/nixos-shim](https://github.com/RaitoBezarius/nixos-shim)
- [https://hydra.newtype.fr/project/nixos-shim](https://hydra.newtype.fr/project/nixos-shim)

Lanzaboote facilitates the signing of custom Secure Boot components with user-defined keys, such as  Microsoft keys. However, it's important to note that signing on behalf of Microsoft is not possible without access to their private key. A common challenge arises during the installation of NixOS on a new laptop with Secure Boot, as the initial step involves enrolling personal keys, that is, adding or registering keys into the system's firmware, which can be intricate.

To streamline this process, we are actively involved in the NixOS shim project. This initiative entails constructing a NixOS shim, developing a systemd boot second-stage bootloader, and creating a final ISO/image for submission to the shim-review project. Shim-review serves as a community-driven solution endorsed by Microsoft, allowing open-source contributors to request signature validation without incurring the standard 20k fee. Users can open an issue, specifying their signing intentions, and Microsoft signs the shim exclusively. This approach simplifies the Secure Boot process for NixOS installation, requiring management of machine-owned or custom keys rather than intricate secure boot keys.
