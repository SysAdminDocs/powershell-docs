powershell-docs
===============

#General

##DateTime Manipulation

###Playing with Days

Getting the previous day's date, and putting it into `ISO` format.

```powershell
$ "{0:yyyyMMdd}" -f (Get-Date).AddDays(-1)
```
