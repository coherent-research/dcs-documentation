# Introduction
From version 3.0 onwards DCS deployment is done by Powershell scripts. 
The powershell scripts can be downloaded from the Coherent Support Site. 

In addition a Licence file from DCS will be required to run DCS.

# DCS Components
There are 4 components that may be installed as part of the DCS installation.
## DCS DB
A database hosted on a SQL server instance. The database contains user information,
meter information and meter readings.
## DCSWebApp 
An ASP.NET Core Web Application that must be hosted by IIS. 
It provides a browser based user interface for DCS. 
## DcsService
A Windows Service Application which fetches the metered data from meters via IDCs, 
TCP/IP or dialup connections and stores it in a database. 
If dialup connections are to be used the modem(s) must be installed on the 
same machine as DcsService.
## IdcWebServices
An ASP.NET Web Application that must be hosted by IIS. 
It provides a Web Services interface used by IDCs to connect to DCS via HTTP. 

> If an installation does not plan to use IDCs this component does not need to be installed.

# Prerequisites
DCS can be installed on any version of Windows Server from Windows Server 2012 R2 onwards. 
Note that desktop versions of Windows are not supported.

The following features must be installed/enabled with these minimum versions 
before DCS can be installed;

- .NET framework version 4.7.2 (see https://dotnet.microsoft.com/download)
- IIS (for DcsWebApp and IdcWebServices)
- ASP.NET Core/.NET Core Runtime & Hosting Bundle version 2.1 (for DcsWebApp) (see https://dotnet.microsoft.com/download/dotnet-core/2.1)
- Powershell 5.0 for installation

To ensure the following prerequistes are satisfied use the Check-DcsPrerequisites script (see below).

> Ensure that IIS is enabled BEFORE installing ASP.NET Core framework or else DCSWebApp will not work.

The DCS DB also requires access to an instance of SQL Server or SQL Server Express version 2012.

> Installing the prerequisites is not covered in this document.

# Downloading the DCS Installation Scripts
> Note that the user acccount used must have administration authority and the Powershell execution policy must be
> such that the user can run the Powershell scripts.

- Download the latest version of the DCS Installation Scripts
which are avaliable at https://www.coherent-research.co.uk/support/download. 
The scripts will only be available to download for users who have logged in to the Coherent Support website.

> Once downloaded ensure that the file isn't blocked because it has come from another computer. 
> If so make sure it is unblocked before proceeding.

- Create a new directory (e.g. DCS). This directory is used when installing and updating DCS. The actual final location for DCS will be specified during the installation process.
- Unzip the DCS Installation Scripts into the new directory

# Before installation
Use Check-DcsPrerequisites to check the the server is ready to install DCS
This will check that the prerequisites listed in the chapter above are met.
```
Check-Prerequisites
```
Use Check-DcsSettings to create an installation settings file and display the settings.
This will show the current settings from the DcsInstallationSettings.psd1 file which is used 
by all other scripts. Any of the settings in this file can be edited. It is possible to override
these settings for individual scripts with command line parameters.

```
Check-DcsSettings
```

> Initially the DcsInstallationSettings file has the bare minimal settings required to get
> DCS installed and running. It is recommended that these miminal settings are 
> reviewed and updated to fit the organisation's requirements. 
> At a mimimum  SMTP settings should be set as there are no default values for these settings.

# Getting Help
All installation scripts have a **Help** parameter which will display information 
about the script and all parameters.

For example, to get help for the Install-DcsWebApp script:
```
Install-DcsWebApp -Help
```

# Windows accounts
The default settings provided in the **DcsInstallationSettings** file will use the Local System Account
for the following purposes:

- Used by the IIS Application Pool used by DcsWebApp
- Used by the IIS Application Pool used by IdcWebServices
- Used to run DcsService

While using the Local System Account is the simplest way to get DCS up and running,
it is recommended that a special Windows or domain account is created 
and used for this purpose instead. In this case the **DcsAccount** field in the **DcsInstallationSettings**
file should be set to the fully qualified user name (e.g. SERVER1\DcsAccount or OURDOMAIN\DcsAccout). 

Whether the Local System Account or a special account is created it will require 
access to DCS DB and the **Install-DcsDb** script will
create the approriate user login in SQL server and 
provide it with the required **DcsFullAccess** database role to allow this.

> When a special account is used the scripts will securely prompt for the 
> associated password when required. The scripts do not store the password anywhere.

# Installing DCS
## Downloading DCS deployment files
Move to the new DCS installation directory created above and download the 
current version of the DCS installation files using Coherent Support Site credentials:

```
Get-Dcs -Username USERNAME

USERNAME: Coherent Support account username
```
The user will be prompted for a password.

This will download the current version of the file into the current directory with the format:
```
DCSvX.Y.Z.zip
```
The contents of the file will then be unzipped into the subdirectory DCSvX.Y.Z
(in the current directory).

> Alternatively, the file can be downloaded manually from the support site with a browser and unzipped into the DCS directory 
> but it is recommended to use the script.

Each time DCS is updated the latest version of DCS can be fetched in the same way.

## DCS DB
### Installing DCS DB
To install DCS DB using the settings defined in the DcsInstallationSettings file:

```
Install-DcsDb -Version X.Y.Z
```

> The database contains the role DcsFullAccess which must be given to any account used
> to run any DCS component.

## DcsWebApp
### Installing DcsWebApp
To install DcsWebApp using the settings defined in the DcsInstallationSettings file:

``` 
Install-DcsWebApp -Version x.y.z
```
### Post installation configuration
In the directory where DcsWebApp was installed the **appsettings.json** file 
may be edited to fine tune the installation before starting the site.

### Starting DcsWebApp
To start DcsWebApp and ensure that it is running using the settings defined in the DcsInstallationSettings file:

```
Start-DcsWebApp 
```

### Stopping DcsWebApp
To take the DcsWebApp offline:

```
Stop-DcsWebApp 
```

This will display a message to users that try to log on to the DCS website 
that it is currently down for maintenance. This also stops users performing any actions
against the database.

### Unstalling DcsWebApp
This command will uninstall DcsWebApp.
 ```
Uninstall-DcsWebApp
```
> Note that the directory where DcsWebApp was installed and the appSettings.json file will not be removed. 
These can be manually removed if DCS will not be reinstalled.

## DcsService
### Installing DcsService
To install DcsService using the settings defined in the DcsInstallationSettings file:

```
Install-DcsService -Version X.Y.Z
```

> Note that DcsService will be configured to start automatically on system start and
> to restart automatically when an error. 

### Post installation configuration
- In the directory where DcsService was installed the **DcsService.exe.config** file 
may be edited to fine tune the installation before starting the service.
- Install licence by copying the **Licence.xml** file to the DcsService directory.

### Starting DcsService
To start DCService

```
Start-DcsService
```

### Stopping DcsService
To start DcsService

```
Stop-DcsService 
```

### Unstalling DcsService
This command will uninstall DcsService.
 ```
Uninstall-DcsService
```
Note that the directory where DcsService was installed and the **DcsService.exe.config** file will not be removed. 
These can be manually removed if DCS will not be reinstalled.

## IdcWebServices
### Installing IdcWebServices
To install IdcWebServices using the settings defined in the DcsInstallationSettings file:

``` 
Install-IdcWebServices -Version x.y.z
```
### Post installation configuration
In the directory where IdcWebServices was installed the **web.config** file 
may be edited to fine tune the installation before starting the site.

### Starting IdcWebServices
To start IdcWebServices and ensure that it is running using the settings defined in the DcsInstallationSettings file:

```
Start-IdcWebServices
```

### Stopping IdcWebServices
To take the IdcWebServices offline:

```
Stop-IdcWebServices 
```
This stops IDCs from connecting to the server and performing any actions
against the database.

### Unstalling IdcWebServices
This command will uninstall IdcWebServices.
 ```
Uninstall-IdcWebServices
```
Note that the directory where IdcWebServices was installed and the **web.config** file will not be removed. 
These can be manually removed if DCS will not be reinstalled.


# Updating an existing installation
New versions of DCS are released from time to time to address bugs or add new features. 
A set of scripts is provided to enable an installation to be updated with minimal effort.

Before starting the update ensure the latest version of the DCS Deployment Files 
have been downloaded as described above.

> The update scripts will **not** modify existing configuration files for installed components.

## Updating DCS DB
> DCS DB does not need to be updated if only the least significant part of the DCS version has changed.
> E.g. if the currently installed version of DCS is 3.1.1 and the latest version is 3.1.2 
> DCS DB does NOT need to be updated, however, if the latest version is 3.2.0 then DCS DB does 
> need to be updated.

Before updating DCS back up the database. This can be done using SQL Server Management Studio or
using the powershell script provided:
```
Backup-DcsDb
```

Once the database has been backed up, update the database to the current version as follows:
```
Update-DcsDb -Version X.Y.X
```

> If any errors are displayed as part of the update it is recommended to restore the backup
> contact Coherent Support.

## Updating DcsWebApp
To update DcsWebApp:
```
Update-DcsWebApp -version X.Y.Z
```
## Updating DcsService

To update DcsService:
```
Update-DcsService -version X.Y.Z
```
## Updating IdcWebServices
To update IdcWebServices:
```
Update-IdcWebServices -version X.Y.Z
```

# Migrating from v2.x.x
## Preliminaries
> Before proceeding, ensure that the all prerequisites outlined above are met. 
> Use the **Check-Prerequisites** script to ensure this is the case.

A few fundamental things have changed in DCS from version 2 to version 3 
which affect the installation:

- DcsWebApp now requires ASP.NET Core 2.1 and requires the runtime as specified above
- The main Windows service is now called DcsService (it was formally called DcService)
and now requires the .NET Framework 4.7.2 or later as specified above
- IdcWebServices now require the .NET Framework 4.7.2 or later as specified above
- MSIs are no longer used to install/uninstall DCS. 
Instead a number of Powershell scripts are used and therefore Powershell 4.0 
or later is required

> The DCS DB scripts provided as part of DCSv3 will only update a database that is at
> version 2.43.0 or later. If the current installation is at an earlier value then use
> the DCSv2 installation guide to update DCS to the latest 2.x version and then follow 
> the procedure below.

## Uninstall v2
Before proceeding, make a note of the locations where DcService, DcsWebApp and IdcWebServices are currently installed. 

- Uninstall all DCS components from the Windows Control Panel. 
- The uninstall routines will leave the "old" configuration files which will contain information 
required for the new installation.  Keep a copy of these files safe but 
remove them from the directories the new version of the DCS components will be installed into.

## Migrating DCS DB
Ensure that a full backup of the database is performed before proceeding. This case be done using SQL Server Management Studio or by running the script **Backup-DcsDb**:
```
Update-DcsDb -Version X.Y.Z
```

## Migrating DcsService
Ensure that this is installed into a different directory than the v2 version was installed in. This is the case by default.
- Run the **Check-DcsSettings** script to set up the **DcsInstallationSettings** file.
- Edit the **DcsInstallationSettings** file if required. If necessary, use the settings
from the previous settings files to set values.
- Install DcsService as described above using the **Install-DcsService** script.
- Copy the **Licence.xml** file from the v2 installation directory to the new installation directory.
- Start as described above.

## Migrating DcsWebApp
- Install DcsWebApp as described above using the **Install-DcsWebApp** script.
- Start as described above

## Migrate IdcWebServices
- Install DcsWebApp as described above using the **Install-DcsWebApp** script.
- Start as described above



