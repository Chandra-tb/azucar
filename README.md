<h1>Introduction</h1>

Azucar is a multi-threaded plugin-based tool to help assess the security of your Azure Cloud environment subscription. By leveraging the Azure API, Azucar automatically gathers and analyses a variety of configuration data related to your particular subscription to determine security risks.

The script <b>will not change or modify</b> any asset deployed in the Azure subscription.

<h1>Operating System Support</h1>

As the script uses the .NET ADAL library for authenticating a user and calling REST APIs, it only supports Windows OS.  

<h1>Features</h1>

* Return a number of attributes on computers, users, groups, contacts, events, etc... from Azure Active Directory.
* Search for High Level accounts in a specific Azure Tenant, including Azure Active Directory, classic administrators, and Directory Roles (RBAC).
* Multi-Threading support.
* Plugin Support.
* The following assets are supported by Azucar:

    	- Azure SQL Databases, including MySQL and PostgreSQL databases,
	- Azure Active Directory,
	* Storage Accounts,
	* Classic Virtual Machines,
	* Virtual Machines V2,
	* Security Status,
	* Security Policies,
	* Role Assignments (RBAC),
	* Missing Security Patches,
	* Missing Security Baseline,
	* Web Application Firewall,
	* Network Security Groups,
	* Classic Endpointsy,
	* Azure Security Alerts,
	* Azure KeyVault.

<h1>Screenshots</h1>

![azucar](https://user-images.githubusercontent.com/5271640/38782164-3edde5ca-40ef-11e8-94e3-b8f005db139d.PNG)

<h1>Reporting</h1>

Support for exporting data driven to several formats like CSV, XML or JSON.

The following screenshot shows an example report in JSON format:

![threat](https://user-images.githubusercontent.com/5271640/38782058-4779800a-40ee-11e8-8bf5-9b16500e5134.PNG)

<h1>Office Support</h1>

Although there is already support for a variety of file formats (CSV, XML or JSON), you can also export your data to Excel 2010/2013/2016. At the time of writing, the tool supports style modification, chart creation, company logo or independent language support.

![excel](https://user-images.githubusercontent.com/5271640/38782057-476050c6-40ee-11e8-9935-3c15f5356980.png)

<h1>Sample reports</h1>

An example of report generated by Azucar can be downloaded from [Azucar_Report_20170308.xlsx](https://github.com/nccgroup/azucar/files/1915480/Azucar_Report_20170308.xlsx).

<h1>Prerequisites</h1>

Azucar works out of the box with PowerShell version 3.x and .NET 4.5. You can check your Windows PowerShell version executing the command <b>$PsVersionTable:</b>

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

You should use an account with at least **read-permission** on the assets you want to access. You could find more information about Role-Based Access Control in Azure by clicking [here](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-portal).

<h1>Installation</h1>

You can download the latest zip by clicking [this link](https://github.com/nccgroup/azucar/archive/master.zip).

Preferably, you can download Azucar by cloning the [repository](https://github.com/nccgroup/azucar.git):

<pre>
git clone https://github.com/nccgroup/azucar.git
</pre>

Once you have unzipped the zip file, you can use the PowerShell V3 Unblock-File cmdlet to unblock files:

<pre>
Get-ChildItem -Recurse c:\Azucar_V10 | Unblock-File
</pre>

<h1>Write your own plugin</h1>

The plugin mechanism introduced in Azucar provides an easy method for PowerShell developers to dynamically add new functionality, so if you want to extend the functionality of Azucar, you can do so by writing your own plugin in PowerShell. 

**To use a custom plugin**, add it to the Plugins\Custom folder. For those not familiar with plugin code, a plugin is essentially any valid PowerShell script saved in a .ps1 extension. Each is self-contained PowerShell code that will be passed as a scriptblock class. The variable names and return values are the same throughout all plugins, so they can be generically loaded. 

The following sample shows a basic structure of an Azucar PowerShell plugin:

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
Once you have your plugin prepared and located into the Plugins\Custom directory, it should be ready to be loaded by using the -Custom flag as shown below:

<p align="center">
	<img src="https://user-images.githubusercontent.com/5271640/38782034-f56d4882-40ed-11e8-8b37-2b2ae1b3bcb2.png">
</p>

To help you getting started, I created various plugins within the Plugins\Custom folder which you can use to get your plugin started.

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

<h1>Remarks</h1>

Due to the amount of work we will not always be able to respond quickly to new issues. But rest assured, you will eventually get a response and if needed a fix. 
