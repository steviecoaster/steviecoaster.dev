---
title: The Many Ways Of Native Commands In Pwsh
date: 2024-11-22
authors:
  - Stephen: author.jpeg
---

Working with native commands in PowerShell can sometimes be tricky business, depending on your use case.
Thankfully, there are several different ways and tools available to make working with native commands in
your scripts easier. In this post we'll highlight the available methods to mixing traditional commands
with PowerShell

Grab a bourbon, and let's get to it!

## The --% Operator

In PowerShell when running a native command from pwsh.exe, you can use the stop parsing operator to tell PowerShell to pass the arguments entered for the native command with interpretting them. NOte that this works best when you're just doing something quickly via powershell.exe or pwsh.exe, not in a scripting scenario.

For example, we can tell PowerShell to interpret the arguments passed to ping exactly as written like so:

```powershell
ping --% -n 4 127.0.0.1
```

## Using Start-Process

`Start-Process` is probably the _most_ common example of running a native command you've seen, and for good reason. It is an excellent way to run a native command with some arguments and receive the output for later processing (if any). 

Windows PowerShell is a _little_ more limited in what Start-Process can do, but is still quite powerful in that you can provide Verbs such as RunAs, Credentials to execute the process as a different user, or Redirect the input, output, and error streams.

In PowerShell 7+ you have the ability to pass in an Environment hashtable that will provide environment variable data to the process.

To save reinventing the wheel, I encourage you to check out the [docs](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/start-process?view=powershell-7.4) for Start-Process to find some great examples on the flexibility to run native commands it provides

## The Call Operator

The call operator, or &, is my go-to method for interacting with native commands in PowerShell the _majority_ of the time. Particulary because my usecases _rarely_ require me to worry about redirecting output streams or error handling more complex than a simple try/catch.

The call operator allows you to define the arguments you wish to pass to a native command as an array, and then pass that array to a native command. This helps us to to make sure we are handling string interpolation correctly by dealing with that in the array, so it is unrolled properly when executed with the native command.

Let's use a Chocolatey command to demonstrate:

```powershell
$version = '1.2.3'
$chocolateyArgs = @('install','putty',"--version='$version'")

& choco @chocolateyArgs
```

When PowerShell calls choco the command will run the expected commandline `choco install putty --version='1.2.3'` because we have properly interpolated the variable inside the array.

By wrapping the entire version string in double quotes, and the variable in singletons, we allow PowerShell to unroll the variable into its value, while wrapping that value in quotes... which you may find is required by many native commands.

You'll also notice that using the call operator allows us to splat the array to the native command much like you an splat a hashtable of parameters and their values to a PowerShell cmdlet.

Now, there is one thing that I do like to point out and do differently that my example above. I like to use the full path to the native command. This ensures that my script will work on systems where the tool may be installed, but is not on the $PATH.

Here's the same example above, improved to handle this use case:

```powershell
$version = '1.2.3'
$chocolateyArgs = @('install','putty',"--version='$version'")

$choco = (Get-Command choco).Source

if(-not $choco) {
    throw 'Choco is required for this command to run, but was not found on the system'
} else {
    & choco @chocolateyArgs
}
```

In this enhanced example we lookup the full path to choco.exe using the `Get-Command` cmdlet, and throw an error if we don't find it.

If you need to capture the output of  native command, just use a variable, e.g. `$output = & $choco @chocolateyArgs`. You'll then be able to parse that output later to do whatever you need to do with it.

## The Microsoft.PowerShell.Crescendo Module

Saving the best for last, Crescendo is my _favorite_ tool for interacting with native commands in PowerShell. This module allows you to generate PowerShell cmdlets with parameters for your native commands. In our ping example above, Crescendo will allow you to very quickly create an `Invoke-Ping` cmdlet with parameters such as `-Packets` and `-Target` turning `ping -n 4 127.0.0.1` into `Invoke-Ping -Target '127.0.0.1' -Packets 4`.

There's more to Crescendo than this blog post can provide, and perhaps it should be its own series. However, I've spoken about it extensively at conferences, and there are some great write-ups already, so I'll just provide you links to those resources below

Sean Wheeler wrote a fantasic series on the [PowerShell Community Dev Blog](https://devblogs.microsoft.com/powershell-community/my-crescendo-journey/)

I spoke at the [Research Triangle PowerShell UG](https://www.youtube.com/watch?v=I215tOTeHhY&list=PLCN8aP0eYmK9CXOtyNb0wZ7fSnyS1W2To&index=9&pp=iAQB) as well as [PowerShell Summit](https://www.youtube.com/watch?v=JBFhptwwYuU&list=PLCN8aP0eYmK9CXOtyNb0wZ7fSnyS1W2To&index=7&pp=iAQB) about it. Though, admittedly the RTPSUG session is a bit better, as the Summit session was a last-minute addition for me, so the examples are a little "thin"

## Wrap-Up

Well, there you have it. In this post we've looked at several methods to invoke native commands in your PowerShell scripts. I hope you've found it helpful. If you have any questions be sure to reach out to me on Bluesky [here](https://bsky.app/profile/steviecoaster.bsky.social) or at any of the contact methods available here on my site. Happy scripting!
