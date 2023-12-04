PE binaries

Relevant PRs
 - [Write support for PE binaries m4b/goblin#361](https://github.com/m4b/goblin/pull/361)
 - [pe: offer basic section manipulations m4b/goblin#381](https://github.com/m4b/goblin/pull/381)
 - [project: initial proof of concept RaitoBezarius/goblin-signing#2](https://github.com/RaitoBezarius/goblin-signing/pull/2)
 - [feat: simple verify algorithms RaitoBezarius/goblin-signing#3](https://github.com/RaitoBezarius/goblin-signing/pull/3)

In the UEFI environment, the use of Portable Executable (PE) and Microsoft components is essential. However, a significant challenge arises in building PE binaries without a straightforward method, as it typically requires a compiler. Assembling a stub involves writing a PE binary, and while template stubs exist, they lack dynamically generated sections.

The dynamic nature of this process necessitates reading and rewriting PE binaries, requiring a library designed for this purpose. Currently, the only available library is written in Go, while Lanzaboote is implemented in Rust. Our objective is to develop a secure, memory-efficient PE library to address this limitation.

Our dual goals include replacing the existing complex library, Objcopy, and providing Rust-based functionality. Goblin, a Rust-based PE library, now supports PE write support. The subsequent challenge involves signing, which involves storing certificates and private keys securely. To achieve this, the industry-standard PKCS#11 11 protocol is employed, allowing secure token storage (e.g., TPM2, YubiKey) and defining an API for signing operations.

The PKCS#11 standard, though effective, is known for its complexity. To simplify the signing process, the Goblin Signing subcrate for Rust has been introduced. It not only streamlines the signing procedure but also adheres to Microsoft's signing scheme, a hierarchical code signing structure based on multiple certificates and certificate authorities.

This intricate signing scheme requires careful handling, especially regarding file hashing to prevent vulnerabilities. Globin Signing, as part of the upstream features, addresses these complexities, ensuring a robust and secure signing process in the UEFI and PE context.
