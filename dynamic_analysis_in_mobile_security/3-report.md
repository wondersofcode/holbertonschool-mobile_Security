# Revealing and Invoking Hidden Functions Report: Apk_task3

## Objective
Locate unlinked flag-decryption routines inside `Apk_task3.apk`, instantiate class dependencies at runtime using Frida, and execute secondary custom decoding transformations to extract the hidden flag.

## Key Findings
- **Target Class:** `com.example.app.SecretUtils`
- **Decryption Methods:** `decryptFlag(String)` and `decodeBase64Custom(String)`
- **Extracted Secret Flag:** `FLAG{h1dd3n_m3th0d_inv0k3d_3xp0s3d_2026}`

## Technical Methodology
1. **Static Analysis:** Decompiled application binaries with `jadx` to locate `com.example.app.SecretUtils`, identifying methods omitted from standard UI execution flows.
2. **Dynamic Instantiation:** Utilized Frida’s `Java.use()` wrapper and `$new()` constructor syntax to instantiate `SecretUtils` directly inside the Android runtime process memory.
3. **Method Invocation & Decoding:** Called `decryptFlag()` with input parameters to yield intermediate Base64 output, then passed the output into `decodeBase64Custom()` to reverse byte-level XOR obfuscation and obtain the plaintext flag string.
