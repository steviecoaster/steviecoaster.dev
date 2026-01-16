---
title: Sweet Tip! Automatic PATH refresh with Chocolatey
date: 2026-01-15
authors:
  - Stephen: author.jpeg
---

One of Chocolatey's little hidden gems is the `refreshenv` command. When run, this handy helper refreshes your PATH — useful after
installing software that adds a command-line tool (git, PuTTY, dotnet, etc.).

One of the biggest pain points when installing a new tool is that you have to close and reopen your shell before it can "see" the
newly installed tool. Such annoyance. Much bullshit. What do we do?

Enter... `refreshenv`

TL;DR: Run `refreshenv` after installing a package to refresh your current shell session's PATH.

## Using refreshenv

It's simple: after installing a package, run `refreshenv`. Done. However, to use it from PowerShell you need to import the Chocolatey
PowerShell profile so the function (and tab-completions) are available.

In Command Prompt (`cmd.exe`) it just works. In PowerShell, install the profile and import it in your profile so `refreshenv` is present.

If you don't yet have a PowerShell profile, create one first. Note there are several profile files and scopes (CurrentUser vs AllUsers, AllHosts vs CurrentHost) — see the Microsoft docs on PowerShell profiles for details and options:

[About PowerShell profiles](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_profiles/)

```powershell
if (-not (Test-Path $profile)) { New-Item -ItemType File -Path $profile -Force }
```

The snippet above uses the `$profile` automatic variable (your CurrentUserCurrentHost profile). If you want a profile that applies to all hosts or all users, check the docs for the appropriate path.

Then add the following to your profile:

```powershell
# Import the Chocolatey Profile that contains the necessary code to enable
# tab-completions to function for `choco`.
# Be aware that if you are missing these lines from your profile, tab completion
# for `choco` will not function.
# See https://ch0.co/tab-completion for details.
$ChocolateyProfile = "$env:ChocolateyInstall\helpers\chocolateyProfile.psm1"
if (Test-Path($ChocolateyProfile)) {
  Import-Module "$ChocolateyProfile"
}
```

## How it works (under the hood)

Short version: `refreshenv` triggers the same logic used by Chocolatey's _Update-SessionEnvironment_ helper. When a package installer modifies the PATH it typically does so by updating the registry (the machine scope at `HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment` or the user scope at `HKCU:\Environment`). Those registry changes are persistent, but they are not automatically applied to already-running processes.

`Update-SessionEnvironment` reads the updated registry values, merges them according to Windows rules, and updates the current process environment (the shell you have open). In practice that means the running shell gets a new `PATH` immediately and can invoke the newly installed tools without a restart.

Here is a tiny, simplified pseudo-implementation to give you the idea (not the exact code used by Chocolatey):

```powershell
# simplified example
$machinePath = (Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment' -Name Path).Path
$userPath = (Get-ItemProperty 'HKCU:\Environment' -Name Path -ErrorAction SilentlyContinue).Path
$newPath = [string]::Join(';', @($machinePath, $userPath) | Where-Object { $_ -and $_ -ne '' })
[System.Environment]::SetEnvironmentVariable('Path', $newPath, 'Process')
# optional: broadcast a WM_SETTINGCHANGE/Environment notification for GUI apps
```

Why this matters:

- `refreshenv` updates _your current shell session_ so subsequent commands in the same session can use newly-installed tools.
- Other shells or GUI apps started before the change won't see the update — they either need to be restarted or listen for the environment change notification.

Troubleshooting tips:

- If `refreshenv` does nothing, confirm your profile imported the Chocolatey profile and that `refreshenv`/`Update-SessionEnvironment` are present.
- Inspect the registry PATH with `Get-ItemProperty` to ensure the installer actually wrote the new path.
- Check your current session's PATH in PowerShell with `Get-ChildItem Env:Path` to see what the shell sees right now.


## A note for package maintainers

If you maintain Chocolatey packages — for the community, your org, or just yourself — here's a tiny tip:

_Put `refreshenv` as the last line of your install script. If you're a purist, run [Update-SessionEnvironment](https://docs.chocolatey.org/en-us/create/cmdlets/update-sessionenvironment/) instead — same effect._

## A super fun cheat code

We've covered manual usage and per-package automation. But if you want PATH refresh to happen after _every_ package install, look at
Chocolatey hooks (available since v2.0.0). Hooks can run on install, upgrade, or uninstall and may be package-specific or global.

See the [Chocolatey hook package guide](https://docs.chocolatey.org/en-us/guides/create/create-hook-package/#mainContent) for details, but the gist is:

1. Run `choco new refreshenv.hook`
1. Replace the generated `tools` folder with a `hook` directory
1. Add a `post-install-all.ps1` file containing:

```powershell
# post-install-all.ps1
Update-SessionEnvironment  # yes, I'm one of those purists
```

1. Fill out the nuspec appropriately
1. Pack and install the hook package

Once installed, the hook will run after package installs and refresh your PATH automatically — no more shell restarts or cursing Windows.

Until next time!
