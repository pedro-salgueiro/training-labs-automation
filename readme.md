# About the training lab preparation scripts

This repository contains scripts to setup virtual machine scenarios for Critical Manufacturing Partners Training Program. It is using the [AutomatedLab](https://github.com/AutomatedLab/AutomatedLab) framework as the engine to orchestrate the virtual machines setup.

The scripts are also useful for setting up testing scenarios and as a demonstration of applying Desired State Configuration (DSC) to implement Configuration Management in your organization.

## Installing AutomatedLab

You should start by installing **AutomatedLab** on your machine or Hyper-V server. Please note that AutomatedLab relaxes some security restrictions on the box where it is being installed and you should revise those changes before running the installation procedure on a production server. This framework is intended for automating test scenarios and not as a production server provisioning system. Make sure your hosting machine meets the requirements described in the readme file.

The installation procedure is documented [here](https://github.com/AutomatedLab/AutomatedLab/wiki/1.-Installation). My experience is that the MSI installer does not work properly and I always install it from the powershell gallery.

## Getting started

Before running the scripts on this folder copy the **settings.template.json** file, name it **settings.user.json** and populate the settings inside. The repository is configured to ignore this file to prevent users from accidentally committing sensitive information to github. I also use this file to setup the windows image names to use to create the virtual machines. The names vary if we are using trial versions or regular versions. 