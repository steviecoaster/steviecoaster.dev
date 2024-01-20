---
Title: Write good help...it's important!
date: 2018-11-23
categories: [powershell]
tags: [powershell,cbh,best practices,help]
---

There's nothing more frustrating that running Get-Help against a cmdlet, and not getting a lot of useful information back. The Powershell Help System is a robust tool that makes your scripts extremely discoverable to its users.

A good Comment Based Help block, in my opinion, contains the following useful information:

1. A synopsis (typically a brief statement) and description (typically a longer explanation of the script functionality), which tells the end user what the script _does_.
2. An explanation for each parameter your script uses.
3. Solid examples of your script's functionality.

I also like to include a HelpUri in the param block, which links out to a MarkDown Formatted document in the project's repository. Using markdown help files also allows you to do things like add GIF examples of your scripts in action, which is really neat! More on how to add this to your script is below.

## Adding help to your Script

Comment based help (CBH as it will herein be referred as) should be added right under the opening `function Your-Function {` call, and is inserted as a multi-line comment.

```powershell
Function My-Function{
    <#
        This is a multiline comment.
        And where CBH should be described.
    #>
}
```

Each section of CBH is denoted with a leading `.` before the keyword. The list of Keywords available are listed in the Powershell documentation [here](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_comment_based_help?view=powershell-6#comment-based-help-keywords).

A full example of a CBH block would look something like this:

```powershell
Function New-FancyFunction{
<#
    .SYNOPSIS
    This function demonstrates Comment Based Help

    .DESCRIPTION
    When you run Get-Help New-FancyFunction you will see all the parameters and other information available to you when you use the Function.

    .PARAMETER SomeParameter
    A brief explanation of what the parameter is and it's use in the function. Add a declaration for each Parameter your script uses.

    .EXAMPLE 
    Examples show how your script can be used, typically in its most basic form, and then a few other examples which use more parameters, or how to leverage it with the pipeline.

    .NOTES
    Any additonal information you wish to add about the function can be added in the Notes field.

#>

}
```

Adding a link to external help, hosted in a Github Repo for your project for example, is incredibly simple to do. Here's an example:

```powershell
Function My-Function {
[cmdletBinding(HelpUri="https://github.com/yourusername/repo/help.md")]
Param(
    #insert your parameters here
)

}
```

That's it! Just add the HelpUri attribute inside `[CmdletBinding()]` and when a user of your script uses `Get-Help _YourFunction_ -Online` it will open their browser to the Uri you specified.

That's all there is to adding really _helpful_ help to your scripts. Please use it. Others who run your code with thank you.

Until next time!
_Stephen_