---
Title: Unwrapping the $PSCmdlet variable
date: 2019-03-24
categories: [powershell]
tags: [powershell,variables,advanced functions]
---

I'm sure we've all seen it in code online. We're checking out some code, probably on Github, and we notice folks using this strange `$PScmdlet` thing. When I first started I considered this variable "black magic", and really after getting used to it and what it does I still kinda feel like that's what it is. But let's dive in and unwrap it so we can see how the magic trick actually works.

`$PSCmdlet` is used in advanced functions and is derived from the [System.Management.Automation.PSCmdlet] base class in .Net. In order this parameter "automatically", you'll need to ensure that you have [CmdletBinding()] declared just above your `Param()` block. 

