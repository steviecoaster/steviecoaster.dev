---
layout: post
Title: Demystifying Pipeline Build Scripts
date: 2019-08-31
categories: [powershell]
tags: [powershell,devops,pipeline,azdos,azuredevops]
comments: true
---

There are a myriad of ways to leverage build scripts inside of Azure DevOps Pipelines, and all the other popular CI/CD providers. In fact, there are whole modules dedicated to it including InvokeBuild,PSDepend,PSake and others.

I hate all of them. Not necessarily because they are bad, quite the opposite. I think they are each quite good and very well written. However, when I build out my pipelines I like to keep things as simple as possible. This helps me to reduce the complexity of the pipeline and make troubleshooting things much much easier. In this blog post I'm going to provide a copy of the build script that I'm currently using for my [PSChocoConfig](https://github.com/steviecoaster/PSChocoConfig) module in Azure Pipelines. I'll give the whole script to you up front, but then we'll break it down in sections as we walk through it.

## The code

Here's the full script:

```powershell

[cmdletBinding()]
Param(
    [Parameter()]
    [Switch]
    $Test,

    [Parameter()]
    [Switch]
    $Build,

    [Parameter()]
    [Switch]
    $Deploy
)

#Make some variables, shall we?
$innvocationPath = "$(Split-Path -Parent $MyInvocation.MyCommand.Definition)"
$PSModuleRoot = Split-Path -Parent $innvocationPath
$TestPath = Join-Path $PSModuleRoot "Tests"

#Do Stuff based on passed Args
Switch($true){

    $Test {

        If(-not (Get-Module Pester)){
            Install-Module -Name Pester -SkipPublisherCheck -Force
        }

        Invoke-Pester -Script $TestPath -OutputFile "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig.Results.xml" -OutputFormat 'NUnitXml'

        #
        Get-ChildItem $env:Build_ArtifactStagingDirectory
    }

    $Build {

        If(Test-Path "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig"){
            Remove-Item "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig" -Recurse -Force
        }

        $null = New-Item "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig" -ItemType Directory

        Get-ChildItem $PSModuleRoot\Public\*.ps1 | Foreach-Object {

            Get-Content $_.FullName | Add-Content "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig\PSChocoConfig.psm1"
        }

        Copy-Item "$PSModuleRoot\PSChocoConfig.psd1" "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig"

        #Verification of contents
        Get-ChildItem -Path "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig" -Recurse

        #Verify we can load the module and see cmdlets
        Import-Module "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig\PSChocoConfig.psd1"
        Get-Command -Module PSChocoConfig

    }

    $Deploy {


        Try {

            $deployCommands = @{
                Path = (Resolve-Path -Path "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig")
                NuGetApiKey = $env:NuGetApiKey
                ErrorAction = 'Stop'
            }

            Publish-Module @deployCommands

        }

        Catch {

            throw $_

        }

    }

    default {

        echo "Please Provide one of the following switches: -Test, -Build, -Deploy"
    }

}

```

## Breaking things down

### Part 1: Testing

Let's have a look at the first section of the script.

```powershell

$Test {

        If(-not (Get-Module Pester)){
            Install-Module -Name Pester -SkipPublisherCheck -Force
        }

        Invoke-Pester -Script $TestPath -OutputFile "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig.Results.xml" -OutputFormat 'NUnitXml'

        #
        Get-ChildItem $env:Build_ArtifactStagingDirectory
    }

```

Inside of Azure Pipelines I have a build step which I've called 'Run Pester Tests'. It calls the build.ps1 file from the Build directory in my repository, and I pass in the `-Test` argument to the script. Because "Test" has been passed in, the Switch statement evaluates to "True" for that switch, and thus the test code is executed.

The If statement simply puts the latest version of the Pester module onto the build agent if it is not there. If you're running this pipeline on a hosted agent, where you can control what modules exist on the agent box, this step will just be ignored as the test for the module will pass and it will happily move along to actually running the tests.

After we have Pester installed we invoke all of the `*.test.ps1` files that are located in the Tests directory of the repository. The `-Script` parameter of `Invoke-Pester` accepts an array of paths, and will execute all the test files that it comes across. 

You'll also notice that I'm passing in an `-OutputFile` and `-OutputFormat` parameter, specifying an xml file name and that they are of type `NUnitXml`. I do this, as I the next step after running the tests is to publish those tests to the pipeline. This gives you a very nice chart-style view of your test results after the pipeline executes, and managers _love_ eye candy, am I right?!

The last line can be ignored, it's simply there to verify that the xml file that I publish test results to is stored as I expect it to be.

Here's the YAML code for that pipeline step in Azure:

```yaml
steps:
- task: PowerShell@2
  displayName: 'Run Pester Tests'
  inputs:
    targetType: filePath
    filePath: ./Build/build.ps1
    arguments: '-Test'
```

### Part 2 : Building the module

After I have ran the Pester tests and published their results I build the module. I develop with all of the functions split up into their own individual ps1 files in a Public folder, and keep a copy of a psm1 that dot sources everything in that folder when I load the psd1. This is so I can test as I develop quickly, but not ultimately how the module should behave when publishing for public consumption.

The build script pulls the contents of each of those ps1 files and, using `Add-Content`, are written to a fresh copy of a PSChocoConfig.psm1 file. This I then just use `Copy-Item` to lift and shift my psd1 file over to the PSChocoConfig directory that I create in the pipeline's ArtifactStagingDirectory. 

Here's what that code looks like:

```powershell
$Build {

        If(Test-Path "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig"){
            Remove-Item "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig" -Recurse -Force
        }

        $null = New-Item "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig" -ItemType Directory

        Get-ChildItem $PSModuleRoot\Public\*.ps1 | Foreach-Object {

            Get-Content $_.FullName | Add-Content "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig\PSChocoConfig.psm1"
        }

        Copy-Item "$PSModuleRoot\PSChocoConfig.psd1" "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig"

        #Verification of contents
        Get-ChildItem -Path "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig" -Recurse

        #Verify we can load the module and see cmdlets
        Import-Module "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig\PSChocoConfig.psd1"
        Get-Command -Module PSChocoConfig

    }
```

Again, the last three lines of this section are simply verification that shows up in the output of the pipeline logs so I can trust that things worked correctly.

And here is what that YAML looks like:

```yaml
steps:
- task: PowerShell@2
  displayName: 'Build Module'
  inputs:
    targetType: filePath
    filePath: ./Build/build.ps1
    arguments: '-Build'
```

### Part 3 : Publishing to PSGallery

The final step in the pipeline is the publish the latest version of the module to the PowerShell Gallery. I do all the work to prep for release on the repository side, like making sure I've bumped the version inside the `psd1` file. If I forget, the step will fail, which will fail the build, so I'll _know_ right away what I did.

Here's what that deploy code looks like:

```powershell
$Deploy {


        Try {

            $deployCommands = @{
                Path = (Resolve-Path -Path "$($env:Build_ArtifactStagingDirectory)\PSChocoConfig")
                NuGetApiKey = $env:NuGetApiKey
                ErrorAction = 'Stop'
            }

            Publish-Module @deployCommands

        }

        Catch {

            throw $_

        }

    }
 ```

You'll notice the `$env` variable there. I've defined that as a secret variable in the Pipeline Variables section of the build pipeline. I reference it into the script using the $(NuGetApiKey) variable in the YAML.

And here's that YAML for the build step:

```yaml
steps:
- task: PowerShell@2
  displayName: 'Deploy to PSGallery'
  inputs:
    targetType: filePath
    filePath: ./Build/build.ps1
    arguments: '-Deploy'
  env:
    NugetApiKey: $(NuGetApiKey)
```

## Wrapping Up

I hope you've found my approach to pipelines useful. Sometimes keeping it simple is the best way to approach things, and this method works quite well. If you enjoyed this article, or have any feedback, please feel free to leave me a message here or drop me a line on Twitter @steviecoaster. Thanks for reading! Until next time...