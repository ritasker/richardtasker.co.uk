Published: 2013-07-28
Title: Setting Up a New PC with Chocolatey
Lead: I have written a powershell script which will install some of the applications I use a lot via the Chocolatey Nuget Gallery.
Author: richtasker
Tags:
  - Chocolatey
  - PowerShell
---
I am starting a new role at [HH Global](http://www.hhglobal.com/) tomorrow. 
I'll be spending most of tomorrow setting up my pc and installing all the programs and applications I want to use. I decided to create a powershell script which will install Chocolatey, then install a bunch of applications I want from their app gallery.

If you haven't heard of [Chocolatey](http://chocolatey.org/) yet it is very similar to linux's apt-get, it uses powershell scripts and Nuget packages to automate the installation of Windows applications.

The script dosen't do anything out of the ordinary, it literally installs Chocolatey, then executes the `cinst` command for every application I want to install.

	# Install Chocolatey

	"Installing Chocolatey..."
	iex ((new-object net.webclient).DownloadString("http://bit.ly/psChocInstall"))
	"Installing Chocolatey - Done..."

	# Install Programes

	# System Tools

	"Installing System Tools..."

	cinst 7zip
	cinst notepadplusplus
	cinst GoogleChrome
	cinst Firefox
	cinst filezilla
	cinst skype
	cinst FoxitReader
	cinst gimp
	cinst keepass
	cinst truecrypt
	cinst CutePDF
	cinst mirc
	cinst putty
	cinst Cygwin
	cinst virtualbox

	"Installing System Tools - Done..."

	# Dev Tools

	"Installing Dev Tools..."

	cinst fiddler
	cinst linqpad4
	cinst ScriptCs
	cinst CoffeeScript
	cinst NewRelic  
	cinst P4Merge
	cinst poshgit

	"Installing Dev Tools - Done..."

You can also find the script on my GitHub [repo](https://github.com/ritasker/Chocolatey-Install-Enviroment).
