powershell-docs
===============

#General

##DateTime Manipulation

###Playing with Days

Getting the previous day's date, and putting it into `ISO` format.

```powershell
$ "{0:yyyyMMdd}" -f (Get-Date).AddDays(-1)
```
##File Manipulation

###Changing Extensions

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
