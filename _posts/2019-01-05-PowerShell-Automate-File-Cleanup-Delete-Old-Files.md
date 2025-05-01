---
title: "Automate File Cleanup with PowerShell: Delete Old Files by Age"  
excerpt: "Step-by-step guide to purge stale files and folders using PowerShell—set retention policies and free up disk space automatically."  
categories:  
  - PowerShell  
  - Automation  
  - File Management  
tags:  
  - PowerShell  
  - Disk Cleanup  
  - Scheduled Tasks  
  - Log Rotation  

toc: true  
comments: true  
---  

### Why Automate File Cleanup?  

In automation workflows, logging is critical for troubleshooting—but over time, log files consume valuable disk space. Instead of manual cleanup, PowerShell lets you systematically remove files older than a defined threshold (e.g., 90 days). Here’s how to implement it.  

### Defining the Scope  

Before writing code, clarify these requirements:  

1. **Scope**: Should subfolders be included?  
2. **Target Files**: Which extensions (e.g., `.log`)?  
3. **Retention**: How many days of files to keep?  

For this example, we’ll use:  

| **Setting**          | **Value**       |  
|----------------------|----------------|  
| Recurse Subfolders   | Yes            |  
| File Extension       | `*.log`        |  
| Retention Period     | 90 days        |  

---

### Step 1: Configure File Selection  

Define variables for the target path, file type, and age limit:  

```powershell  
[string]$filePath = 'C:\TestCleanup\'  
[string]$fileExtension = '*.log'  
[int]$retentionDays = 90  
[datetime]$cutoffDate = (Get-Date).AddDays(-$retentionDays)  
```  

- **`$cutoffDate`** calculates the threshold date (today minus 90 days).  

---

### Step 2: Retrieve Old Files  

Use `Get-ChildItem` to find files matching the criteria:  

```powershell  
[array]$oldFiles = Get-ChildItem -Path $filePath -Filter $fileExtension -Recurse -File |  
    Where-Object { $_.LastWriteTime -lt $cutoffDate }  
```  

This fetches all `.log` files last modified before the cutoff, including subfolders.  

---

### Step 3: Safe Deletion  

Loop through the results and delete files—with a check to avoid errors if no files exist:  

```powershell  
if ($oldFiles.Count -gt 0) {  
    foreach ($file in $oldFiles) {  
        Remove-Item -Path $file.FullName -Confirm:$false  
        Write-Verbose "Deleted: $($file.FullName)"  
    }  
}  
else {  
    Write-Host "No files older than $retentionDays days found."  
}  
```  

**Key Improvements**:  
- Added `-Recurse` to scan subfolders.  
- Verbose logging for transparency.  

---

### Extending the Script  

This baseline can be enhanced with:  
- **Multiple Paths**: Process several directories in one run.  
- **Exclusions**: Skip specific folders (e.g., `C:\TestCleanup\Keep\*`).  
- **Centralized Config**: Store settings in a CSV/JSON file.  

---

### Ready-Made Solution  
**, which adds:  
✅ Custom retention rules per folder  
✅ Email notifications  
✅ Ignore lists for critical paths  

---

### Final Thoughts  

Automating file cleanup reduces manual effort and prevents disk bloat. While the example above handles basic needs, consider integrating it into:  
- **Scheduled Tasks**: Run weekly/monthly.  
- **CI/CD Pipelines**: Prune build artifacts post-deployment.  

--- 