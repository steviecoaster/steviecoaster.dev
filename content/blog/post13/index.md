---
title: Stop Installing Software Manually
summary: Getting started with the Chocolatey package manager for all your Windows software needs.
date: 2019-11-16
authors:
  - Stephen Valdinger: author.jpeg
---

Today's post is a little bit biased. I'll be talking about the `choco upgrade` command and how it can save you oodles of time moving forward keeping your system up to date. Why am I biased? Well, that's because I'm currently a Senior Support Engineer for [Chocolatey Software](https://chocolatey.org).

# The Concept

Chocolatey, or 'choco' for short, is a software package management application for Windows. With right around 7000 packages available on the [Community Repository](https://chocolatey.org/packages) the software makes it dead simple to not only get what you need to be productive, but easily maintain those softwares over time. 

# Using it

Getting chocolatey installed is dead simple. Open up an administrative powershell prompt and enter `Set-ExecutionPolicy Bypass -Force -Scope Process; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/Install.ps1'))`. If you want to see what that install script contains before running it, you can view it [here](https://chocolatey.org/Install.ps1).

Once you have chocolatey installed, you will be able to install any packages listed on the community repository. As an example I'll demonstrate installing Google Chrome, and Visual Studio Code. 

You could do it this way:

```powershell
choco install googlechrome -y
choco install vscode -y
```

The `-y` here just skips the confirmations prompts about "Do you really want to run this script to install this software?"

We can shorten that command down to one line though, as choco accepts a space separated list of package names. The following does the same thing as above:

```powershell
choco install googlechrome vscode -y
```

Similar to installs, uninstalls are just as simple. Simply replace `install` with `uninstall`, and Bob's your uncle!

# Updates over time

Now that you are managing Google Chrome and VSCode with chocolatey, it's time to upgrade those packages to their latest version. You can do all currently installed software packages in one fell swoop with `choco upgrade all`. Alternatively, you could supply just the name of a single package to the `upgrade` command to only install an update to that specific application's package.

If you have a situation where you've installed a version of a particular application with chocolatey, and you _must_ maintain that version of that package over time, but wish to upgrade all other packages with `choco upgrade all`, don't worry, we thought of that too. Using the `choco pin` command you are able to maintain a particular package to a specific version, which will be skipped during a `choco upgrade` operation.

# Wrap up

That's it. Short and sweet. Running one line of PowerShell to get choco up and running on your system can save you _hours_ of time keeping software packages up to date manually. I hope you found this information helpful. Chocolatey is Open Source for everyone, with a Business Edition available if you would like to leverage it in your enterprise. The business editon comes with features like:
* Package Builder (_Right Click => Create Chocolatey Package functionality._)
* Package Internalizer (_Bring in a Community Repository including any binaries for use on your own private repositories_) 
* Self Service (_Non-Admin users can install packages from a catalog via a wonderful GUI application similar to SCCM's Software Center_)
* Central Management (_Centralized reporting of all machines with choco installed, detailing installed packages, and their versions, with more features coming soon_)
