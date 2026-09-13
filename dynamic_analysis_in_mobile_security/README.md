# Dynamic Analysis in Mobile Security

> *"Static analysis reveals the blueprint of an application, but dynamic analysis exposes its true behavior."*

---

## 📌 Introduction

Static analysis provides essential structural insights, but modern mobile applications routinely employ heavy code obfuscation, dynamic class loading, runtime decryption, and anti-tampering controls designed to defeat offline decompilation. Critical security vulnerabilities, unlinked routines, and sensitive data flows often manifest exclusively while the application is executing in memory.

**Dynamic Analysis in Mobile Security** transitions the assessment focus from reading static source code to actively manipulating the runtime environment. By leveraging dynamic instrumentation frameworks like **Frida** and **Objection**, along with network proxies such as **Burp Suite** and **mitmproxy**, security researchers can hook active Java and native C/C++ functions, bypass root detection and SSL certificate pinning, force the execution of hidden code paths, and decrypt network payloads in real time.

---

## 🎯 Why It Matters

Dynamic analysis provides decisive operational advantages when assessing hardened Android applications:

* **Runtime Visibility:** Observing memory allocations, active API parameters, and cryptographic transformations directly during application execution.
* **Security Control Bypass:** Neutralizing anti-debugging routines, client-side validation logic, root enforcement checks, and SSL certificate pinning dynamically without modifying the underlying APK package on disk.
* **Native Memory Interception:** Hooking lower-level C/C++ functions exposed through the Java Native Interface (JNI) to inspect unencrypted strings and keys processed in `.so` shared libraries.
* **Dead Code Activation:** Dynamically instantiating unlinked classes and invoking dormant functions hidden within application packages to evaluate isolated threat surfaces.

---

## 🧠 Learning Objectives

Upon completing this project, the following dynamic analysis skills, instrumentation frameworks, and exploitation methodologies are mastered:

* **Dynamic vs. Static Analysis Trade-offs:** Determining when static decompilation must be complemented by runtime memory instrumentation and function hooking.
* **Runtime Instrumentation with Frida:** Writing custom JavaScript injection hooks to alter method arguments, overwrite return values, and trace internal method invocation trees.
* **Rapid Runtime Auditing with Objection:** Utilizing Objection to automate memory exploration, dump heap objects, evaluate active activity stacks, and bypass client-side protections.
* **Native JNI Function Hooking:** Intercepting native C/C++ functions in shared libraries (`.so`) using Frida's `Interceptor` API (`Interceptor.attach`) to extract dynamic memory buffers.
* **Network Interception & SSL Pinning Bypass:** Intercepting encrypted HTTPS application traffic via Proxy tools (Burp Suite / mitmproxy) and neutralizing custom trust managers and network security configs dynamically.
* **Hidden Function Invocation:** Dynamically discovering and instantiating hidden or uncalled Java classes (`Java.choose` / `Java.use`) to execute secret flag generation and decryption routines.

---

## 🛠️ Mobile Dynamic Analysis Tooling Matrix

All assessments are conducted within a controlled local environment using an instrumented Android emulator (x86/x86_64) or physical test device connected to a Kali Linux workstation.

| Tool Category | Permitted Utilities | Primary Operational Role |
| --- | --- | --- |
| **Dynamic Instrumentation** | Frida, Objection | Inject JavaScript code into running process memory, hook Java/Native methods, bypass security checks. |
| **Bridge & Device Interface** | Android Debug Bridge (ADB) | Control device state, manage shell sessions, push/pull files, inspect `logcat` telemetry. |
| **Network Interception** | Burp Suite, mitmproxy, Wireshark | Intercept, log, modify, and analyze cleartext and encrypted HTTP/HTTPS traffic. |
| **Decompilation & Disassembly** | JADX, APKTool, Ghidra, IDA Pro | Locate target class definitions, method signatures, JNI exports, and resource files statically prior to hooking. |
| **Native Debugging** | GDB, Android Studio Profiler | Attach to native processes, inspect stack frames, and set hardware/software breakpoints. |

---

## 📂 Repository Layout

```
holbertonschool-mobile_Security/
└── dynamic_analysis_in_mobile_security/
    ├── 0-flag.txt
    ├── 1-flag.txt
    ├── 1-report.md
    ├── 2-flag.txt
    ├── 2-report.md
    ├── 3-flag.txt
    └── 3-report.md

```

---

## ⚡ Technical Tasks & Implementation Details

### Task 0: Android App Security (`Apk_task0`)

* **Objective:** Perform dynamic analysis on `Apk_task0.apk` to intercept a key generation method, iterate over valid seed values at runtime using a custom Frida script, and extract the secret flag.
* **Execution & Analysis Walkthrough:**
1. Decompile `Apk_task0.apk` using JADX to locate the target flag-generation method signature (e.g., `com.example.app.FlagGenerator.generateString(int seed)`).
2. Spawn and attach Frida to the running application process via ADB:
```bash
frida -U -f com.example.app -l hook_seed.js

```


3. Execute an automated parameter brute-force loop inside the injected JavaScript context:
```javascript
Java.perform(function () {
    var FlagClass = Java.use("com.example.app.FlagGenerator");

    for (var seed = 0; seed <= 1000; seed++) {
        var result = FlagClass.generateString(seed);
        if (result.startsWith("Holberton{")) {
            console.log("[+] Valid Seed Found: " + seed);
            console.log("[+] Revealed Flag: " + result);
            break;
        }
    }
});

```


4. Record the extracted flag string into `0-flag.txt`.


* **Deliverable Path:** `dynamic_analysis_in_mobile_security/0-flag.txt`

---

### Task 1: Hooking Native Functions in Android (`Apk_task1`)

* **Objective:** Analyze an Android application utilizing native libraries via the Java Native Interface (JNI). Intercept the native C/C++ function `getSecretMessage` inside `libnative-lib.so` to extract the decrypted string directly from process memory.
* **Execution & Analysis Walkthrough:**
1. Inspect the loaded libraries and exported JNI functions using Frida's `Module` API or Ghidra disassembly.
2. Construct a Frida script attaching an `Interceptor` hook to the native function memory address:
```javascript
var moduleName = "libnative-lib.so";
var nativeFuncOffset = 0x1234; // Address or exported symbol

var targetAddr = Module.findExportByName(moduleName, "Java_com_example_app_Native_getSecretMessage");

Interceptor.attach(targetAddr, {
    onEnter: function (args) {
        console.log("[*] Intercepted native function call");
    },
    onLeave: function (retval) {
        var env = Java.vm.getEnv();
        var jstring = ptr(retval);
        var cString = env.getStringUtfChars(jstring, null).readCString();
        console.log("[+] Native Function Returned Decrypted Flag: " + cString);
    }
});

```


3. Extract the decrypted flag string into `1-flag.txt` and document the disassembly, memory layout, and hooking steps in `1-report.md`.


* **Deliverables Path:** `dynamic_analysis_in_mobile_security/1-flag.txt`, `1-report.md`

---

### Task 2: Intercepting and Decrypting Network Data (`Apk_task2`)

* **Objective:** Intercept encrypted HTTP/HTTPS traffic between `Apk_task2.apk` and its remote server, bypass SSL pinning mechanisms, analyze the cryptographic routine (AES/RSA/Base64), and decrypt the transmitted flag.
* **Execution & Analysis Walkthrough:**
1. Configure an interception proxy (Burp Suite / mitmproxy) on the Kali workstation and import the proxy CA certificate into the Android device trust store.
2. Inject an Objection / Frida SSL Pinning bypass script to bypass custom TrustManager and OkHttp pinning protections:
```bash
objection --g com.example.app explore -s "android sslpinning disable"

```


3. Capture the encrypted server HTTP response payload in Burp Suite.
4. Decompile the APK to identify the secret AES key, initialization vector (IV), or RSA decryption functions used to process the payload.
5. Decrypt the captured payload offline or via Frida method invocation to retrieve the hidden flag.


* **Deliverables Path:** `dynamic_analysis_in_mobile_security/2-flag.txt`, `2-report.md`

---

### Task 3: Revealing and Invoking Hidden Functions (`Apk_task3`)

* **Objective:** Locate hidden or unlinked decryption methods within `Apk_task3.apk` that are omitted from standard execution paths, dynamically invoke them in process memory using Frida, and reverse any secondary encoding to obtain the secret flag.
* **Execution & Analysis Walkthrough:**
1. Perform static analysis with JADX to identify uncalled target classes and unlinked flag-decryption functions (e.g., `com.example.app.SecretUtils.decryptFlag()`).
2. Write a Frida script to instantiate the class or locate live heap instances using `Java.choose()` / `Java.use()`:
```javascript
Java.perform(function () {
    var SecretUtils = Java.use("com.example.app.SecretUtils");
    var instance = SecretUtils.$new(); // Instantiate object in heap

    var encodedFlag = instance.decryptFlag("enc_param_value");
    console.log("[+] Raw Encoded Output: " + encodedFlag);

    // Pass raw string into local transformation decoder
    var finalFlag = instance.decodeBase64Custom(encodedFlag);
    console.log("[+] Uncovered Secret Flag: " + finalFlag);
});

```


3. Save the retrieved flag to `3-flag.txt` and document the reverse-engineering methodology in `3-report.md`.


* **Deliverables Path:** `dynamic_analysis_in_mobile_security/3-flag.txt`, `3-report.md`

---

## 🔬 Dynamic Instrumentation Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                 Target APK Deployment & Launch               │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                Frida / Objection Session Spawn              │
├─────────────────────────────────────────────────────────────┤
│  • Bypass Root Detection Checks                             │
│  • Neutralize SSL Certificate Pinning                       │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│             Runtime Execution & Memory Interception         │
└──────────────┬───────────────────────────────┬──────────────┘
               │                               │
               ▼                               ▼
┌──────────────────────────────┐┌──────────────────────────────┐
│     Java / ART Runtime       ││     Native C/C++ (JNI)       │
├──────────────────────────────┤├──────────────────────────────┤
│ • Hook Java Methods          ││ • Attach Interceptor to .so  │
│ • Brute-force Seed Loops     ││ • Read Native Pointer Buffers│
│ • Instantiate Hidden Classes ││ • Intercept Dynamic Symbols  │
└──────────────┬───────────────┘└──────────────┬───────────────┘
               │                               │
               └───────────────┬───────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│             Network Interception & Flag Decryption          │
└─────────────────────────────────────────────────────────────┘

```

---

## 🛡️ Defensive Hardening Matrix

> [!IMPORTANT]
> ### 1. Advanced Anti-Frida & Anti-Hooking Controls
> 
> 
> Implement runtime checks that inspect `/proc/self/maps` for Frida artifacts (`re.frida.server`), detect listening TCP ports (`27042`), and verify function prologues for software breakpoints (`0x2D00007B` / `BKPT`).
> ### 2. Robust Root & Emulator Detection
> 
> 
> Perform multi-factored integrity checks verifying system binary paths (`/system/xbin/su`), build tags (`ro.build.tags=test-keys`), device properties, and SElinux status (`getenforce`).
> ### 3. Hardware-Backed Certificate Pinning
> 
> 
> Implement network security policies backed by Android KeyStore and hardware-backed attestation to prevent intercepting proxies from injecting user-installed CA certificates.
> ### 4. Native Code Obfuscation & Dynamic JNI Registration
> 
> 
> Use OLLVM to flatten control flow inside C/C++ native libraries and register JNI functions dynamically (`JNINativeMethod` tables inside `JNI_OnLoad`) to prevent simple static symbol exports.

---

## ⚠️ Disclaimer

> [!WARNING]
> This repository is maintained exclusively for educational purposes, mobile security research, and authorized penetration testing exercises. Hooking, intercepting, or reverse-engineering third-party mobile applications without explicit written authorization from the system owner is strictly prohibited.
