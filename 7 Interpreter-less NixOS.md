Relevant PRs
- [Replace simple activationScripts  #263203](https://github.com/NixOS/nixpkgs/pull/263203)
- [systemd: enable sysusers by default #264879](https://github.com/NixOS/nixpkgs/pull/264879)
- [Rebuildable system & appliance #263462](https://github.com/NixOS/nixpkgs/pull/263462)

The upcoming milestone focuses on enhancing system security to an uncompromisable level. In the context of NixOS or software in general, programs fall into two categories: compiled programs with a singular purpose and interpreters. Interpreters, like Python, can execute arbitrary code upon command, offering broad code execution capabilities. However, this flexibility poses a security risk during the boot process, where bash and Perl scripts are employed in Nix packages and switch configurations. Though challenging, an attacker could potentially inject malicious code into these scripts, compromising boot security.

The Interpreter-less NixOS initiative aims to create a NixOS system that cannot be easily rebuilt. By setting system rebuildable=false in the NixOS module and preventing further switch configurations, the system becomes resistant to reconstruction. Moreover, this project endeavours to minimise the presence of interpreters in the boot chain, replacing bash scripts, Perl scripts, and other elements with systemd native code. The ultimate goal is to eliminate interpreters entirely, making it exceedingly difficult to mount an attack on such a system.
