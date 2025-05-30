---
description: An ISEFile object represents a file in Windows PowerShell ISE.
ms.date: 03/27/2025
title: The ISEFile Object
---

# The ISEFile Object

An **ISEFile** object represents a file in Windows PowerShell Integrated Scripting Environment
(ISE). It's an instance of the **Microsoft.PowerShell.Host.ISE.ISEFile** class. This topic lists its
member methods and member properties. The `$psISE.CurrentFile` and the files in the Files collection
in a PowerShell tab are all instances of the **Microsoft.PowerShell.Host.ISE.ISEFile** class.

## Methods

### Save( [saveEncoding] )

Supported in Windows PowerShell ISE 2.0 and later.

Saves the file to disk.

`[saveEncoding]` - optional [System.Text.Encoding][01] An optional
character encoding parameter to be used for the saved file. The default value is **UTF8**.

### Exceptions

- **System.IO.IOException**: The file couldn't be saved.

```powershell
# Save the file using the default encoding (UTF8)
$psISE.CurrentFile.Save()

# Save the file as ASCII.
$psISE.CurrentFile.Save([System.Text.Encoding]::ASCII)

# Gets the current encoding.
$myfile = $psISE.CurrentFile
$myfile.Encoding
```

### SaveAs(filename, [saveEncoding])

Supported in Windows PowerShell ISE 2.0 and later.

Saves the file with the specified file name and encoding.

**filename** - String
The name to be used to save the file.

`[saveEncoding]` - optional [System.Text.Encoding][01] An optional
character encoding parameter to be used for the saved file. The default value is **UTF8**.

### Exceptions

- **System.ArgumentNullException**: The **filename** parameter is null.
- **System.ArgumentException**: The **filename** parameter is empty.
- **System.IO.IOException**: The file couldn't be saved.

```powershell
# Save the file with a full path and name.
$fullpath = "C:\temp\newname.txt"
$psISE.CurrentFile.SaveAs($fullPath)
# Save the file with a full path and name and explicitly as UTF8.
$psISE.CurrentFile.SaveAs($fullPath, [System.Text.Encoding]::UTF8)
```

## Properties

### DisplayName

Supported in Windows PowerShell ISE 2.0 and later.

The read-only property that gets the string that contains the display name of this file. The name is
shown on the **File** tab at the top of the editor. The presence of an asterisk (`*`) at the end of
the name indicates that the file has changes that haven't been saved.

```powershell
# Shows the display name of the file.
$psISE.CurrentFile.DisplayName
```

### Editor

Supported in Windows PowerShell ISE 2.0 and later.

The read-only property that gets the [editor object][04] that's used for the specified file.

```powershell
# Gets the editor and the text.
$psISE.CurrentFile.Editor.Text
```

### Encoding

Supported in Windows PowerShell ISE 2.0 and later.

The read-only property that gets the original file encoding. This is a **System.Text.Encoding**
object.

```powershell
# Shows the encoding for the file.
$psISE.CurrentFile.Encoding
```

### FullPath

Supported in Windows PowerShell ISE 2.0 and later.

The read-only property that gets the string that specifies the full path of the opened file.

```powershell
# Shows the full path for the file.
$psISE.CurrentFile.FullPath
```

### IsSaved

Supported in Windows PowerShell ISE 2.0 and later.

The read-only Boolean property that returns `$true` if the file has been saved after it was last
modified.

```powershell
# Determines whether the file has been saved since it was last modified.
$myfile = $psISE.CurrentFile
$myfile.IsSaved
```

### IsUntitled

Supported in Windows PowerShell ISE 2.0 and later.

The read-only property that returns `$true` if the file has never been given a title.

```powershell
# Determines whether the file has never been given a title.
$psISE.CurrentFile.IsUntitled
$psISE.CurrentFile.SaveAs("temp.txt")
$psISE.CurrentFile.IsUntitled
```

## See Also

- [The ISEFileCollectionObject][05]
- [Purpose of the Windows PowerShell ISE Scripting Object Model][02]
- [The ISE Object Model Hierarchy][03]

<!-- link references -->
[01]: /dotnet/api/system.text.encoding
[02]: Purpose-of-the-Windows-PowerShell-ISE-Scripting-Object-Model.md
[03]: The-ISE-Object-Model-Hierarchy.md
[04]: The-ISEEditor-Object.md
[05]: The-ISEFileCollection-Object.md
