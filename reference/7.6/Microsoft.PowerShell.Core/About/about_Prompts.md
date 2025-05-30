---
description: Describes the `prompt` function and demonstrates how to create a custom `prompt` function.
Locale: en-US
ms.date: 08/21/2023
online version: https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_prompts?view=powershell-7.6&WT.mc_id=ps-gethelp
schema: 2.0.0
title: about_Prompts
---
# about_Prompts

## Short description

Describes the `prompt` function and demonstrates how to create a custom
`prompt` function.

## Long description

The PowerShell command prompt indicates that PowerShell is ready to run a
command:

```
PS C:\>
```

PowerShell has a built-in `prompt` function. You can define your own customized
`prompt` function in your PowerShell profile script.

## About the `prompt` function

The `prompt` function determines the appearance of the PowerShell prompt.
PowerShell comes with a built-in `prompt` function, but you can override it by
defining your own `prompt` function.

The `prompt` function has the following syntax:

```powershell
function prompt { <function-body> }
```

The `prompt` function must return an object. As a best practice, return a
string or an object that's formatted as a string. The maximum recommended
length is 80 characters.

For example, the following `prompt` function returns a "Hello, World" string
followed by a  right angle bracket (`>`).

```powershell
PS C:\> function prompt {"Hello, World > "}
Hello, World >
```

### Getting the `prompt` function

To get the `prompt` function, use the `Get-Command` cmdlet or use the
`Get-Item` cmdlet in the Function drive.

For example:

```powershell
PS C:\> Get-Command prompt

CommandType     Name      ModuleName
-----------     ----      ----------
Function        prompt
```

To get the script that sets the value of the prompt, use the dot method to get
the **ScriptBlock** property of the `prompt` function.

For example:

```powershell
(Get-Command prompt).ScriptBlock
```

```Output
"PS $($ExecutionContext.SessionState.Path.CurrentLocation)$('>' * ($NestedPromptLevel + 1)) "
# .Link
# https://go.microsoft.com/fwlink/?LinkID=225750
# .ExternalHelp System.Management.Automation.dll-help.xml
```

Like all functions, the `prompt` function is stored in the `Function:` drive.
To display the script that creates the current `prompt` function, type:

```powershell
(Get-Item Function:prompt).ScriptBlock
```

### The default prompt

The default prompt appears only when the `prompt` function generates an error
or doesn't return an object.

The default PowerShell prompt is:

```
PS>
```

For example, the following command sets the `prompt` function to `$null`, which
is invalid. As a result, the default prompt appears.

```powershell
PS C:\> function prompt {$null}
PS>
```

Because PowerShell comes with a built-in prompt, you usually don't see the
default prompt.

### Built-in prompt

PowerShell includes a built-in `prompt` function.

```powershell
function prompt {
  "PS $($ExecutionContext.SessionState.Path.CurrentLocation)$('>' * ($NestedPromptLevel + 1)) ";
  # .Link
  # https://go.microsoft.com/fwlink/?LinkID=225750
  # .ExternalHelp System.Management.Automation.dll-help.xml
}
```

The function uses the `Test-Path` cmdlet to test whether the `$PSDebugContext`
automatic variable has a value. If `$PSDebugContext` has a value, you are
running in debugging mode, and `[DBG]:` is added to the prompt, as follows:

```Output
[DBG]: PS C:\ps-test>
```

If `$PSDebugContext` isn't populated, the function adds `PS` to the prompt.
And, the function uses the `Get-Location` cmdlet to get the current file system
directory location. Then, it adds a right angle bracket (`>`).

For example:

```Output
PS C:\ps-test>
```

If you are in a nested prompt, the function adds two angle brackets (`>>`) to
the prompt. You are in a nested prompt if the value of the `$NestedPromptLevel`
automatic variable is greater than 0.

For example, when you are debugging in a nested prompt, the prompt resembles
the following prompt:

```Output
[DBG] PS C:\ps-test>>>
```

### Changes to the prompt

The `Enter-PSSession` cmdlet prepends the name of the remote computer to the
current `prompt` function. When you use the `Enter-PSSession` cmdlet to start a
session with a remote computer, the command prompt changes to include the name
of the remote computer. For example:

```Output
PS Hello, World> Enter-PSSession Server01
[Server01]: PS Hello, World>
```

Other PowerShell host applications and alternate shells might have their own
custom command prompts.

For more information about the `$PSDebugContext` and `$NestedPromptLevel`
automatic variables, see [about_Automatic_Variables][01].

### How to customize the prompt

To customize the prompt, write a new `prompt` function. The function isn't
protected, so you can overwrite it.

To write a `prompt` function, type the following:

```powershell
function prompt { }
```

Then, between the braces, enter the commands or the string that creates your
prompt.

For example, the following prompt includes your computer name:

```powershell
function prompt {"PS [$Env:COMPUTERNAME]> "}
```

On the Server01 computer, the prompt resembles the following prompt:

```Output
PS [Server01] >
```

The following `prompt` function includes the current date and time:

```powershell
function prompt {"$(Get-Date)> "}
```

The prompt resembles the following prompt:

```Output
03/15/2012 17:49:47>
```

You can also change the default `prompt` function:

For example, the following modified `prompt` function adds `[ADMIN]:` to the
built-in PowerShell prompt when running in an elevated session.

```powershell
function prompt {
  $identity = [Security.Principal.WindowsIdentity]::GetCurrent()
  $principal = [Security.Principal.WindowsPrincipal] $identity
  $adminRole = [Security.Principal.WindowsBuiltInRole]::Administrator

  $(if (Test-Path Variable:/PSDebugContext) { '[DBG]: ' }
    elseif($principal.IsInRole($adminRole)) { "[ADMIN]: " }
    else { '' }
  ) + 'PS ' + $(Get-Location) +
    $(if ($NestedPromptLevel -ge 1) { '>>' }) + '> '
}
```

When you start PowerShell using the **Run as administrator** option, a prompt
that resembles the following prompt appears:

```Output
[ADMIN]: PS C:\ps-test>
```

The following `prompt` function displays the history ID of the next command. To
view the command history, use the `Get-History` cmdlet.

```powershell
function prompt {
   # The at sign creates an array in case only one history item exists.
   $history = @(Get-History)
   if($history.Count -gt 0)
   {
      $lastItem = $history[$history.Count - 1]
      $lastId = $lastItem.Id
   }

   $nextCommand = $lastId + 1
   $currentDirectory = Get-Location
   "PS: $nextCommand $currentDirectory >"
}
```

The following prompt uses the `Write-Host` and `Get-Random` cmdlets to create a
prompt that changes color randomly. Because `Write-Host` writes to the current
host application but doesn't return an object, this function includes a
`return` statement. Without it, PowerShell uses the default prompt, `PS>`.

```powershell
function prompt {
    $color = Get-Random -Min 1 -Max 16
    Write-Host ("PS " + $(Get-Location) +">") -NoNewline `
     -ForegroundColor $Color
    return " "
}
```

### Saving the `prompt` function

Like any function, the `prompt` function exists only in the current session. To
save the `prompt` function for future sessions, add it to your PowerShell
profiles. For more information about profiles, see [about_Profiles][04].

## See also

- [about_Automatic_Variables][01]
- [about_Debuggers][02]
- [about_Functions][03]
- [about_Profiles][04]
- [about_Scopes][05]
- [Get-History][09]
- [Write-Host][12]
- [Get-Location][10]
- [Enter-PSSession][08]
- [Get-Random][11]

<!-- link references -->
[01]: about_Automatic_Variables.md
[02]: about_Debuggers.md
[03]: about_Functions.md
[04]: about_Profiles.md
[05]: about_Scopes.md
[08]: xref:Microsoft.PowerShell.Core.Enter-PSSession
[09]: xref:Microsoft.PowerShell.Core.Get-History
[10]: xref:Microsoft.PowerShell.Management.Get-Location
[11]: xref:Microsoft.PowerShell.Utility.Get-Random
[12]: xref:Microsoft.PowerShell.Utility.Write-Host
