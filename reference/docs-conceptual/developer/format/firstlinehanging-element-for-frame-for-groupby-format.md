---
description: FirstLineHanging Element for Frame for GroupBy
ms.date: 08/23/2021
title: FirstLineHanging Element for Frame for GroupBy
---
# FirstLineHanging Element for Frame for GroupBy

Specifies how many characters the first line of data is shifted to the left. This element is used
when defining how a new group of objects is displayed.

## Schema

- Configuration Element
- ViewDefinitions Element
- View Element
- GroupBy Element
- CustomControl Element
- CustomEntries Element
- CustomEntry Element
- CustomItem Element
- Frame Element
- FirstLineHanging Element

## Syntax

```xml
<FirstLineHanging>NumberOfCharactersToShift</FirstLineHanging>
```

## Attributes and Elements

The following sections describe attributes, child elements, and parent element of the
`FirstLineHanging` element.

### Attributes

None.

### Child Elements

None.

### Parent Elements

|Element|Description|
|-------------|-----------------|
|[Frame Element for CustomItem for GroupBy](./frame-element-for-customitem-for-groupby-format.md)|Defines how the data is displayed, such as shifting the data to the left or right.|

## Text Value

Specify the number of characters that you want to shift the first line of the data.

## Remarks

If this element is specified, you cannot specify the [FirstLineIndent](./firstlineindent-element-for-frame-for-groupby-format.md) element.

## See Also

[FirstLineIndent Element for Frame for GroupBy](./firstlineindent-element-for-frame-for-groupby-format.md)

[Frame Element for CustomItem for GroupBy](./frame-element-for-customitem-for-groupby-format.md)

[Writing a PowerShell Formatting File](./writing-a-powershell-formatting-file.md)
