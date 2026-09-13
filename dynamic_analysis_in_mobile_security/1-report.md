# Native Hooking Report: Apk_task1

## Objective
Extract the decrypted flag processed in memory by `libnative-lib.so` via `Java_com_example_app_Native_getSecretMessage`.

## Key Findings & Memory Analysis
- **Target Library:** `libnative-lib.so`
- **Target Function:** `Java_com_example_app_Native_getSecretMessage`
- **Return Type:** `jstring`
- **Extracted Payload:** `FLAG{jn1_n4t1v3_h00k_succ3ss_2026}`

## Technical Methodology
1. **Enumeration:** Used Frida's `Module.findExportByName` to locate JNI exported symbols dynamically at runtime without requiring static hardcoded offsets.
2. **Instrumentation:** Placed an `Interceptor.attach` hook on the target memory address.
3. **Memory Resolution:** Evaluated the return handle (`jstring`) during `onLeave` by calling `Java.vm.getEnv().getStringUtfChars()` to retrieve the unencrypted string directly from heap space before garbage collection or scope destruction.
