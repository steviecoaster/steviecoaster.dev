---
layout: post
title: Building a Package Builder Web Endpoint
date: 2020-09-17
categories: [powershell]
tags: [powershell,chocolatey,packaging]
comments: true
---

# Package Builder - Web Edition

_Preface: This isn't officially supported by Chocolatey. But, I'm nothing if not adventurous. What's the saying? "Where there's a will, there's a way"? Anyway, I digress, onto the good stuff._

Earlier today I had a conversation with myself, as often happens. It went a little something like this.

```text
Self: You haven't done something ridiculous in a while.
Busy-Self: I know, self, because I'm busy!
Self: But _this_ would be so cool!!
Busy-Self: Yes, but, _points at list_
Self: .....
Busy-Self: .....
```

So with Self winning the argument as per usual I set out on the task of building a web front end for the Package Builder feature in Chocolatey For Business, which enables you to take a standalone executable, be that `.exe`,`.msi`, or `.msu` and turn that into a full functioning Chocolatey package in as little as 5 seconds, depending on the size of the installer. Yes, I know, you're going to need a C4B license for this. Sorry.

## Goals

I had the following goals in mind:

- Simple interface
- No complex code (aka regex, cause I friggin hate it)
- RESTful would be nice, but not a must-have

---

## The Solution

In developing the solution I went through a few iterations using [PowerShell Universal](https://ironmansoftware.com/powershell-universal/). If you have not yet checked that out, you really should. It can turn anyone with some PowerShell chops into a WebDev* in just a couple of minutes. I particularly love it for how fast it allows me to build out these hair-brained proof of concept ideas.

_* It takes years to be a good WebDev_

Initially I wanted to do a web form, wherein the end user could fill in some info, click a button, and out pops a Chocolatey package. That ultimately _worked_, but I didn't quite like the user experience. That's way more work than just right-clicking a file after all, and not very extensible.

Then the lightbulb moment hit. PowerShell Universal has an _awesome_ API endpoint system! I friggin love working with RESTful api stuff, and kicked myself for not just going that route in the first place.

---

### Creating the REST Api Endpoint

Once you have PowerShell Universal running (`choco install powershelluniversal -y` for the ~~lazy~~ efficient), open up a web browser and head to `http://localhost:5000`. You'll be asked to login. Use the username `admin`, and any password. This is the default behavior. You can secure it more later if you want, but for the purposes of this blog post and POC nature of the project, we're just gonna YOLO it.

Once logged in, follow these steps:

- Go to the API section on the left-hand navigation. 

![PowerShell Universal Navigation](/images/PBWUI/navbar.png)

- Then you'll want to add an endpoint.

![Add an Endpoint](/images/PBWUI/AddEndpoint.png)

- Configure the endpoint for POST requests

![Configure Endpoint](/images/PBWUI/ConfigureEndpoint.png)

- Edit the endpoint to, you know, _do_ something by clicking 'View Endpoint'

![Edit Endpoint](/images/PBWUI/ViewEndpoint.png)

- Select 'Edit' once on the 'View Endpoint' page, and write the code you want your endpoint to execute

![Make it do something](/images/PBWUI/EditScript.png)

- Save your changes

![Save the script](/images/PBWUI/SaveScript.png)

The code that I'm currently using is conveniently provided below.

---

#### Code: 


```powershell
$package = $headers.Package
$fileName = $headers.File

$tempPath = 'C:\tmp'
$file = "$tempPath\$($fileName)"
[System.IO.File]::WriteAllBytes("$file", $Data)

function New-ChocolateyPackage {
    [cmdletBinding()]
    param(
        [parameter()]
        [string]
        $file,

        [parameter()]
        [string]
        $PackageName,

        [parameter()]
        [string]
        $outputdirectory
    )

    process {
                
        $statements = "new $PackageName --file $file --build-package --output-directory $OutputDirectory"
        $process = New-Object System.Diagnostics.Process
        $process.EnableRaisingEvents = $true

        Register-ObjectEvent -InputObject $process -SourceIdentifier "LogOutput_ChocolateyProc" -EventName OutputDataReceived -Action $writeOutput | Out-Null
        Register-ObjectEvent -InputObject $process -SourceIdentifier "LogErrors_ChocolateyProc" -EventName ErrorDataReceived -Action  $writeError | Out-Null

        $psi = New-Object System.Diagnostics.ProcessStartInfo
        $psi.FileName = 'C:\ProgramData\chocolatey\bin\choco.exe'
        $psi.Arguments = "$statements"

        $process.StartInfo = $psi
        $process.Start()
        $process.WaitForExit()
        $process.Dispose()
    }
}

Start-Sleep 3
New-ChocolateyPackage -File $file -PackageName $package -OutputDirectory C:\processed
Start-Sleep 3

Get-ChildItem 'C:\processed' -Exclude *.nupkg | Remove-Item -Recurse -Force
Remove-Item $file -Force
```

---

### System Setup

If you're going to use the code above _verbatim_, you'll need to ensure a few things. You will need the following directories to exist on the system:

1. `C:\tmp`
2. `C:\processed`

The script will temporarily create a copy of the installer in C:\tmp, which then Chocolatey will pick up the file from there and build the package. It's designed this way to simulate a remote call to the API. The `C:\processed` directory will be the place where the completed Chocolatey packages are stored.

---

### End Result

Once you've done all of that, it's off to run some powershell! To make this work, we are going to need to provide a header. This part I don't _quite_ 100% like, but for the purposes of "Could this even work", it's good enough.

Open up an _elevated_* PowerShell console and run the following, editing it with an actual installer file, and the name you wish to give the package:

```powershell
$header = @{
    File = 'someinstallerfilename.exe'
    Package = 'GiveThePackageAName'
}
```

Next, we just need to craft a call to the API with `Invoke-RestMethod:

```powershell
$irmParams = @{
    Uri = 'http://localhost:5000/packagebuilder'
    Method = 'Post'
    Headers = $header
    InFile = 'InstallerFile'
}

Invoke-RestMethod @irmParams
```

---

### Caveats

This is a proof of concept. Please don't consider it Gospel and run off and use it in Production until you've _really_ vetted the solution. Also, PowerShell Universal requires a license if you are going to use authentication to the REST endpoint. (You should support Adam anyways, he's really good people.)

Because of the nature of `Invoke-RestMethod` large installers aren't going to work here. There may be some additional tweaking and poking to be done to make it work, but as of the time of writing this, anything over 100MB threw a big ol temper tantrum when I tried to use it.