PE binaries

Relevant PRs
 - [Write support for PE binaries m4b/goblin#361](https://github.com/m4b/goblin/pull/361)
 - [pe: offer basic section manipulations m4b/goblin#381](https://github.com/m4b/goblin/pull/381)
 - [project: initial proof of concept RaitoBezarius/goblin-signing#2](https://github.com/RaitoBezarius/goblin-signing/pull/2)
 - [feat: simple verify algorithms RaitoBezarius/goblin-signing#3](https://github.com/RaitoBezarius/goblin-signing/pull/3)

Portable Executable (PE) binaries refer to executable files that adhere to the PE file format. The PE format is a file structure used for executables, object code, DLLs (Dynamic Link Libraries), and other types of files used in Windows operating systems. PE files have a structured layout that includes different sections, each serving a specific purpose. The two primary types of sections are the "code section" and the "data section":

- **Code Section**: The code section contains the actual executable instructions or machine code that the computer's processor executes. The code section is marked as executable and read-only to ensure that the program's instructions are not modified during runtime.

- **Data Section**: The data section, on the other hand, stores various types of data that the program uses during its execution. This data can include global variables, constants, static data structures, and other information that needs to be stored in the executable file. Unlike the code section, the data section may have read and write permissions, allowing the program to modify its own data during runtime.

Of particular interest, for this project, is the data section which includes critical information such as digital signatures, checksums, configuration settings, or other data necessary for the proper functioning and security of the executable. Read and write support for the data section is crucial for verifying and updating this information as needed. By having separate sections for code and data, PE files follow a modular structure that allows for efficient organization and management of the program's components. This separation also aligns with security principles, enabling control over the permissions associated with code and data to prevent unintended modifications and maintain the integrity of the executable.

However, making changes to the internal structure of a Portable Executable (PE) file, can be quite challenging. As it often involves rewriting entire structures, such as headers or data sections, within the file. Additionally, adjusting these structures may require recalculating the positions of various components, such as section headers, due to space constraints. To tackle this complexity rather than directly modifying the PE structures, we propose the strategy of introducing a "meta-writer structure". This meta-writer structure doesn't belong to the original PE file but serves as a higher-level guide. It keeps track of essential information needed to recreate the existing PE file while also incorporating details about any new sections being introduced. Essentially, it acts as a helpful tool for managing the intricacies of modifying the PE file without the need for a complete overhaul of its internal structures. Moreover, the dynamic nature of this process necessitates reading and rewriting PE binaries, requiring a library designed for this purpose. Currently, the only available library is written in Go, while Lanzaboote is implemented in Rust. One of our objectives is to develop a secure, memory-efficient PE library to address this limitation.

Our dual goals include replacing the existing complex library, Objcopy, and providing Rust-based functionality. Goblin, a Rust-based PE library, now supports PE write support. The subsequent challenge involves signing, which involves storing certificates and private keys, within the data section, securely. To achieve this, the industry-standard Public-Key Cryptography Standard #11 (PKCS#11) protocol is employed, allowing secure token storage (e.g., TPM2, YubiKey) and defining an API for signing operations.

The PKCS#11 standard, though effective, is known for its complexity. To simplify the signing process, the Goblin Signing subcrate for Rust has been introduced. It not only streamlines the signing procedure but also adheres to Microsoft's signing scheme, a hierarchical code signing structure based on multiple certificates and certificate authorities. This intricate signing scheme requires careful handling, especially regarding file hashing to prevent vulnerabilities. Goblin Signing, as part of the upstream features, addresses these complexities, ensuring a robust and secure signing process in the UEFI and PE context.