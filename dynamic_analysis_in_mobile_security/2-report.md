# Network Interception & Cryptographic Decryption Report: Apk_task2

## Objective
Intercept encrypted HTTPS traffic between `Apk_task2.apk` and its server, bypass dynamic SSL pinning controls, identify the symmetric encryption key/IV via decompilation, and decrypt the transmitted flag.

## Key Findings & Cryptographic Parameters
- **Interception Tool:** Burp Suite Community / Objection
- **SSL Pinning Bypass Method:** Dynamic runtime instrumentation via Frida / Objection `android sslpinning disable`
- **Symmetric Cipher:** `AES/CBC/PKCS5Padding`
- **Secret Key:** `321c_s3cr3t_k3y!`
- **Initialization Vector (IV):** `1234567890abcdef`
- **Extracted Flag:** `FLAG{a3s_cbc_n3tw0rk_d3cryp110n_succ3ss_2026}`

## Technical Methodology
1. **Traffic Capture & Pinning Bypass:** Intercepted outbound HTTPS traffic by installing Burp Suite CA onto the test device and disabling OkHttp/TrustManager pinning logic using Objection.
2. **Decompilation:** Reverse-engineered the application using `jadx-gui` to locate network response handling inside `com.example.cryptoapp.network.CryptoManager`.
3. **Parameter Extraction:** Extracted hardcoded AES-128 key and IV constants from decompiled source code.
4. **Decryption:** Decoded Base64 server response payload and applied offline AES-CBC decryption to extract the hidden plaintext flag.
