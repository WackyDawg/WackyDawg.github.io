---
title: "Mastering PowerShell Hashtables"
excerpt: "A comprehensive guide to working with Hashtables in PowerShell — from the basics to practical use cases."
categories:
  - PowerShell
tags:
  - PowerShell
  - Hashtable
  - PowerShell Core
  - Dictionaries
  - Associative Arrays
---

## Introduction to PowerShell Hashtables

Whether you call them **Hashtables**, *Dictionaries*, or *Associative Arrays*, they are essential data structures in PowerShell. Although they might appear daunting at first, they offer incredible power and flexibility once understood.

Personally, I’ve found hashtables indispensable — not just for performance reasons, but because they were central to one of the first major scripts I ever wrote.

## What is a Hashtable?

In PowerShell, a **hashtable** is a collection of key/value pairs. It works similarly to an array, but instead of referencing items by numerical index, you use a unique key.

Here’s how you define and use one:

```powershell
# Standard array
[array]$myArray = @(
    'Element 1',
    'Element 2',
    'Element 3'
)

# Hashtable with keys
[hashtable]$myHashtable = @{
    1 = 'Element 1';
    2 = 'Element 2';
    3 = 'Element 3'
}

# Creating an empty hashtable
[hashtable]$emptyHash = @{}
```

Notice: Arrays use parentheses `()` while hashtables use curly braces `{}`. Hashtables also structure values in key/value pairs.

## Adding Items to a Hashtable

Hashtables in PowerShell are dynamic, meaning you can append new items at any time.

```powershell
# Initialize
[hashtable]$userAges = @{}

# Add entries
$userAges.Add('Alice', 28)
$userAges.Add('Bob', 34)
$userAges.Add('Charlie', 41)

$userAges
```

⚠️ **Note:** All keys in a hashtable must be **unique**.

## Retrieving Values from a Hashtable

Fetching values by key is just as intuitive:

```powershell
$charlieAge = $userAges['Charlie']
$charlieAge  # Outputs 41
```

## Modifying Existing Entries

To update a value, assign a new one to the corresponding key:

```powershell
$userAges['Alice'] = 29
```

You're using the same syntax you would for reading a value — the brackets specify the key you want to update.

## Real-World Examples

### Using Hashtables as Lookup Maps

A common scenario: storing Active Directory paths based on city names.

```powershell
[hashtable]$ouLocations = @{
    'London'      = 'OU=London,DC=Domain,DC=com'
    'Berlin'      = 'OU=Berlin,DC=Domain,DC=com'
    'San Diego'   = 'OU=San Diego,DC=Domain,DC=com'
}

$userCity = 'London'
Move-ADUser -Identity $user -DestinationPath $ouLocations[$userCity]
```

This was part of an actual script I wrote for automating AD user moves based on UI input.

### Nesting Hashtables

Hashtables can contain other hashtables — useful for organizing hierarchical or grouped data.

```powershell
[hashtable]$siteInfo = @{
    'Paris' = 'OU=Paris,DC=Domain,DC=com'
    'Tokyo' = 'OU=Tokyo,DC=Domain,DC=com'
    'Metadata' = @{
        'Region' = 'APAC'
        'Language' = 'Japanese'
    }
}

# Display nested hashtable
$siteInfo['Metadata']
$siteInfo['Metadata']['Region']
```

### Exporting Hashtables to CSV

To export a hashtable to a CSV file, convert it into a format that Export-CSV can handle:

```powershell
$siteInfo | ForEach-Object { [pscustomobject]$_ } | Export-CSV -Path $csvFile -NoTypeInformation
```

### Converting Hashtables to Strings for Logging

When logging data to files, use `Out-String` to serialize your hashtable:

```powershell
$hashAsString = ($siteInfo | Out-String).Trim()
```

This converts the contents to a clean string that can be used in logs.

## Final Thoughts

PowerShell hashtables are incredibly versatile and powerful. We’ve covered the fundamentals and real-world use cases, but there’s still more — including serialization to JSON, deep merging, and more — which I’ll explore in future posts.
```
---