---  
title: "PowerShell: How to Create a Folder Only If It Doesn’t Exist"  
excerpt: "Discover how to use PowerShell to verify folder existence and automatically create it when missing."  
categories:  
  - PowerShell  
  - Tutorials  
tags:  
  - PowerShell  
  - File System Management  
  - Automation  

toc: true  
comments: true  
---  

This is a quick guide addressing a common question—one that pops up often enough to deserve a dedicated write-up.  

## Verifying Folder Existence in PowerShell  

When working with file operations, such as saving log files, ensuring the target directory exists is crucial. Without this check, your script might crash with an exception or, even worse, fail silently and lose data.  

PowerShell (and PowerShell Core) provides a built-in cmdlet to test whether a path exists:  

```powershell  
Test-Path -Path 'C:\Temp\'  
```  

The **Test-Path** cmdlet evaluates a given path and returns **$True** (a boolean) if the folder exists. You can leverage this in your scripts to conditionally create missing directories, like so:  

```powershell  
# Set the log directory path  
[string]$logPath = 'C:\MyLogPath\'  

# Create the folder only if it doesn't exist  
if (!(Test-Path -Path $logPath)) {  
    $paramNewItem = @{  
        Path      = $logPath  
        ItemType  = 'Directory'  
        Force     = $true  
    }  
    New-Item @paramNewItem  
}  
```  

I use this pattern so frequently that I’ve added it to my default script template—saving time and avoiding repetition.  

## The .NET Alternative  

Behind the scenes, **Test-Path** relies on .NET methods. If you prefer a more "hardcore" approach (or just want to show off), you can achieve the same result like this:  

```powershell  
# Define the log directory  
[string]$logPath = 'C:\MyLogPath\'  

# Check and create using .NET  
if (![System.IO.Directory]::Exists($logPath)) {  
    [System.IO.Directory]::CreateDirectory($logPath)  
}  
```  

## Final Thoughts  

Both methods accomplish the same goal. The .NET version is marginally faster but sacrifices some readability.  

Given the minimal performance difference, I recommend the first approach—it keeps your code clean and maintainable, especially for others who might work with it later.  

---