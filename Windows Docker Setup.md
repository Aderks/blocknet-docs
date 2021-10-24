# Windows Server 2016 Hyper-V Docker Setup

## Requirements:

* 64bit CPU that supports virtualization
* Windows 10 64bit (Pro, Enterprise, Education), Windows Server 2016
* Virtualization enabled in BIOS (typically enabled by default)
* CPU SLAT-capable
* At least 4GB RAM

---

## Installing Docker

* Download Docker: https://www.docker.com/products/docker-desktop

* Follow the install instructions and install Docker on the OS or VM of your choice

**Docker Startup Errors:**

```
Unable to start: The running command stopped because the preference variable "ErrorActionPreference" or common parameter is set to Stop: 'MobyLinuxVM' failed to start.

Failed to start the virtual machine 'MobyLinuxVM' because one of the Hyper-V components is not running.

'MobyLinuxVM' failed to start. (Virtual machine ID BBD755F7-05B6-4933-B1E0-F8ACA3D2467B)     

The Virtual Machine Management Service failed to start the virtual machine 'MobyLinuxVM' because one of the Hyper-V components is not running (Virtual machine ID BBD755F7-05B6-4933-B1E0-F8ACA3D2467B).
at Start-MobyLinuxVM, <No file>: line 315
at <ScriptBlock>, <No file>: line 410
   at Docker.Backend.ContainerEngine.Linux.DoStart(Settings settings, String daemonOptions) in C:\gopath\src\github.com\docker\pinata\win\src\Docker.Backend\ContainerEngine\Linux.cs:line 256
   at Docker.Backend.ContainerEngine.Linux.Start(Settings settings, String daemonOptions) in C:\gopath\src\github.com\docker\pinata\win\src\Docker.Backend\ContainerEngine\Linux.cs:line 130
   at Docker.Core.Pipe.NamedPipeServer.<>c__DisplayClass9_0.<Register>b__0(Object[] parameters) in C:\gopath\src\github.com\docker\pinata\win\src\Docker.Core\pipe\NamedPipeServer.cs:line 47
   at Docker.Core.Pipe.NamedPipeServer.RunAction(String action, Object[] parameters) in C:\gopath\src\github.com\docker\pinata\win\src\Docker.Core\pipe\NamedPipeServer.cs:line 145
```   

**Solution:**

*Open Hyper-V Manager (outside of VM's)*

* Run PowerShell (Run as adminstrator)

* Set the VM Setting:

```
Set-VMProcessor -VMName <Enter-VM-Name> -ExposeVirtualizationExtensions $true -Verbose
```

*Open VM where Docker is installed*

* Run PowerShell (Run as administrator)

* Enable Windows Hyper-V features:

```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All -Verbose
```

* Enable Windows containers feature:

```
Enable-WindowsOptionalFeature -Online -FeatureName Containers -All -Verbose
```

* Enable Hypervisor to auto-start: 

```
bcdedit /set hypervisorlaunchtype Auto
```

* Restart the VM

* If Docker doesn't start automatically, run 'Docker Desktop'

* Docker should now be running properly

---

## Testing Docker is setup properly

* Open command prompt/terminal of choice

* Type `docker version`

* If you get get this error proceed to the next step:

```
docker: error during connect: Post http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.39/containers/create: open //./pipe/docker_engine: The system cannot find the file specified. In the default daemon configuration on Windows, the docker client must be run elevated to connect. This error may also indicate that the docker daemon is not running.
See 'docker run --help'.
```

* Type the following to fix the above error:

```
cd "C:\Program Files\Docker\Docker"
./DockerCli.exe -SwitchDaemon
```

* Side-click the Docker icon in the icon tray and ensure it says "Switch to Windows containers..." but don't click it

* Try `docker version` again, it should be error free

* Pull and run the hello-world container by typing: `docker run hello-world`

* After the image is pulled, installed and the container is deployed you should see:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```