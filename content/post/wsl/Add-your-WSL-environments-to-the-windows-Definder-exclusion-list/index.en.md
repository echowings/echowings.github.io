---
title: "Add Your WSL Environments to the Windows Definder Exclusion List"
date: 2023-12-22T22:48:34+08:00
draft: False
image: https://picsum.photos/800/600.webp?random=3f59145a
categories:
  - wsl
tags:
  - wsl
  - powershell
  - windows definder
  - windows security
slug:
  - wsl
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---


```powershell
############
# This script will add your WSL environments to the Windows Defender exclusion list so that
# realtime protection does not have an adverse effect on performance.
#
# You should be aware that this could make your system less secure. Use at your own risk.
# Note: This should be run from an administrative PowerShell prompt
############

# Find registered WSL environments
$wslPaths = (Get-ChildItem HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss | ForEach-Object { Get-ItemProperty $_.PSPath}).BasePath

# Get the current Windows Defender exclusion paths
$currentExclusions = $(Get-MpPreference).ExclusionPath
if (!$currentExclusions) {
  $currentExclusions = ''
}

# Find the WSL paths that are not excluded
$exclusionsToAdd = ((Compare-Object $wslPaths $currentExclusions) | Where-Object SideIndicator -eq "<=").InputObject

# List of paths inside the Linux distro to exclude (https://github.com/Microsoft/WSL/issues/1932#issuecomment-407855346)
$dirs = @("\bin", "\sbin", "\usr\bin", "\usr\sbin", "\usr\local\bin", "\usr\local\go\bin")

# Add the missing entries to Windows Defender
if ($exclusionsToAdd.Length -gt 0) {
  $exclusionsToAdd | ForEach-Object { 
    
    # Exclude paths from the root of the WSL install
    Add-MpPreference -ExclusionPath $_
    Write-Output "Added exclusion for $_"

    # Exclude processes contained inside WSL
    $rootfs = $_ + "\rootfs"
    $dirs | ForEach-Object {
        $exclusion = $rootfs + $_ + "\*"
        Add-MpPreference -ExclusionProcess $exclusion
        Write-Output "Added exclusion for $exclusion"
    }
  }
}
```

