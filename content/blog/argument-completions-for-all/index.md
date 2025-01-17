---
title: Add  argument completions to your PowerShell parameters
date: 2025-01-16
authors:
  - Stephen: author.jpeg
---

There are many ways to perform argument (parameter) completions in PowerShell. The three we'll look at are:

1. ValidateSet
2. Register-ArgumentCompleter
3. [ArgumentCompleter()]

Each of these offer their own sets of pros and cons, and having a solid understanding of each will not doubt help you to
make an informed decision when you're sitting down to tackle your next challenge with the help of PowerShell.

Let's dive in, shall we?

## ValidateSet

Our first argument completion mechanism is the [ValidateSet()] attribute. Typically you would find this added to a parameter
declaration when you are writing an advanced function. That will usually look something like this:

```powershell
[Parameter()]
[ValidateSet('dog','cat','bird')]
[String]
$Animal
```

A ValidateSet is an array of items you want to be valid input for a particular parameter, and anything _other_ than what
you place inside the set will cause an exception to be thrown. This kind of "fail early" approach will help the user to correct
a mistake before any "real" damage happens. How many of us have had a typo in our input and spent hours tracking it down as
the code ran fine?

The PowerShell documentation has more information that you can read [here](https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/validateset-attribute-declaration?view=powershell-7.4.)

## Register-ArgumentCompleter

The second completion mechanism we will discuss is the _Register-ArgumentCompleter_ cmdlet. This cmdlet allows you to 
register completion options for a particular parameter of a specific PowerShell function _or_ native command executable.

For example, say you want to use _Get-ChildItem_ inside of a function that is a part of your module. You may wish to limit
what paths are allowed to be provided to that cmdlet to enumerate its contents. You can register an argument completer to
limit the paths allowed with the following:

```powershell
Register-ArgumentCompleter -CommandName Get-ChildItem -ParameterName Path -ScriptBlock { @('D:\Data','C:\Temp','C:\installation_media') | % { $_ }}
```

⚠️WARNING!

_When you register a completer for a common cmdlet, such as Get-ChildItem that completion is valid for ALL scopes and contexts!_
_The above is just an example for demonstration purposes, please don't do that in real life._

These argument completers are fairly easy to make and the [documentation](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/register-argumentcompleter?view=powershell-7.4) has a lot of great examples. It's important to note that these completers
only exist in the current session, so if you're working within a module you may wish to add them as part of your .psm1
file such that they are available when the module loads. Or if you have command tasks you do in the shell, add them to your
profile!

## [ArgumentCompleter()]

The [ArgumentCompleter()] attribute is certainly the most complex method to use when trying to add completions to your 
parameters, but it does offer the most flexibility. Let's look at how you would define one for a parameter, and then we can
break it down. For this I'll use a real example, from one of my modules, NexuShell:

```powershell
[Parameter(Mandatory,ValueFromPipeline,ValueFromPipelineByPropertyName)]
[Alias('Name')]
[ArgumentCompleter( {
    param($CommandName, $ParameterName, $WordToComplete, $CommandAst, $FakeBoundParameters)
    $repositories = (Get-NexusRepository).Name

    if ($WordToComplete) {
        $repositories.Where{ $_ -match "^$WordToComplete" }
    }
    else {
        $repositories
    }
})]
[String[]]
$Repository
```

To help explain things I'll borrow some info from the [documentation](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_functions_argument_completion?view=powershell-7.4#argumentcompletions-attribute).

The script block parameters are set to the following values:

$commandName (Position 0) - This parameter is set to the name of the command for which the script block is providing tab completion.
$parameterName (Position 1) - This parameter is set to the parameter whose value requires tab completion.
$wordToComplete (Position 2) - This parameter is set to value the user has provided before they pressed _Tab_. Your script
block should use this value to determine tab completion values.
$commandAst (Position 3) - This parameter is set to the Abstract Syntax Tree (AST) for the current input line.
$fakeBoundParameters (Position 4) - This parameter is set to a hashtable containing the $PSBoundParameters for the cmdlet,
before the user pressed _Tab_.

In my example above I'm fetching all of the available repositories on a server, and only storing their name property. I
then use a Where _method_ to unroll the array based on a regular expression. This allows me to start typing a repostiory
name, press tab, and have my completions start where the first match is, _very_ helpful!

It's important to note that you can run any code you want inside the scriptblock to gather your completion results. These
are _incredibly_ flexible and powerful and I encourage you to read the documentation and play around with them. You will
most certainly find a use for them.

## Final thoughts

I love putting guardrails in my code to help shield the end user from themselves. Argument completions are just one arrow
in PowerShell's quiver of tricks to help you increase the user experience of the tools you create. Use 'em!

Happy Scripting!