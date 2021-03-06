# Introduction

Going from manual deployment to continuous deployment can be a hard path. One of the toughest challenge is to build increasingly complex virtual environments to simulate common production scenarios. This small framework builds a complete lab environment.

# AutomatedLab

At the heart of this solution is AutomatedLab. AutomatedLab is a provisioning solution to deploy complex labs on HyperV and Azure maintained mostly by a couple of Microsoft Engineers. Its a lot of fun to study if you want to get better at PowerShell and DevOps on the Windows world. The idea is that you declare your environment using a small set of cmdlets that let you set the virtual network, add virtual machines and so one. AutomatedLab has built in validations that make sure you enter a configuration that is within expected ranges and is able to do the provisioning for you. It will create the HyperV objects, install the operating systems and can configure some of Windows Server roles. The presented scenario uses the Domain Controller role.

You can find on github in the [AutomatedLab](https://github.com/AutomatedLab/AutomatedLab) repo. 

# Training labs automation

Training Labs Automation builds on top of Automated Lab by adding json configuration files that you can tweak to configure certain aspects of the lab configurations that coded in the framework. You can set things like the administration account and password for the machines, the domain that will be created and the network address spaces. 

# Creating a lab step-by-step

You start in a Windows machine with HyperV installed where you have admin rights, you will need to enter most of the commands in an elevated command prompt and be advised that AutomatedLab will relax some security settings to make things easier. This system is not intended to be used in production environments where relaxing these settings would make the machines more exposed to malicious attacks.

## Install Automated Lab

Start by installing the AutomatedLab framework. The AllowCluber flag is required because this module needs to enhance some builtin cmdlets to redirect verbose and debug messages into a test log. 

You also need to install SqlServerDsc. These modules are used by the scripts to prepare SQL Server installations.

``` PowerShell
Install-Module AutomatedLab -AllowClobber
Install-Module SqlServerDsc
Install-Module -Name xWebAdministration
```

Because these modules are coming from PSGalery the system will prompt you if you want to proceed:

"
You are installing the modules from an untrusted repository. If you trust this repository, change its
InstallationPolicy value by running the Set-PSRepository cmdlet. Are you sure you want to install the modules from
'PSGallery'?
"

It should be ok to answer Yes here.

**NOTE:** AutomatedLab will copy DSC resources to the guest machines automatically, if you want to tweak the scripts and use other resources you only need to install them at your authoring machine.

## Create the LocalSources folder

The LabSources folder will be created at the root of one of your hard disks and contains the tools that AutomatedLab requires, folders where you have to place Windows ISO files and folders where you have to place the applications you want to install in the virtual machines.

To create it use the following cmdlet, remember to choose the drive letter where you want to store those items, depending on how many variations you will have this might take a lot of space:

``` PowerShell
Import-Module AutomatedLab
New-LabSourcesFolder -DriveLetter C
```

## Download installation ISO files:

If you have an MSDN subscription like we do you can go to [my.visualstudio.com](http://my.visualstudio.com) and download the files for the Windows versions you want to install. For this example you will need at least a server edition and a desktop edition for Windows and one edition of SQL Server.

If you don't have an MSDN subscription or you want to share the machines with someone else (because you can't redistribute Windows copies using your MSDN downloaded software) you can get 180 days trial versions on [Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019).

Then you need to place your ISO files in the LabSources folder (the drive letter should be the one you used in the previous step):
```
explorer C:\LabSources\ISOs
```

## Download dependencies:

- Google Chrome Offline Installer available [here](https://www.google.com/intl/en/chrome/?standalone=1).

For the sake of simplicy the system expects to find them on the same folder as the ISOs:

```
explorer C:\LabSources\ISOs
```


## Get the training-labs-automation scripts

Clone the lab repo:

Configure the framework:

``` PowerShell
cd training-labs-automation
copy settings.template.json settings.user.json
```

Edit the settings.user.json file you just created based on the provided template. Here goes one example for your reference:

``` json
{
    "username" : "Administrator",
    "password" : "pa$$w0rd",
    "domain" : "mylab.local",
    "shortDomain" : "mylab",    
    "headlessWindowsServerOperatingSystem" : "Windows Server 2016 Datacenter",
    "serverWindowsOperatingSystem": "Windows Server 2016 Datacenter (Desktop Experience)",
    "clientWindowsOperatingSystem": "Windows 10 Enterprise",
    "virtualMachinesFolder": "D:\\Partners",
    "sqlServerIsoFile": "en_sql_server_2017_standard_x64_dvd_11294407.iso",
    "ssmsInstallerFile": "SSMS-Setup-ENU.exe",
    "reportingServicesInstallerFile": "SQLServerReportingServices.exe",
    "sqlUser": "mylabsqluser",
    "sqlPassword": "pa$$w0rd"
}
```

To get the names of the operating systems you can use the following cmdlet:

``` PowerShell
Get-LabAvailableOperatingSystem
```

And you should obtain a listing like this one:
![Operating System List][os-list]

[lab-overview]: ./img/lab-overview.png "Virtual Lab Overview"
[os-list]: ./img/operating-systems-list.png "Operating Systems List"

# Prepare the classroom

Open an elevated command prompt on the ClassLab folder. It is important to be on this folder because it contains a json file with important data that the system will search for in the current folder. 

Examine the labSettings.default.json file. It contains the name of the lab, the address space, the number of machines to create and other useful information. For example labPrefix is used to prefix all hyper-v objects with two letters to allow having multiple instances of the lab. If you want to change this file the recommend approach is to create a copy and name it **labSettings.user.json**.

Finally run:

``` PowerShell
. .\create_lab.ps1
```

Notice that we are dot sourcing the script (the first dot on the command). This is really important and the script will not work if you forget that little dot. The scripts need to retain data on the powershell session and the way its done only works if the script is dot sourced.

