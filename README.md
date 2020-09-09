<h1>Introduction</h1>

Azucar is a multi-threaded plugin-based tool to help assess the security of a Azure Cloud environment subscription. By leveraging the Azure API, Azucar automatically gathers a variety of configuration data and analyses all the data relating to a particular subscription in order to determine security risks.

The script <b>will not change or modify</b> any asset deployed in the Azure subscription.

<h1>Operating System Support</h1>

As the script uses the .NET ADAL library for authenticating the user and calling the REST APIs, only Windows OS are currently supported.  

<h1>Features</h1>

* Return a number of attributes on computers, users, groups, contacts, events, etc... from an Azure Active Directory
* Search for High level accounts in Azure Tenant, including Azure Active Directory, classic administrators and Directory Roles (RBAC)
* Multi-Threading support
* Plugin Support
* The following assets are supported by Azucar:
    * Azure SQL Databases, including MySQL and PostgreSQL databases
	* Azure Active Directory
	* Storage Accounts
	* Classic Virtual Machines
	* Virtual Machines V2
	* Security Status
	* Security Policies
	* Role Assignments (RBAC)
	* Missing Security Patches
	* Missing Security Baseline
	* Web Application Firewall
	* Network Security Groups
	* Classic Endpoints
	* Azure Security Alerts
	* Azure KeyVault

<h1>Screenshots</h1>

![azucar](https://user-images.githubusercontent.com/5271640/38782164-3edde5ca-40ef-11e8-94e3-b8f005db139d.PNG)

<h1>Reporting</h1>

Support for exporting data driven to several formats like CSV, XML or JSON.

The following screenshot shows an example report in JSON format

![threat](https://user-images.githubusercontent.com/5271640/38782058-4779800a-40ee-11e8-8bf5-9b16500e5134.PNG)

<h1>Office Support</h1>

Although there is already support for a variety of file formats (CSV, XML or JSON), there is also support for exporting data driven to EXCEL format. Currently, it supports style modification, chart creation, company logo or independent language support. At the moment Office Excel 2010/2013/2016 are supported by the tool.

![excel](https://user-images.githubusercontent.com/5271640/38782057-476050c6-40ee-11e8-9935-3c15f5356980.png)

<h1>Sample reports</h1>

An example of report generated by Azucar can be downloaded from [Azucar_Report_20170308.xlsx](https://github.com/nccgroup/azucar/files/1915480/Azucar_Report_20170308.xlsx)

<h1>Prerequisites</h1>

AZUCAR works out of the box with PowerShell version 3.x and .NET 4.5. You can check your Windows PowerShell version executing the command <b>$PsVersionTable:</b>

```powershell
PS C:\Users\silverhack> $psversiontable

Name                           Value
----                           -----
PSVersion                      5.1.14393.693
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.14393.693
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
```

You should use an account with at least read-permission on the assets you want to access. You could find more information about Role-Based Access Control in Azure by clicking [here](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-portal)

<h1>Installation</h1>

You can download the latest zip by clicking [here](https://github.com/nccgroup/azucar/archive/master.zip).

Preferably, you can download AZUCAR by cloning the [repository](https://github.com/nccgroup/azucar.git):

<pre>
git clone https://github.com/nccgroup/azucar.git
</pre>

Before to start, you need to unblock files. Once you have unzipped the zip file, you can use the fantastic PowerShell V3 Unblock-File cmdlet that will do this task for you:

<pre>
Get-ChildItem -Recurse c:\Azucar_V10 | Unblock-File
</pre>

<h1>Write your own plugin</h1>

The plugin mechanism introduced in Azucar provides an easy method for PowerShell developers to dynamically add functionality, so if you want to extend the functionality of Azucar, you can do so by writing your own plugin in PowerShell. 

To create a custom plugin, add it to the Plugins\Custom folder. The plugin code is simple. A script plugin is essentially any valid PowerShell script saved in a .ps1 extension. Each is a self-contained PowerShell that will be passed as a scriptblock class. The variable names and return values are the same throughout all plugins, so they can be generically loaded. The following sample shows a basic structure of an Azucar PowerShell plugin:

```powershell
#Sample skeleton PowerShell plugin code
[cmdletbinding()]
    Param (
            [Parameter(HelpMessage="Background Runspace ID")]
            [int]
            $bgRunspaceID,

            [Parameter(HelpMessage="Not used in this version")]
            [HashTable]
            $SyncServer,

            [Parameter(HelpMessage="Azure Object with valuable data")]
            [Object]
            $AzureObject,

            [Parameter(HelpMessage="Object to return data")]
            [Object]
            $ReturnPluginObject,

            [Parameter(HelpMessage="Verbosity Options")]
            [System.Collections.Hashtable]
            $Verbosity,

            [Parameter(Mandatory=$false, HelpMessage="Save message in log file")]
	        [Bool] $WriteLog

        )
    Begin{
        #Import Azure API
        $LocalPath = $AzureObject.LocalPath
        $API = $AzureObject.AzureAPI
        $Utils = $AzureObject.Utils
        . $API
        . $Utils

        #Import Localized data
        $LocalizedDataParams = $AzureObject.LocalizedDataParams
        Import-LocalizedData @LocalizedDataParams;
    }
    Process{
        #Do things here
        $ReturnValue = [PSCustomObject]@{Name='myCustomType';Expression={"NCCGroup Labs"}}
		
    }
    End{
        if($ReturnValue){
            #Work with SyncHash
            $SyncServer.$($PluginName)=$ReturnValue
            $ReturnValue.PSObject.TypeNames.Insert(0,'AzureRM.NCCGroup.myDecoratedObject')
            #Create custom object for store data
            $MyVar = New-Object -TypeName PSCustomObject
            $MyVar | Add-Member -type NoteProperty -name Section -value $Section
            $MyVar | Add-Member -type NoteProperty -name Data -value $ReturnValue
            #Add data to object
            if($MyVar){
                $ReturnPluginObject | Add-Member -type NoteProperty -name Example -value $MyVar
            }
        }
        else{
            Write-AzucarMessage -WriteLog $WriteLog -Message ($message.AzureADGeneralQueryEmptyMessage -f "My Super Plugin", $AzureObject.TenantID) `
                                -Plugin $PluginName -Verbosity $Verbosity -IsWarning
        }
    }
```
Once you have your plugin prepared and located into the Plugins\Custom directory, your plugin should be ready to be loaded by using the -Custom flag, as shown below:

<p align="center">
	<img src="https://user-images.githubusercontent.com/5271640/38782034-f56d4882-40ed-11e8-8b37-2b2ae1b3bcb2.png">
</p>

To help you getting started I created various plugins within the Plugins\Custom folder which you can use to get your plugin started.

<h1>Usage</h1>

To get a list of basic options and switches use:

```powershell
get-help .\azucar.ps1
```

To get a list of examples use:

```powershell
get-help .\azucar.ps1 -Examples
```

To get a list of all options and examples with detailed info use:

```powershell
get-help .\azucar.ps1 -Detailed
```

<h1>Examples</h1>

This example retrieves information of an Azure Tenant and print results. The script will try to connect using the ADAL library, and if no credential passed, the script will try to connect using the bearer token for logged user

```powershell
.\Azucar.ps1 -ExportTo PRINT | Format-List
```

This example will retrieve information of an Azure Tenant and print results to a local variable. The script will try to connect using the ADAL library, and if no credential passed, the script will try to connect using the bearer token for logged user

```powershell
$data = .\Azucar.ps1 -AuthMode UseCachedCredentials -Verbose -WriteLog -Debug -ExportTo PRINT
```

This example will retrieve information of an Azure Tenant and print results to a local variable. The script will try to connect by using the ADAL library and will try to connect by using a cached credential

```powershell
$data = .\Azucar.ps1 -AuthMode Client_Credentials -Verbose -WriteLog -Debug -ExportTo PRINT
```
This example will retrieve information of an Azure Tenant and print results to a local variable. The script will try to connect by using the ADAL library and will try to connect by using the client credential flow

```powershell
.\Azucar.ps1 -ExportTo CSV,JSON,XML,EXCEL -AuthMode Certificate_Credentials -Certificate C:\AzucarTest\server.pfx -ApplicationId 00000000-0000-0000-0000-000000000000 -TenantID 00000000-0000-0000-0000-000000000000
```
This example will retrieve information of an Azure Tenant and export data driven to CSV, JSON, XML and Excel format into Reports folder. The script will try to connect by using the Azure Active Directory Application Certificate credential flow

```powershell
.\Azucar.ps1 -ExportTo CSV,JSON,XML,EXCEL -AuthMode Certificate_Credentials -Certificate C:\AzucarTest\server.pfx -CertFilePassword MySuperP@ssw0rd! -ApplicationId 00000000-0000-0000-0000-000000000000 -TenantID 00000000-0000-0000-0000-000000000000
```
This example will retrieve information of an Azure Tenant and export data driven to CSV, JSON, XML and Excel format into Reports folder. The script will try to connect by using the Azure Active Directory Application Certificate credential flow

```powershell
.\Azucar.ps1 -ResolveTenantDomainName microsoft.com
```
This example will try to resolve the TenantID for an specific domain name

```powershell
.\Azucar.ps1 -ResolveTenantUserName user@company.com
```
This example will try to resolve the TenantID for an specific username

