# ubuntu-vscode
How long does it take to set up your test automation development environment? How long does it take a new automatin engineer on your team to set up their dev environment? Say you want to try out a new programming language, how much time do you spend figuring out what tools to use before you actually try out the language? 

## Requirements for ubuntu-vscode
- Be able to `git clone` a project
- Type a single command and be able to edit and debug it
- The IDE tailored to the language and domain of the project

In this README, we will illustrate how to simplify the development experience using recent releases from Ubuntu, Visual Studio Code and Docker with the age-old X Window forwarding.

# Preprequisite Software
This section outlines the prerequisite software you need installed on your local system to get this all running.
- Docker
- Git
- [X Windows Server](https://sourceforge.net/projects/vcxsrv/) (see details below)
- (Optional) [Chocolatey Windows package manager](https://chocolatey.org/)

## Docker
Grab it [here](https://hub.docker.com/editions/community/docker-ce-desktop-windows)

## Git
Grab it [here](https://git-scm.com/download/win)

## X Windows Server on Windows
[VcXsrv](https://sourceforge.net/projects/vcxsrv/) looks like a good choice for an X Server on Windows. It is free, maintained, has good ratings, automatable, and easy to install via Chocolatey: `choco install -y vcxsrv`. 

That installed the most recent 64-bit version vcxsrv v1.20.1.4  released in Jan 2019 on my machine.

## Chocolatey Windows Package Manager Installation
`@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"`

For additional details [see...](https://chocolatey.org/install)

# How to use it?  Start the Container and Get VSCode Running
- Ensure all the required software is installed (all steps outlined above)
- Git clone this repo
- Execute the powershell script [run-docker-for-windows.ps1](ubuntu-vscode/run-dockerforwindows.ps1), if things work out the first try, this should cause a VS code window to be opened on your local system
- Start working with VS code.  Typically this entails pulling a project from Git or starting a project from skratch.

# Approach
All of the this is automatable, but let's walk through how it works. We are going to start Visual Studio Code within the Docker container. VSCode will not start up as root, so I created a vscode user on the image. We will switch to that user `su - vscode` and start VSCode in the current directory of /home/vscode `code -w .`. We will run the latest build of image oviney/ubuntu-vscode that I built. We will send the VSCode window to the X Window Server over a TCP/IP connection on default display 0 (tcp port 6000). To do that, we set the DISPLAY environment variable in the container to to the IP address provided by the Docker host. Run ipconfig and look for "vEthernet (DockerNAT)".  Note:  This might be different on your environment, by default mine was : "vEthernet (Default Switch) 4".

Using that IP address, we can set the env var and start VSCode. `docker run` will pull (download) the image if it doesn't exist locally. When these commands complete, VSCode will pop open, assuming you have X Server running already.

To illustrate how we can automate these repeatable steps, below you will find the powershell script source code.

## Start VcXsrv if it is not started yet

`$prog="$env:ProgramFiles\VcXsrv\vcxsrv.exe"`

`if (! (ps | ? {$_.path -eq $prog})) {& $prog -multiwindow -ac}`

## Get the IP address used by Docker for Windows

`$ip = Get-NetIPAddress `

    `| where {$_.InterfaceAlias -eq 'vEthernet (DockerNAT)' -and $_.AddressFamily -eq 'IPv4'} `
    
    `| select -ExpandProperty IPAddress`

## Start Visual Studo Code as the vscode user

`$cmd="export DISPLAY=${ip}:0; code -w ."`

`docker run --rm `

    `--security-opt seccomp=unconfined `
    
    `oviney/ubuntu-vscode `
    
    `su - vscode -c $cmd`
