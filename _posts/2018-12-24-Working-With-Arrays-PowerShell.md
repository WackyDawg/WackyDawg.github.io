---
title: "Modifying Arrays in PowerShell: Adding and Removing Items"  
excerpt: "Discover how to insert or delete items within a PowerShell array using dynamic list structures."  
categories:
  - PowerShell  
tags: 
  - PowerShell  
  - Arrays  
  - PowerShell Fundamentals  
  - Tutorial  

toc: true  
---

This is a topic I'm often asked about, so I decided to write a guide on it. As the name suggests, this tutorial will walk you through the process of inserting or deleting values from an array in PowerShell after its initial creation.

## Understanding Arrays in PowerShell

Let’s begin with how PowerShell defines an array, straight from its official documentation:

> An array is a data container crafted to hold multiple values. These items may be of identical types or varied ones.  
> Starting with PowerShell version 3.0, even a single item (or none) can mimic the behavior of an array.

Creating arrays in PowerShell is very straightforward. You've likely done it already—say, when retrieving user records from Active Directory:

```powershell
$adUsers = Get-AdUser -Filter '*' -Server $adDomainController
```

This command yields an array of Active Directory entries that fit the specified criteria.

You can also initialize a blank array like this:

```powershell
$myArray = @()

# Declare specific object type
[array]$myArray = @()
```

This creates an empty array ready to be filled—either with queried data or static values:

```powershell
$myArray = (1,2,3,4,5)
```

This assigns five integers to *myArray*, effectively initializing a new array with a length of **5**.

## Adding Data to a PowerShell Array

So far, we’ve discussed array creation and basic value assignment. But what if you want to add a sixth value to *myArray*?

If you try the following:

```powershell
$myArray.Add(6)
```

You’ll encounter this error:

```powershell
Exception calling "Add" with "1" argument(s): "Collection was of a fixed size."
At line:1 char:1
+ $myArray.Add(6)
+ ~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
+ FullyQualifiedErrorId : NotSupportedException
```

This error occurs because arrays in PowerShell have a predetermined size and cannot grow dynamically.

A common workaround is using the `+=` operator, like so:

```powershell
$myArray += 6

# Check the new array length
$myArray.length

6
```

Behind the scenes, here’s what PowerShell does:

- It builds a fresh array combining the old elements with the new one.  
- It then replaces the original array with this newly created version.

To you, it seems like one seamless action—but it involves a complete reallocation.

## Introducing ArrayList for Dynamic Arrays

If you’d rather avoid this behind-the-scenes data copying, you can use the **[ArrayList](https://docs.microsoft.com/en-us/dotnet/api/system.collections.arraylist?view=netframework-4.7.2)** class, which supports dynamic growth and shrinkage:

```powershell
# Instantiate an ArrayList object
$myArrayList = New-Object System.Collections.ArrayList($null)

# Insert values into the list
[void]($myArrayList.Add(1))
[void]($myArrayList.Add(2))

# Check count
2
```

To remove an item, you can do:

```powershell
# Removes element at index 1
$myArrayList.RemoveAt(1)
```

**Note:** Using `.Remove()` deletes by value, not by index—so be cautious.

> Prefixing with `[void]` silences the output that `.Add()` usually returns (i.e., the new size).

### A Simpler Way to Create ArrayLists

Calling `New-Object` is a bit resource-heavy. A more efficient approach is:

```powershell
[System.Collections.ArrayList]$myArray = @()
```

This form not only reduces overhead but also keeps your scripts optimized and cleaner—especially useful for large or performance-critical scripts.

Alternatively, convert a regular array into an `ArrayList` with:

```powershell
$myArray = [System.Collections.ArrayList]@()
```

## Final Thoughts

Although PowerShell doesn’t enforce data type declarations, I recommend explicitly stating types, like so:

```powershell
[string]$myString = 'Some Text'
```

Instead of:

```powershell
$myString = 'Some Text'
```

Doing so boosts clarity and lets you know exactly what properties and methods an object supports—particularly helpful in larger scripts when tracing a variable declared hundreds of lines earlier.

---