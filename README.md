# RunAsUser Module

This module has been created to have the ability to run scripts under the current user session while the application executing this script only has SYSTEM access. This is especially useful when performing tasks from RMM(Remote Monitoring and Management) systems that do not have the abilty to execute monitoring components in user-space.

This script is based on [Murrayju](https://github.com/murrayju/CreateProcessAsUser) his work with CreateProcessAsUser.

# Installation instructions

This module has been published to the PowerShell Gallery. Use the following command to install:  

    install-module RunAsUser


# Usage

To execute a script under the current user you'll need to run the script as SYSTEM using your RMM or other methods. To execute the script run the following command

    $scriptblock = { "Hello world" | out-file "C:\Temp\HelloWorld.txt" }
    
    invoke-ascurrentuser -scriptblock $scriptblock

The script will run, store a file with the results in C:\Temp\Helloworld.txt that you can pick up with another PowerShell command such as get-content.

  
**Examples:**

To get the OneDrive files in the currently logged on user profile:

 
    $scriptblock = {
    $IniFiles = Get-ChildItem "$ENV:LOCALAPPDATA\Microsoft\OneDrive\settings\Business1" -Filter 'ClientPolicy*' -ErrorAction SilentlyContinue
 
    if (!$IniFiles) {
        write-host 'No Onedrive configuration files found. Stopping script.'
        exit 1
    }
     
    $SyncedLibraries = foreach ($inifile in $IniFiles) {
        $IniContent = get-content $inifile.fullname -Encoding Unicode
        [PSCustomObject]@{
            'Item Count' = ($IniContent | Where-Object { $_ -like 'ItemCount*' }) -split '= ' | Select-Object -last 1
            'Site Name'  = ($IniContent | Where-Object { $_ -like 'SiteTitle*' }) -split '= ' | Select-Object -last 1
            'Site URL'   = ($IniContent | Where-Object { $_ -like 'DavUrlNamespace*' }) -split '= ' | Select-Object -last 1
        }
    }
    $SyncedLibraries | ConvertTo-Json | Out-File 'C:\programdata\Microsoft OneDrive\OneDriveLibraries.txt'
    }
    Invoke-ascurrentuser -scriptblock $scriptblock
    start-sleep 2 #Sleeping 2 seconds to allow script to write to disk.
    $SyncedLibraries = (get-content "C:\programdata\Microsoft OneDrive\OneDriveLibraries.txt" | convertfrom-json)
    if (($SyncedLibraries.'Item count' | Measure-Object -Sum).sum -gt '280000') { 
    write-host "Unhealthy - Currently syncing more than 280k files. Please investigate."
    $SyncedLibraries
    }
    else {
    write-host "Healthy - Syncing less than 280k files."
    }

As this script demonstrates, all user variables are the one of the current logged on user, instead of the SYSTEM account. You can also use this to browse the HCKU registry tree, or any files or shares to which only the user has access

# Contributions

Feel free to send pull requests or fill out issues when you encounter them. I'm also completely open to adding direct maintainers/contributors and working together! :)

# Future plans

Version 1.0 includes all things I required for myself, if you need a feature, shoot me a feature request :)

- [x] Allow running scripts impersonating the currently logged on user