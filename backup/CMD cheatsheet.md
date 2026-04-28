


## Actions

* open the current directory: `start .`

*  use the `help` command followed by the name of the command: `help dir`

* [Windows command line search for exact extension with dir](https://stackoverflow.com/questions/2423935/windows-command-line-search-for-exact-extension-with-dir) [^2]:  `dir  /s /b *.txt | findstr /v .txt.`

  







## Lookup

| Commands(windows)                        | Short description                        | Equivalent<br />PowerShell      | macOS |
| ---------------------------------------- | ---------------------------------------- | ------------------------------- | ----- |
| [ START](https://ss64.com/nt/start.html) | Start a program, command or batch file • | Invoke-Item <br />Start-Process | open  |
| [ DEL](https://ss64.com/nt/del.html)     | Delete one or more files •               |                                 | rm    |
| [ DIR](https://ss64.com/nt/dir.html)     | Display a list of files and folders •    | ls                              |       |
| [ HELP](https://ss64.com/nt/help.html)   | Online Help                              | Get-Help                        | man   |





## System Environment Variable

**Set the Environment Variable**: 

**Open Command Prompt as Administrator**: Press `Windows` + `X` and select “Command Prompt (Admin)” or “Windows PowerShell (Admin)” to open a terminal with administrator privileges.

```powershell
$Env:PATH
```

nicely printed
```powershell
$Env:PATH -split ';' | ForEach-Object { $_ }
```

```cmd
for %G in ("%PATH:;=" "%") do @echo %G
```

with filter 

```cmd
for %G in ("%PATH:;=" "%") do @echo %G | find "conda"
```



**Set the Environment Variable**: You can use the `setx` command to add the directory containing `conda.bat` to your system's `Path` environment variable. The command format is as follows:

```pow	
setx PATH "%PATH%;C:\ProgramData\anaconda3\condabin"
```



Remove a PATH

* You can do this through the System Properties dialog (search for "Edit the system environment variables") 

  * Press `Windows Key + R` to open the Run dialog. 
    * OR open `CMD `windown
  * Type `sysdm.cpl` and press `Enter` or click `OK`.
  * This command opens the System Properties dialog directly. 
    From there, you can navigate to the `Advanced` tab 
    and click on `Environment Variables` to edit, add, or delete system and user environment variables.

  Another method to directly reach the Environment Variables window is:

  1. Press `Windows Key + R` to open the Run dialog.
  2. Type `rundll32 sysdm.cpl,EditEnvironmentVariables` and press `Enter` or click `OK`.

FAQ:
 * conda cannot activate env in PowerShell[^3]

## References

[ ^1 ]:  https://ss64.com/nt/
[^2]: [ DIR](https://ss64.com/nt/dir.html) Display a list of files and folders • 
[^3]: https://github.com/jinsanity07git/jinsanity07git.github.io/issues/9
