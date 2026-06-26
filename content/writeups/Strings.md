---
title: "Mobile Hacking Lab: Strings"
date: 2026-06-25
weight: 4
tags: ["mobilehackinglab","android","frida","jadx"]
author: "Gourav."
showToc: true
TocOpen: false
hidemeta: false
description: "Android deep link intent injection with Frida instrumentation — Challenge Lab Writeup."
summary: "Android deep link intent injection with Frida instrumentation — Challenge Lab Writeup."
hideSummary: false
ShowReadingTime: true
ShowPostNavLinks: true
ShowWordCount: true
UseHugoToc: true
cover:
    image: "/images/strings_cover.svg"
    alt: "Strings challenge"
    caption: "Strings challenge (Android RE)"
    relative: false
    hidden: false
---

{{< callout warning >}}

**Lab:** **Strings**

**Category:** Android Reverse Engineering / Frida

**Vulnerability:** Deep link intent injection + hardcoded cryptographic secrets

**Platform:** Mobile Hacking Lab
{{< /callout>}}

## Lab Description

{{< callout insight "Briefing" >}}
*Welcome to the Strings Challenge! In this lab, your goal is to find the flag. The flag's format should be "MHL{...}". The challenge will give you a clear idea of how intents and intent filters work on Android, also you will get a hands-on experience using Frida APIs.*
{{< /callout>}}

---

## Initial Analysis

Decompiling the APK in jadx reveals two activities — `MainActivity` (the launcher) and `Activity2`. `Activity2` is `exported="true"` and carries a custom intent filter, which immediately signals that it's reachable from outside the app via a deep link.

```xml
<activity
    android:name="com.mobilehackinglab.challenge.Activity2"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data
            android:scheme="mhl"
            android:host="labs"/>
    </intent-filter>
</activity>
```

{{< figure align="center" src="/images/Strings_1.png" caption="AndroidManifest.xml in jadx — Activity2 exported with a custom mhl://labs deep link intent filter" >}}

Key observations:

- **Exported** — any app on the device can start this activity
- **Action: `VIEW`** — designed to handle deep links
- **Scheme: `mhl`** — a custom URI scheme embedded in the manifest
- **Host: `labs`** — restricts the URI authority

This means `Activity2` is reachable via `mhl://labs/...` deep links.

## Decompiling the Validation Logic

Decompiling `Activity2.onCreate()` reveals a multi-gate validation flow that must all pass before the flag is delivered:

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_2);

    SharedPreferences sharedPreferences = getSharedPreferences("DAD4", 0);
    String u_1 = sharedPreferences.getString("UUU0133", null);
    boolean isActionView = Intents.areEqual(
        getIntent().getAction(), "android.intent.action.VIEW"
    );
    boolean isU1Matching = Intents.areEqual(u_1, cd());

    if (isActionView && isU1Matching) {
        Uri uri = getIntent().getData();
        if (uri != null
            && Intents.areEqual(uri.getScheme(), "mhl")
            && Intents.areEqual(uri.getHost(), "labs")) {

            String base64Value = uri.getLastPathSegment();
            byte[] decodedValue = Base64.decode(base64Value, 0);

            if (decodedValue != null) {
                String ds = new String(decodedValue, Charsets.UTF_8);
                byte[] bytes = "your_secret_key_1234567890123456"
                    .getBytes(Charsets.UTF_8);
                String str = decrypt("AES/CBC/PKCS5Padding",
                    "bqGrDKdQ8zo26HflRsGvVA==",
                    new SecretKeySpec(bytes, "AES"));

                if (str.equals(ds)) {
                    System.loadLibrary("flag");
                    String s = getflag();
                    Toast.makeText(getApplicationContext(), s, 1).show();
                }
            }
        }
    }
}
```

{{< figure align="center" src="/images/Strings_2.png" caption="Decompiled Activity2.onCreate() in jadx — the full three-gate validation flow" >}}

### Gate 1 — Intent Action

The intent action must be `android.intent.action.VIEW`. This is satisfied trivially by any deep link intent.

### Gate 2 — Date-Based SharedPreference Check

The code reads a preference key {{< badge blue >}}UUU0133{{< /badge>}} from the {{< badge gray >}}DAD4{{< /badge>}} shared preferences file and compares it against the return value of `cd()`, which formats the current date:

```java
private final String cd() {
    SimpleDateFormat sdf = new SimpleDateFormat(
        "dd/MM/yyyy", Locale.getDefault()
    );
    String str = sdf.format(new Date());
    Activity2Kt.cu_d = str;
    return str;
}
```

The preference is populated by `KLOW()`, a method on `MainActivity`:

```java
public final void KLOW() {
    SharedPreferences sharedPreferences = getSharedPreferences("DAD4", 0);
    SharedPreferences.Editor editor = sharedPreferences.edit();
    SimpleDateFormat sdf = new SimpleDateFormat(
        "dd/MM/yyyy", Locale.getDefault()
    );
    String cu_d = sdf.format(new Date());
    editor.putString("UUU0133", cu_d);
    editor.apply();
}
```

{{< figure align="center" src="/images/Strings_7.png" caption="MainActivity.java in jadx — KLOW() stores today's date into the DAD4 SharedPreferences file" >}}

So calling `KLOW()` before the deep link sets the date preference to today's date, which matches what `cd()` returns. Gate 2 becomes a non-issue at that point.

### Gate 3 — Base64 + AES Decryption

The last path segment of the deep link URI is Base64-decoded and compared against the decryption of a hardcoded ciphertext:

```java
String base64Value = uri.getLastPathSegment();
byte[] decodedValue = Base64.decode(base64Value, 0);
String ds = new String(decodedValue, Charsets.UTF_8);

String str = decrypt("AES/CBC/PKCS5Padding",
    "bqGrDKdQ8zo26HflRsGvVA==",
    new SecretKeySpec(bytes, "AES"));
```

The `decrypt` method reveals the algorithm and IV:

```java
public final String decrypt(String algorithm, String cipherText,
    SecretKeySpec key) {
    Cipher cipher = Cipher.getInstance(algorithm);
    byte[] bytes = Activity2Kt.fixedIV.getBytes(Charsets.UTF_8);
    IvParameterSpec ivSpec = new IvParameterSpec(bytes);
    cipher.init(2, key, ivSpec);
    byte[] decodedCipherText = Base64.decode(cipherText, 0);
    byte[] decrypted = cipher.doFinal(decodedCipherText);
    return new String(decrypted, Charsets.UTF_8);
}
```

{{< figure align="center" src="/images/Strings_3.png" caption="The decrypt() and cd() helper methods, decompiled in jadx" >}}

```java
public final class Activity2Kt {
    private static String cu_d = null;
    public static final String fixedIV = "1234567890123456";
}
```

{{< figure align="center" src="/images/Strings_4.png" caption="Activity2Kt.kt — the hardcoded fixed IV used by decrypt()" >}}

All cryptographic parameters are hardcoded:

| Parameter | Value |
|---|---|
| **Algorithm** | AES/CBC/PKCS5Padding |
| **Secret Key** | `your_secret_key_1234567890123456` |
| **IV** | `1234567890123456` |
| **Ciphertext (Base64)** | `bqGrDKdQ8zo26HflRsGvVA==` |

Plugging these into CyberChef (From Base64 → AES Decrypt) confirms the plaintext is **`mhl_secret_1337`**:

{{< figure align="center" src="/images/Strings_5.png" caption="CyberChef — decrypting the hardcoded ciphertext with the recovered key and IV yields mhl_secret_1337" >}}

The deep link needs this value Base64-*encoded* (since `Activity2` does the opposite operation on whatever path segment it receives), so we add a `To Base64` step to the same recipe:

{{< figure align="center" src="/images/Strings_6.png" caption="CyberChef — re-encoding the decrypted secret to Base64 produces the payload for the deep link: bWhsX3NlY3JldF8xMzM3" >}}

Which gives us the payload we need to send:

```
bWhsX3NlY3JldF8xMzM3
```

---

## Exploitation

### Step 1: Write the Frida Hook

We need to satisfy Gates 1 and 2 before triggering the deep link. Gate 2 requires `KLOW()` to have been called on `MainActivity`. We use Frida to invoke it at runtime:

```javascript
Java.perform(function () {

  setTimeout(function () {
    Java.choose("com.mobilehackinglab.challenge.MainActivity", {
      onMatch: function(instance) {
        console.log("Found instance: " + instance);
        console.log("call KLOW func: " + instance.KLOW());
      },
      onComplete: function() {}
    });
  }, 1000);

  setTimeout(function () {
    Java.choose("com.mobilehackinglab.challenge.Activity2", {
      onMatch: function(instance) {
        console.log("Found instance: " + instance);
        console.log("cd func: " + instance.cd());
        console.log("native func: " + instance.getflag());
      },
      onComplete: function() {}
    });
  }, 10000);

});
```

The script does two things:

1. **At 1 second** — finds `MainActivity` and calls `KLOW()` to persist today's date into `UUU0133`. Since `KLOW()` is declared `void`, Frida logs its return value as `undefined` — that's expected, not a failure.
2. **At 10 seconds** — finds `Activity2` and calls `cd()` and `getflag()` repeatedly (once per `Activity2` instance Frida discovers) to confirm state

### Step 2: Launch the App and Attach Frida

Start the Frida server on the device and confirm the device is visible:

```bash
$ adb shell "/data/local/tmp/frida-server &"
$ adb devices
```

{{< figure align="center" src="/images/Strings_8.png" caption="adb devices — confirming the emulator is up and reachable" >}}

Launch the app:

```bash
$ adb shell am start -n com.mobilehackinglab.challenge/.MainActivity
```

{{< figure align="center" src="/images/Strings_9.png" caption="The Strings app running on the emulator before instrumentation" >}}

Get the process PID and attach Frida:

```bash
$ PID=$(adb shell pidof com.mobilehackinglab.challenge)
echo "PID: $PID"

$ frida -D emulator-5554 -p $PID -l f.js
```

{{< figure align="center" src="/images/Strings_10.png" caption="Frida attaching to the running process and invoking KLOW() on MainActivity" >}}

### Step 3: Trigger the Deep Link

With the date preference set, Frida continues polling `Activity2` — `cd()` returns today's date and the native `getflag()` call succeeds — while we fire the deep link intent in a separate shell:

```bash
$ adb shell am start -W -a android.intent.action.VIEW -d "mhl://labs/bWhsX3NlY3JldF8xMzM3" \
    -n com.mobilehackinglab.challenge/.Activity2
```

{{< figure align="center" src="/images/Strings_11.png" caption="Frida polling Activity2 (cd() and getflag() succeeding) while the mhl://labs deep link intent is sent via adb — Status: ok" >}}

Breaking down the intent:

| Component | Value | Purpose |
|---|---|---|
| `-a` | `android.intent.action.VIEW` | Satisfies Gate 1 (action check) |
| `-d` | `mhl://labs/bWhsX3NlY3JldF8xMzM3` | Matches scheme, host, and carries the Base64 payload |
| `-n` | `com.mobilehackinglab.challenge/.Activity2` | Explicitly targets the exported activity |

The Base64 segment `bWhsX3NlY3JldF8xMzM3` decodes to `mhl_secret_1337`, matching the AES decrypted value — Gate 3 passed.

### Step 4: All Gates Pass — But the Toast Isn't the Flag

All three gates pass, `System.loadLibrary("flag")` runs, and a Toast appears on screen:

{{< figure align="center" src="/images/Strings_12.png" caption="A 'Success' Toast confirms all three gates passed — note this is a status string, not the MHL{...} flag itself" >}}

This is worth calling out explicitly: both the Frida log (`native func: Success`) and the on-device Toast only ever show the literal string **"Success"**. The native `getflag()` method returns a status indicator here, not the actual flag text — so the deep link bypass confirms the vulnerability logically, but doesn't hand us the flag string directly. To get the real flag, we need to look at what's sitting in the process's memory once `libflag.so` is loaded.

### Step 5: Extracting the Flag from Process Memory

Since the flag is loaded into memory by the native library at some point during execution, we can recover it with a full process memory dump using `fridump3`:

```bash
$ fridump3 -u -s strings
```

{{< figure align="center" src="/images/Strings_13.png" caption="fridump3 dumping the app's process memory and running strings against every dumped page" >}}

Then grep the dumped files for the flag pattern:

```bash
$ strings --print-file-name * | grep --with-filename MHL
$ strings * | grep -i "MHL{"
```

{{< figure align="center" src="/images/Strings_14.png" caption="The flag, MHL{IN_THE_MEMORY}, surfaces twice in the dumped memory pages — confirming it was resident in plaintext at runtime" >}}

```
MHL{IN_THE_MEMORY}
```

---

## Why This Works

The core issue is that all security controls are implemented client-side with hardcoded, recoverable secrets, and the flag itself sits in plaintext in process memory at runtime:

- The AES key `your_secret_key_1234567890123456` and IV `1234567890123456` are embedded in the decompiled source — anyone with `jadx` or `apktool` can extract them
- The SharedPreference check (Gate 2) is trivially bypassed via Frida by calling `KLOW()` directly — no user interaction required
- Even though the app never *displays* the actual flag (the Toast only shows a generic "Success" string), the flag value is loaded into the process's memory by `libflag.so` and is recoverable with a basic `fridump3` + `strings` memory dump, no decryption required

A properly hardened Android app should:

- Never hardcode cryptographic keys in client-side code — they will always be recoverable
- Validate deep link inputs server-side or via cryptographic signatures, not local string comparisons
- Avoid exporting activities unless absolutely necessary, and enforce custom permissions when doing so
- Avoid keeping sensitive plaintext values resident in memory for longer than necessary, and clear or obfuscate them immediately after use

## Remediation

| Issue | Fix |
|---|---|
| AES key and IV hardcoded in source | Store secrets server-side; never embed them in the APK |
| SharedPreference written by one activity, read by another | Use scoped storage or enforce signature-level permissions |
| Exported activity with no permission requirement | Set `android:exported="false"` or require a custom permission |
| Flag value resident in plaintext in process memory | Encrypt or obfuscate sensitive values in memory and clear them promptly; validate server-side |
| No integrity check on deep link parameters | Sign the deep link payload and verify the signature in the backend |

---

{{< callout tip "The takeaway">}}

Client-side security controls in Android applications are speed bumps, not walls. Hardcoded secrets, exported activities, and plaintext values sitting in process memory can all be recovered with basic reverse engineering and Frida — even when the app is careful enough not to print the secret on screen. The only reliable defense is to keep sensitive operations and secrets on a backend you control — treat the mobile client as an untrusted consumer of your API.


{{< /callout>}}