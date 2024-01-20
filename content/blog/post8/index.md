---
Title: "ELI5: Powershell objects"
date: 2019-06-01
categories: [powershell]
tags: [powershell,objects,learn]
---

Hello everyone! I apologize for the gap in posts here, it's been quite a busy season of life starting a new job at [Chocolatey Software](https://chocolatey.org), and going to several conferences as well as other travel.

In this post we will be taking a look at Powershell objects, and I'll try to explain them in a very clear way that will help you understand them better, and ultimately (hopefully!) make your code endeavors easier, and more enjoyable.

Powershell is an _object oriented_ scripting language. This means that unlike, say, Bash (which only returns _string data_) by and large Powershell will do its best to return an object to you.
A Powershell object contains many things that define what the object _is_, and how you can _use_ it.
Most notably, these are properties, and methods.

Let's take a step away from Powershell for a moment, and I'll explain this using something other than code.
In our example we will use a car.
The _car_ is our object.
We can define how the car looks, and its features as being the _properties_ of the car.
These properties are what are known as _note properties_ when working with objects in Powershell.
Let's describe our car next.

In this example our car will be blue in color, have 4 doors, and will be a Chevy Camaro.
If we were to define our car in Powershell, it would look something like this:

```powershell
$car = New-Object -Typename PSCustomObject
Add-Member -InputObject $car -MemberType NoteProperty -Name Color -Value Blue
Add-Member -InputObject $car -MemberType NoteProperty -Name Doors -Value 4
Add-Member -InputObject $car -MemberType Noteproperty -Name Make -Value Chevrolet
Add-Member -InputObject $car -MemberType Noteproperty -Name Model -Value Camaro
```

There's a faster way, both in terms of code effort, as well as underlying execution speed to define properties of an object using a hashtable with a trick called "casting".
Here is how that would look:

```powershell
$car = [pscustomobject]@{
    'Color' = 'Blue'
    'Doors' = 4
    'Make' = 'Chevrolet'
    'Model' = 'Camaro'
}
```

Powershell objects can also define _methods_.
In keeping with our car analogy, we can think of methods in terms of what you can _do_ both to, and with the car.
A few examples of that would be: Park, Drive, Turn, Start, Stop, and Wash.

When working with Powershell, objects contain the methods defined as part of the _type_ object that is returned, but can be extended with methods that you create yourself.
These are called _Script Methods_, and are simply defined as Script blocks when creating an object.
Importantly, these are defined using the `Add-Member` cmdlet.

Let's add a method to our object:

```powershell
Add-Member -InputObject $car -MemberType ScriptMethod -Name Start -Value {Write-Host "The car is now started"}
```

In the example above the `Value` parameter was defined as a scriptblock.
Now, we are able to execute our new method against the object.
Go ahead, give it a try:
`$car.Start()`

You should see that it returns `The car is now started`, which is what we told it to do in our scriptblock.
Other means to add scriptmethods to objects are outside the scope of this post, and as such won't be covered, but may be covered in a separate post in the future.

So far we have mentioned NoteProperties, and ScriptMethods.
There is another `MemberType` which can be defined, and that is the `ScriptProperty`.
The `NoteProperty` member type is a static property, meaning whatever value you give it, or change it too, is what that value is.
ScriptProperties differ in that, if you reference a static valued `Noteproperty`, the ScriptProperty`'s value can _change_, depending on what you are doing.

The last item I'd like to cover here is discoverability of important information about the different objects available in Powershell.
The first is the `.GetType()` method.
This will return the Type of a given Powershell object.
The second, is the `Get-Member` cmdlet.
This cmdlet will return a listing of all the different properties and methods available to an object.
Let's run these two things against our now finished `$car` object.

```powershell
#GetType() Method
$car.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     False    PSCustomObject                           System.Object

#Get-member
$car | Get-Member

   TypeName: System.Management.Automation.PSCustomObject
Name        MemberType   Definition
----        ----------   ----------
Equals      Method       bool Equals(System.Object obj)
GetHashCode Method       int GetHashCode()
GetType     Method       type GetType()
ToString    Method       string ToString()
Color       NoteProperty string Color=Blue
Doors       NoteProperty int Doors=4
Make        NoteProperty string Make=Chevrolet
Model       NoteProperty string Model=Camaro
```

As you can see, these two things are extremely useful in understanding the type of object you are dealing with, and what is available to you to use when working with them.

I hope you've found this post useful.
Until next time...
