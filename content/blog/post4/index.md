---
layout: post
title: Being Cheeky With Speech
date: 2018-11-20
categories: [powershell]
tags: [powershell,justforfun]
comments: true
---

# Goofing off in the shell

This was born of a trying to be clever and funny in the Powershell Slack channel last night. It ended up working better than expected. Want to make a coworker's cheeks turn red? Wanna make your significant other roll their eyes at you? Don't worry, I've got you covered.

Save the following to a ps1 file, or just copy and paste it into a console, and enjoy your console telling you random pickup lines from the internet. And yes folks, apparently there really _is_ an API for everything...

```powershell
Function Invoke-PickupLine {

    Add-Type -AssemblyName System.speech
    $speak = New-Object System.Speech.Synthesis.SpeechSynthesizer
    $line = (Invoke-RestMethod http://pebble-pickup.herokuapp.com/tweets/random).tweet
    $speak.Speak($line)

}
```

Wanna execute it on a remote workstation? Load this into memory so it shows up in your Function Drive

```powershell
PS C:\> gci Function:\Invoke-Pickupline

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        Invoke-PickupLine
```

Once you know it's loaded in your function drive, you can access it's scriptblock like this:
\* _this only works for non-compiled Script cmdlets, without a lot more work. It's theoretically possible, but kinda ugly_

```powershell
PS C:\> ${Function:\Invoke-PickupLine}

    Add-Type -AssemblyName System.speech
    $speak = New-Object System.Speech.Synthesis.SpeechSynthesizer
    $line = (Invoke-RestMethod http://pebble-pickup.herokuapp.com/tweets/random).tweet
    $speak.Speak($line)

```

That makes sending the function across the network to a remote computer pretty easy! Just execute your command like this:

```powershell
Invoke-Command -Computername remotecomputer -Scriptblock ${Function:\Invoke-PickupLine}
```

If you'd like to know more about that Function:\ drive business, Joel Sallow has a real nice write-up on his blog over [here](https://vexx32.github.io/2018/11/02/Transferring-Functions/) that I would encourage you to read.

Happy trolling! Until next time...