---
title: "PowerShell Arrays: The Complete Guide to Collection Management"  
excerpt: "Master PowerShell arrays - from basic operations to advanced manipulation techniques for efficient scripting."  
categories:  
  - PowerShell  
  - Data Structures  
tags:  
  - Arrays  
  - Collection Types  
  - PowerShell Core  
  - Script Optimization  

toc: true  
comments: true  
---  

## Understanding PowerShell Arrays  

Arrays are fundamental data structures that store collections of items in PowerShell. They can hold elements of any type and are essential for effective scripting and data management.  

### Array Declaration Basics  

```powershell  
# Basic array with values  
$numberArray = 22, 5, 10, 8, 12, 9, 80  

# Empty array initialization  
$emptyArray = @()  

# Range operator array  
$rangeArray = 100..110  

# Mixed-type array  
$mixedArray = "PowerShell", 42, (Get-Date), $true  
```  

**Key Property**:  
```powershell  
$rangeArray.Length  # Returns 11 (100-110 inclusive)  
```  

---

## Accessing Array Elements  

### Index-Based Access  

PowerShell arrays are zero-indexed:  

```powershell  
$rangeArray[0]    # Returns 100 (first element)  
$rangeArray[-1]   # Returns 110 (last element)  
$rangeArray[2..5] # Returns 102,103,104,105 (range)  
$rangeArray[0,3,6] # Returns 100,103,106 (specific indices)  
```  

### Iterating Through Arrays  

```powershell  
foreach ($number in $rangeArray) {  
    Write-Output "Processing: $number"  
}  

# Alternative with For loop  
for ($i=0; $i -lt $rangeArray.Length; $i++) {  
    Write-Output "Index $i contains $($rangeArray[$i])"  
}  
```  

---

## Specialized Array Techniques  

### Strongly Typed Arrays  

Enforce data type consistency:  

```powershell  
[int[]]$intOnly = 1300, 22, 4578, 8000  
[datetime[]]$dateArray = (Get-Date), "2023-01-01"  

# This will throw an error:  
$intOnly[0] = "string"  
```  

### Practical AD Example  

```powershell  
$adUsers = @()  
$adUsers = Get-ADUser -Filter * -Properties Mail  

foreach ($user in $adUsers) {  
    if ($user.Mail) {  
        Write-Output "Processing $($user.SamAccountName)"  
    }  
}  
```  

---

## Array Manipulation  

### Modifying Elements  

```powershell  
# Update by index  
$rangeArray[0] = 200  

# Using SetValue method  
$rangeArray.SetValue(201, 1)  
```  

### Adding Elements  

```powershell  
# Note: Creates new array (inefficient for large arrays)  
$rangeArray += 500  

# Better alternative for large arrays:  
$list = [System.Collections.ArrayList]::new($rangeArray)  
$list.Add(500) | Out-Null  
```  

### Removing Elements  

```powershell  
# Create new array without last element  
$trimmedArray = $rangeArray[0..($rangeArray.Length-2)]  

# Filter specific elements  
$filteredArray = $rangeArray | Where-Object { $_ -gt 105 }  
```  

### Clearing Arrays  

```powershell  
$rangeArray = $null  # Best performance  
Remove-Variable rangeArray  # Alternative method  
```  

---

## Performance Considerations  

1. **Array Addition**: `+=` creates new arrays - use `ArrayList` for frequent modifications  
2. **Large Datasets**: Consider `[System.Collections.Generic.List[object]]` for better performance  
3. **Memory Management**: Explicitly clear large arrays when no longer needed  

**Pro Tip**: For advanced scenarios, explore:  
- Multidimensional arrays  
- Jagged arrays (arrays of arrays)  
- `[System.Collections.Specialized.CollectionsUtil]::CreateCaseInsensitiveHashtable()`  

---

---