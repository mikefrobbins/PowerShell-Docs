---
description: This article shows how to use WMI classes in PowerShell to manage network configuration setting in Windows.
ms.date: 02/12/2025
title: Performing networking tasks
---
# Performing networking tasks

> This sample only applies to Windows platforms.

Because TCP/IP is the most commonly used network protocol, most low-level network protocol
administration tasks involve TCP/IP. In this section, we use PowerShell and WMI to do these tasks.

## Listing IP addresses for a computer

To get all IP addresses in use on the local computer, use the following command:

```powershell
Get-CimInstance -Class Win32_NetworkAdapterConfiguration -Filter IPEnabled=$true |
    Select-Object -ExpandProperty IPAddress
```

Since the **IPAddress** property of a **Win32_NetworkAdapterConfiguration** object is an array, you
must use the **ExpandProperty** parameter of `Select-Object` to see the entire list of addresses.

```Output
10.0.0.1
fe80::60ea:29a7:a233:7cb7
2601:600:a27f:a470:f532:6451:5630:ec8b
2601:600:a27f:a470:e167:477d:6c5c:342d
2601:600:a27f:a470:b021:7f0d:eab9:6299
2601:600:a27f:a470:a40e:ebce:1a8c:a2f3
2601:600:a27f:a470:613c:12a2:e0e0:bd89
2601:600:a27f:a470:444f:17ec:b463:7edd
2601:600:a27f:a470:10fd:7063:28e9:c9f3
2601:600:a27f:a470:60ea:29a7:a233:7cb7
2601:600:a27f:a470::2ec1
```

Using the `Get-Member` cmdlet, you can see that the **IPAddress** property is an array:

```powershell
Get-CimInstance -Class Win32_NetworkAdapterConfiguration -Filter IPEnabled=$true |
    Get-Member -Name IPAddress
```

```Output
   TypeName: Microsoft.Management.Infrastructure.CimInstance#root/cimv2/Win32_NetworkAdapterConfiguration

Name      MemberType Definition
----      ---------- ----------
IPAddress Property   string[] IPAddress {get;}
```

The IPAddress property for each network adapter is actually an array. The braces in the definition
indicate that **IPAddress** isn't a **System.String** value, but an array of **System.String**
values.

## Listing IP configuration data

To display detailed IP configuration data for each network adapter, use the following command:

```powershell
Get-CimInstance -Class Win32_NetworkAdapterConfiguration -Filter IPEnabled=$true
```

The default display for the network adapter configuration object is a very reduced set of the
available information. For in-depth inspection and troubleshooting, use `Select-Object` or a
formatting cmdlet, such as `Format-List`, to specify the properties to be displayed.

In modern TCP/IP networks you are probably not interested in IPX or WINS properties. You can use the
**ExcludeProperty** parameter of `Select-Object` to hide properties with names that begin with
"WINS" or "IPX".

```powershell
Get-CimInstance -Class Win32_NetworkAdapterConfiguration -Filter IPEnabled=$true |
    Select-Object -ExcludeProperty IPX*,WINS*
```

This command returns detailed information about DHCP, DNS, routing, and other minor IP configuration
properties.

## Pinging computers

You can perform a simple ping against a computer by using **Win32_PingStatus**. The following
command performs the ping, but returns lengthy output:

```powershell
Get-CimInstance -Class Win32_PingStatus -Filter "Address='127.0.0.1'"
```

The response from **Win32_PingStatus** contains 29 properties. You can use `Format-Table` to select
the properties that are most interesting to you. The **AutoSize** parameter of `Format-Table`
resizes the table columns so that they display properly in PowerShell.

```powershell
Get-CimInstance -Class Win32_PingStatus -Filter "Address='127.0.0.1'" |
    Format-Table -Property Address,ResponseTime,StatusCode -AutoSize
```

```Output
Address   ResponseTime StatusCode
-------   ------------ ----------
127.0.0.1            0          0
```

A StatusCode of 0 indicates a successful ping.

You can use an array to ping multiple computers with a single command. Because there is more than
one address, use the `ForEach-Object` to ping each address separately:

```powershell
'127.0.0.1','localhost','bing.com' |
  ForEach-Object -Process {
    Get-CimInstance -Class Win32_PingStatus -Filter ("Address='$_'") |
      Select-Object -Property Address,ResponseTime,StatusCode
  }
```

You can use the same command format to ping all the addresses on a subnet, such as a private network
that uses network number 192.168.1.0 and a standard Class C subnet mask (255.255.255.0)., Only
addresses in the range of 192.168.1.1 through 192.168.1.254 are legitimate local addresses (0 is
always reserved for the network number and 255 is a subnet broadcast address).

To represent an array of the numbers from 1 through 254 in PowerShell, use the expression `1..254`.
A complete subnet ping can be performed by adding each value in the range to a partial address in
the ping statement:

```powershell
1..254| ForEach-Object -Process {
  Get-CimInstance -Class Win32_PingStatus -Filter ("Address='192.168.1.$_'") } |
    Select-Object -Property Address,ResponseTime,StatusCode
```

Note that this technique for generating a range of addresses can be used elsewhere as well. You can
generate a complete set of addresses in this way:

```powershell
$ips = 1..254 | ForEach-Object -Process {'192.168.1.' + $_}
```

## Retrieving network adapter properties

Earlier, we mentioned that you could retrieve general configuration properties using the
**Win32_NetworkAdapterConfiguration** class. Although not strictly TCP/IP information, network
adapter information such as MAC addresses and adapter types can be useful for understanding what's
going on with a computer. To get a summary of this information, use the following command:

```powershell
Get-CimInstance -Class Win32_NetworkAdapter -ComputerName .
```

## Assigning the DNS domain for a network adapter

To assign the DNS domain for automatic name resolution, use the **SetDNSDomain** method of the
**Win32_NetworkAdapterConfiguration**. The **Query** parameter of `Invoke-CimMethod` takes a WQL
query string. The cmdlet calls the method specified on each instance returned by the query.

```powershell
$wql = 'SELECT * FROM Win32_NetworkAdapterConfiguration WHERE IPEnabled=True'
$args = @{ DnsDomain = 'fabrikam.com'}
Invoke-CimMethod -MethodName SetDNSDomain -Arguments $args -Query $wql
```

Filtering on `IPEnabled=True` is necessary, because even on a network that uses only TCP/IP, several
of the network adapter configurations on a computer aren't true TCP/IP adapters. they're general
software elements supporting RAS, VPN, QoS, and other services for all adapters and thus don't have
an address of their own.

## Performing DHCP configuration tasks

Modifying DHCP details involves working with a set of network adapters, just as the DNS
configuration does. There are several distinct actions you can perform using WMI.

### Finding DHCP-enabled adapters

To find the DHCP-enabled adapters on a computer, use the following command:

```powershell
Get-CimInstance -Class Win32_NetworkAdapterConfiguration -Filter "DHCPEnabled=$true"
```

To exclude adapters with IP configuration problems, you can retrieve only IP-enabled adapters:

```powershell
Get-CimInstance -Class Win32_NetworkAdapterConfiguration -Filter "IPEnabled=$true and DHCPEnabled=$true"
```

### Retrieving DHCP properties

Because DHCP-related properties for an adapter generally begin with `DHCP`, you can use the Property
parameter of `Format-Table` to display only those properties:

```powershell
Get-CimInstance -Class Win32_NetworkAdapterConfiguration -Filter  "IPEnabled=$true and DHCPEnabled=$true" |
  Format-Table -Property DHCP*
```

### Enabling DHCP on each adapter

To enable DHCP on all adapters, use the following command:

```powershell
$wql = 'SELECT * from Win32_NetworkAdapterConfiguration WHERE IPEnabled=True and DHCPEnabled=False'
Invoke-CimMethod -MethodName ReleaseDHCPLease -Query $wql
```

Using the filter statement `IPEnabled=True and DHCPEnabled=False` avoids enabling DHCP where it's
already enabled.

### Releasing and renewing DHCP leases on specific adapters

Instances of the **Win32_NetworkAdapterConfiguration** class has `ReleaseDHCPLease` and
`RenewDHCPLease` methods. Both are used in the same way. In general, use these methods if you only
need to release or renew addresses for an adapter on a specific subnet. The easiest way to filter
adapters on a subnet is to choose only the adapter configurations that use the gateway for that
subnet. For example, the following command releases all DHCP leases on adapters on the local
computer that are obtaining DHCP leases from 192.168.1.254:

```powershell
$wql = 'SELECT * from Win32_NetworkAdapterConfiguration WHERE DHCPServer="192.168.1.1"'
Invoke-CimMethod -MethodName ReleaseDHCPLease -Query $wql
```

The only change for renewing a DHCP lease is to use the `RenewDHCPLease` method instead of the
`ReleaseDHCPLease` method:

```powershell
$wql = 'SELECT * from Win32_NetworkAdapterConfiguration WHERE DHCPServer="192.168.1.1"'
Invoke-CimMethod -MethodName RenewDHCPLease -Query $wql
```

> [!NOTE]
> When using these methods on a remote computer, be aware that you can lose access to the remote
> system if you are connected to it through the adapter with the released or renewed lease.

### Releasing and renewing DHCP leases on all adapters

You can perform global DHCP address releases or renewals on all adapters by using the
**Win32_NetworkAdapterConfiguration** methods, `ReleaseDHCPLeaseAll` and `RenewDHCPLeaseAll`.
However, the command must apply to the WMI class, rather than a particular adapter, because
releasing and renewing leases globally is performed on the class, not on a specific adapter. The
`Invoke-CimMethod` cmdlet can call the methods of a class.

```powershell
Invoke-CimMethod -ClassName Win32_NetworkAdapterConfiguration -MethodName ReleaseDHCPLeaseAll
```

You can use the same command format to invoke the **RenewDHCPLeaseAll** method:

```powershell
Invoke-CimMethod -ClassName Win32_NetworkAdapterConfiguration -MethodName RenewDHCPLeaseAll
```

## Creating a network share

To create a network share, use the `Create` method of **Win32_Share**:

```powershell
Invoke-CimMethod -ClassName Win32_Share -MethodName Create -Arguments @{
    Path = 'C:\temp'
    Name = 'TempShare'
    Type = [uint32]0 #Disk Drive
    MaximumAllowed = [uint32]25
    Description = 'test share of the temp folder'
}
```

This is equivalent to the following `net share` command on Windows:

```powershell
net share tempshare=C:\temp /users:25 /remark:"test share of the temp folder"
```

To call a method of a WMI class that takes parameters you must know what parameters are available
and the types of those parameters. For example, you can list the methods of the **Win32_Class** with
the following commands:

```powershell
(Get-CimClass -ClassName Win32_Share).CimClassMethods
```

```Output
Name          ReturnType Parameters                                   Qualifiers
----          ---------- ----------                                   ----------
Create            UInt32 {Access, Description, MaximumAllowed, Name…} {Constructor, Implemented, MappingStrings, Stati…
SetShareInfo      UInt32 {Access, Description, MaximumAllowed}        {Implemented, MappingStrings}
GetAccessMask     UInt32 {}                                           {Implemented, MappingStrings}
Delete            UInt32 {}                                           {Destructor, Implemented, MappingStrings}
```

Use the following command to list the parameters of the `Create` method.

```powershell
(Get-CimClass -ClassName Win32_Share).CimClassMethods['Create'].Parameters
```

```Output
Name            CimType Qualifiers                                  ReferenceClassName
----            ------- ----------                                  ------------------
Access         Instance {EmbeddedInstance, ID, In, MappingStrings…}
Description      String {ID, In, MappingStrings, Optional}
MaximumAllowed   UInt32 {ID, In, MappingStrings, Optional}
Name             String {ID, In, MappingStrings}
Password         String {ID, In, MappingStrings, Optional}
Path             String {ID, In, MappingStrings}
Type             UInt32 {ID, In, MappingStrings}
```

You can also read the documentation for
[Create][02] method of the
**Win32_Share** class.

## Removing a network share

You can remove a network share with **Win32_Share**, but the process is slightly different from
creating a share, because you need to retrieve the specific instance to be removed, rather than the
**Win32_Share** class. The following example deletes the share **TempShare**:

```powershell
$wql = 'SELECT * from Win32_Share WHERE Name="TempShare"'
Invoke-CimMethod -MethodName Delete -Query $wql
```

## Connecting a Windows-accessible network drive

The `New-PSDrive` cmdlet can create a PowerShell drive that's mapped to a network share.

```powershell
New-PSDrive -Name "X" -PSProvider "FileSystem" -Root "\\Server01\Public"
```

However, drives created this way are only available to PowerShell session where they're created. To
map a drive that's available outside of PowerShell (or to other PowerShell sessions), you must use
the **Persist** parameter.

```powershell
New-PSDrive -Persist -Name "X" -PSProvider "FileSystem" -Root "\\Server01\Public"
```

> [!NOTE]
> Persistently mapped drives may not be available when running in an elevated context. This is the
> default behavior of Windows UAC. For more information, see the following article:
>
> - [Mapped drives aren't available from an elevated prompt when UAC is configured to Prompt for credentials][01]

<!-- link references -->
[01]: /troubleshoot/windows-client/networking/mapped-drives-not-available-from-elevated-command
[02]: /windows/win32/cimwin32prov/create-method-in-class-win32-share
