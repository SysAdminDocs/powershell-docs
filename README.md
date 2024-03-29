powershell-docs
===============

# Table of Contents

- [General](#general)
    - [DateTime Manipulation](#datetime-manipulation)
        - [Playing with Days](#playing-with-days)
    - [File Manipulation](#file-manipulation)
        - [Changing Extensions](#changing-extensions)
        - [Pin Pointing Information in a CSV](#pin-pointing-information-in-a-csv)
    - [SSH & SCP Operations](#ssh-and-scp-operations)
        - [Execute Commands Over SSH](#execute-commands-over-ssh)
        - [Synchronize a Remote Directory to Local](#synchronize-a-remote-directory-to-local)

# General

## DateTime Manipulation

### Playing with Days

Getting the previous day's date, and putting it into `ISO` format.

```powershell
$ "{0:yyyyMMdd}" -f (Get-Date).AddDays(-1)
```

Another method is by piping the values. In the following example I want to get the previous months abbreviated name.

```powershell
$ (Get-Date).AddMonths(-1) | Get-Date -UFormat %b
```
## File Manipulation

### Changing Extensions

Change the extensions of all files within a folder.
```powershell
$Global:path = "C:\Temp"
$Global:filesToChange = $(Get-ChildItem -Path "$Global:path\*.*" -Include *.txt)

ForEach($file in $Global:filesToChange) {
    $newFileName = $(Split-Path -Path $file -Leaf | % { $_.split(.)[0] } | % { "$_.csv" }
    Rename-Item -Path $file -NewName $newFileName
}
```

This can also be done recursively as per below:

```powershell
$Global:path = "C:\Temp"
$Global:filesToChange = $(Get-ChildItem -Recurse -Path "$Global:path\" -Include *.txt)

ForEach($file in $Global:filesToChange) {
    $newFileName = $(Split-Path -Path $file -Leaf | % { $_.split(.)[0] } | % { "$_.csv" }
    Rename-Item -Path $file -NewName $newFileName
}
```

### Pin Pointing Information in a CSV

This is about getting a value of a column using in essence _x,y_ coordinates.

Here is an example CSV that we will use. This example will be looking at a backup schedule, and if a server should be backed up on a particular date or not.

```csv
serverName,20140826,20140827,20140828,20140829
myServer1,0,1,0,0
myServer2,1,0,1,0
myServer3,0,0,1,1
```

The above csv has the filename `schedule.csv`, and will be sitting in the same directory as the script.

```powershell
$todaysDate = Get-Date -Format "yyyyMMdd"
$localMachine = $env:computername
$scheduleCsv = Import-Csv -Path "$PSScriptRoot\schedule.csv"

$returnValue = $scheduleCsv | ? { $_.serverName -eq $localMachine } | % { $_.$todaysDate }

if($returnValue -eq 1) {
    Write-Host ("{0} should backup today" -f $localMachine)
} else {
    Write-Warning ("{0} should not backup today" -f $localMachine)
}
```
## SSH & SCP Operations

These operations all require WinSCP.net and the following import line.
`Add-Type -Path "C:\WinSCP\WinSCPnet.dll"

### Execute Command Over SSH

This can be used to execute a command on a remote server using SSH. Please keep in mind that in order to prefix variables with `$Local:` they must be declared inside of a function, otherwise omit the `$Local:`.

```powershell
# -- Declare all the Session Options
$Local:sessionOptions = New-Object WinSCP.SessionOptions
$Local:sessionOptions.Protocol = [WinSCP.Protocol]::Sftp
$Local:sessionOptions.HostName = "server.example.com"
$Local:sessionOptions.UserName = "root"
  # -- Please see note below about the SshPrivateKeyPath
$Local:sessionOptions.SshPrivateKeyPath = "myPrivateKey"
$Local:sessionOptions.SshHostKeyFingerprint = "ssh-rsa 2048 xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx"
$Local:sessionOptions.TimeoutInMilliseconds = "30000"

# -- Create the Session
$Local:session = New-Object WinSCP.Session

try
{
    # -- Initialize and Open the Session
    $Local:session.Open($Local:sessionOptions)
    
    # -- Build the command for execution
    $Local:command = "ls -l /bin"
    
    # -- Execute the Command
    $Local:session.ExecuteCommand($Local:command)
}
catch [Exception]
{
    Write-Error $_.Exception.Message
}
finally
{
    $Local:session.Dispose()
}
```

As per the above note about the `SshPrivateKeyPath`, with this you can either define the path to the key (_where the key is not encrypted and does not require a password_) or, if you just put the __name__ of the key and nothing else, as long as the key is loaded in [Pageant](http://en.wikipedia.org/wiki/PuTTY#Components) then it will use that keys. As such you will be able to have an encrypted SSH Private Key.

### Synchronize a Remote Directory to Local

This will sync a directory from a remote server to a directory on our local machine, after the sync has completed it will also remove the file from the remote machine.

```powershell
# -- Declare all the Session Options
$Local:sessionOptions = New-Object WinSCP.SessionOptions
$Local:sessionOptions.Protocol = [WinSCP.Protocol]::Sftp
$Local:sessionOptions.HostName = "server.example.com"
$Local:sessionOptions.UserName = "root"
$Local:sessionOptions.SshPrivateKeyPath = "myPrivateKey"
$Local:sessionOptions.SshHostKeyFingerprint = "ssh-rsa 2048 xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx"
$Local:sessionOptions.TimeoutInMilliseconds = "30000"

# -- Create the Session
$Local:session = New-Object WinSCP.Session

try
{
    # -- Initialize and Open Session
    $Local:session.Open($Local:sessionOptions)
    
    # -- Set the Remote and Local Paths to Variables
    $Local:remotePath = "/home/documents"
    $Local:localPath = "C:\Users\myUser\Documents"
    
    # -- Synchronize Directories and Collect Results
    $Local:synchronizationResult = $Local:session.SychronizeDirectories(
        [WinSCP.SynchronizationMode]::Local, $Local:localPath, $Local:remotePath, $False)
    
    ForEach($Local:download in $Local:synchronizationResult.Downloads) {
        if($Local:download.Error -eq $Null) {
            Write-Host ("Download of {0} succeeded, removing from source" -f
                $Local:download.FileName)
            
            # -- Remove Remote File
            $Local:removalResult = $Local:session.RemoveFiles(
                $Local:session.EscapeFileMask($Local:download.FileName))
            
            if($Local:removalResult.IsSuccess) {
                Write-Host ("Removal of {0} succeeded." -f
                    $Local:download.FileName)
            } else {
                Write-Error ("Removal of {0} failed!" -f
                    $Local:download.FileName)
            }
        } else {
            Write-Error ("Download of {0} failed: {1}" -f
                $Local:download:FileName, $Local:download.Error.Message)
        }
    }
}
catch [Exception]
{
    Write-Error $_.Exception.Message
}
finally
{
    $Local:session.Dispose()
}
```
