---
title: Extending AutomatedLab with AutomatedLab.Utils and PowerShell
date: 2025-07-10
authors:
  - Stephen: author.jpeg
---

For the past several months (since my switch from MacOS back to Windows at work), I've been using [AutomatedLab](https://automatedlab.org) and Hyper-V to build ephemeral VMs to test various scenarios as part of my work.

## What is AutomatedLab?

AutomatedLab is a PowerShell module that lets you define and deploy complex virtual machine environments using code. Think of it as "Infrastructure as Code" for your local development and testing needs. Instead of manually clicking through VM setup wizards, you write a PowerShell script that describes what you want, and AutomatedLab builds it for you.

If you're unfamiliar with AutomatedLab and want a fantastic jump start, I highly recommend reading my friend [Jake Hildreth's](https://bsky.app/profile/dotdot.horse) article on his blog titled [A Dive into AutomatedLab - Infrastructure as Code for PowerShellers](https://jakehildreth.github.io/blog/2025/07/04/A-Dive-into-AutomatedLab-Infrastructure-as-Code-for-PowerShellers.html)

## Getting Started

Before diving into the advanced techniques, here's what you'll need:

**Prerequisites:**

- Windows 10/11 Pro or Windows Server (for Hyper-V)
- Administrative privileges
- At least 8GB RAM (16GB+ recommended)
- PowerShell 5.1 or later

**Quick setup:**

1. Enable Hyper-V: `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All`
2. Install AutomatedLab: `choco install automatedlab -y -s https://community.chocolatey.org/api/v2`

Once you have these basics in place, you can start building your first lab!

## Extending Labs w/ PowerShell

Every lab in AutomatedLab starts with a definition. This definition is _just_ a PowerShell script. Given that it is just a script, we can employ ALL of the tricks we know and love about PowerShell when building labs.

Here are two practical techniques I use regularly to make my labs more flexible and maintainable:

### Variable VM sizes

One of the biggest challenges when building VMs is deciding how much resources to allocate. Too little and your tests run slowly, too much and you might overwhelm your host machine.

In the advanced parameter block of _nearly_ all of my lab definitions I provide a parameter to determine the size of the VM, size being the number of CPU cores and amount of RAM I wish for the VM to consume from my host machine. This parameter uses a ValidateSet so I can only choose between Small, Medium, and Large. Here's a sample with _only_ that parameter defined:

```powershell
[CmdletBinding()]
Param(
  [Parameter()]
  [ValidateSet('Small','Medium','Large')]
  [String]
  $VMSize
)
```

Inside the body of my definition script, I use a hashtable to define the resource limits based on the value of the `$VMSize` parameter. Let's add to our example above that hashtable:

```powershell
[CmdletBinding()]
Param(
  [Parameter()]
  [ValidateSet('Small','Medium','Large')]
  [String]
  $VMSize
)

end {
   $resources = @{
        Large  = @{
            Memory     = 16GB
            Processors = 4
        }
        Medium = @{
            Memory     = 8GB
            Processors = 4
        }
        Small  = @{
            Memory     = 2GB
            Processors = 2
        }
    }
}
```

Doing this allows me to reference the hashtable for the values of `Memory` and `Processors` when defining the actual VMs for my lab. This approach gives me flexibility without hardcoding resource values throughout my script.

**Why this matters:** Instead of creating separate lab definitions for different resource requirements, I can use the same script and just change the `$VMSize` parameter. Perfect for when I need to run the same lab on my laptop (Small) versus my workstation (Large).

### Dynamic VM options

Sometimes you need to test against different operating systems or configurations. Rather than maintaining separate lab definitions for each OS, I use a menu system I wrote to dynamically select options.

I won't pollute your screen with the code here, but if things like that interest you all of my lab definitions are available on my GitHub in my [ChocoLabs](https://github.com/steviecoaster/ChocoLabs) repository. For an example of that menu based system mentioned above, check out my [Multi-Client](https://github.com/steviecoaster/ChocoLabs/tree/main/Labs/Licensed/Multi-Client) lab.

## Managing AutomatedLab with AutomatedLab.Utils

When you work with AutomatedLab as much as I do, you quickly realize that keeping your various lab definitions organized is a challenge. Being a PowerShell developer I solved my management mess by writing [AutomatedLab.Utils](https://github.com/steviecoaster/automatedlab.utils).

### What AutomatedLab.Utils provides

- **Lab management**: Organize and categorize multiple lab configurations
- **Easy start/stop**: Quick commands to bring labs up and tear them down
- **Custom roles**: Enhanced support for AutomatedLab's [Custom Roles](https://automatedlab.org/en/latest/Wiki/Advanced/customroles/) feature
- **Streamlined workflow**: Less time managing, more time testing

You can find all the information on getting started managing multiple lab environments with AutomatedLab.Utils on its [docs](https://steviecoaster.github.io/automatedlab.utils/) site.

## Wrap up

AutomatedLab is an absolutely fantastic way to quickly stand up infrastructure to test things at work or at home. When you couple AutomatedLab's capabilities with your own imagination, and a little bit of PowerShell, you can make very complex environments that are repeatable, and shareable, with relative ease!

If you're looking at getting started and need some help or advice don't hesitate to reach out to me on [Bluesky](https://bsky.app/profile/steviecoaster.dev)! Made an aweome lab that you're proud of? Let me know that too, as I may learn a thing or two from you!

Until next time...happy scripting!