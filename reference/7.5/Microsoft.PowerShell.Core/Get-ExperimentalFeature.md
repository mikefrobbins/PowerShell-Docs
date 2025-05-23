---
external help file: System.Management.Automation.dll-Help.xml
Locale: en-US
Module Name: Microsoft.PowerShell.Core
ms.date: 12/04/2023
online version: https://learn.microsoft.com/powershell/module/microsoft.powershell.core/get-experimentalfeature?view=powershell-7.5&WT.mc_id=ps-gethelp
schema: 2.0.0
title: Get-ExperimentalFeature
---
# Get-ExperimentalFeature

## SYNOPSIS
Gets experimental features.

## SYNTAX

```
Get-ExperimentalFeature [[-Name] <String[]>] [<CommonParameters>]
```

## DESCRIPTION

The `Get-ExperimentalFeature` cmdlet returns all experimental features discovered by PowerShell.
Experimental features can come from modules or the PowerShell engine. Experimental features allow
users to safely test new features and provide feedback (typically via GitHub) before the design is
considered complete and any changes can become a breaking change.

## EXAMPLES

### Example 1

Gets the list of currently registered experimental features and their current state.

```powershell
Get-ExperimentalFeature
```

```Output
Name                         Enabled Source      Description
----                         ------- ------      -----------
PSImplicitRemotingBatching   False PSEngine      Batch implicit remoting proxy commands to improve performance
```

## PARAMETERS

### -Name

Name or names of specific experimental features to return.

```yaml
Type: System.String[]
Parameter Sets: (All)
Aliases:

Required: False
Position: 0
Default value: None
Accept pipeline input: True (ByValue)
Accept wildcard characters: False
```

### CommonParameters

This cmdlet supports the common parameters: -Debug, -ErrorAction, -ErrorVariable,
-InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose,
-WarningAction, and -WarningVariable. For more information, see
[about_CommonParameters](https://go.microsoft.com/fwlink/?LinkID=113216).

## INPUTS

### System.String[]

You can pipe a string containing the name of an experimental feature to this cmdlet.

## OUTPUTS

### System.Management.Automation.ExperimentalFeature

This cmdlet returns instances that match the requested names or all experimental features if no
name is specified.

## RELATED LINKS

[Disable-ExperimentalFeature](Disable-ExperimentalFeature.md)

[Enable-ExperimentalFeature](Enable-ExperimentalFeature.md)

[about_Experimental_Features](About/about_Experimental_Features.md)

[Using Experimental Features](/powershell/scripting/learn/experimental-features)
