![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-0.png)

Covenant is an open source .NET command and control framework aimed at making .NET tradecraft easier and being a collaborative C2 for red teamers. It is multi-platform as it is written in .NET core which means it can run natively on Windows, MacOS and Linux. It also supports docker in case the user prefers containers. In this blog post, we will look at installing and running Covenant both natively and in a docker container. Then review some basic usage of the framework.

# Foreword
**NOTE 1**: These steps/commands are not unique, they are mostly directly copied and rewritten from the  [wiki](https://github.com/cobbr/Covenant/wiki)

**NOTE 2**: The screenshots in this post, are all taken while running the master GitHub branch, the dev branch is still under active development and changes are being made regularly.

**IMPORTANT** - Be sure to clone Covenant recursively to initialize the git submodules: `git clone --recurse-submodules https://github.com/cobbr/Covenant`

As of the publishing of this blog post, the required Dotnet for version for each branch is as follow:

| master      			| dev 			|
| ----------- 			| ----------- 	|
| dotnet core 3.1 SDK  	| dotnet 5.0 SDK|

There are two ways to use Covenant, Dotnet/Dotnet core or Docker. They are both similar and straight forward to setup. If you are not familiar with Docker or do not want to learn some Docker, then I recommend going the Dotnet/Dotnet core route. 

**NOTE**: For the remainder of this post I am going to refer to either SDK from above as "Dotnet", unless there is a reason for calling out a specific version. 

<br />

# Installation 
## Option 1 - Dotnet Core
You can download Dotnet core 3.1 for your platform from [here](https://dotnet.microsoft.com/download/dotnet-core/3.1) and Dotnet 5.0 for your platform [here](https://dotnet.microsoft.com/download/dotnet/5.0).

Be sure to install the Dotnet core version **3.1 SDK**! (If you are running the dev branch you will need Dotnet version **5.0 SDK**)

### Linux Installation
Microsoft nicely lays out the steps for installing Dotnet on Linux machines and makes it very simple with just a few commands:


* Debian

Install the Microsoft package signing key and package repository:
```
root@debian ~ > wget https://packages.microsoft.com/config/debian/10/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
root@debian ~ > sudo dpkg -i packages-microsoft-prod.deb
```

Then install the Dotnet SDK:
Dotnet 3.1 (Master)
```
root@debian ~ > sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-3.1
```

Dotnet 5.0 (Dev)
```
root@debian ~ > sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-5.0
```


* Ubuntu (20.10)

Install the Microsoft package signing key and package repository:
```
root@ubuntu ~ > wget https://packages.microsoft.com/config/ubuntu/20.10/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
root@debian ~ > sudo dpkg -i packages-microsoft-prod.deb
```

Then install the Dotnet SDK:
Dotnet 3.1 (Master)
```
root@ubuntu ~ > sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-3.1
```

Dotnet 5.0 (Dev)
```
root@ubuntu ~ > sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-5.0
```

Once Dotnet is installed, we can run Covenant using the Dotnet CLI:
```
$ ~ > git clone --recurse-submodules https://github.com/cobbr/Covenant
$ ~ > cd Covenant/Covenant
$ ~/Covenant/Covenant > dotnet run
warn: Microsoft.EntityFrameworkCore.Model.Validation[10400]
      Sensitive data logging is enabled. Log entries and exception messages may include sensitive application data, this mode should only be enabled during development.
WARNING: Running Covenant non-elevated. You may not have permission to start Listeners on low-numbered ports. Consider running Covenant elevated.
Covenant has started! Navigate to https://127.0.0.1:7443 in a browser
```

We now have Covenant running on our local machine and can access the admin UI be navigating to [https://127.0.0.1:7443](https://127.0.0.1:7443).

## Option 2 - Docker
As stated above, if you are not familiar with Docker and do not want to learn some container magic, this is the wrong option for you.

Docker needs to be installed on the system where you intend to run Covenant. The steps outlined [here](https://docs.docker.com/engine/install/) in the Docker documentation can be used to install the engine on your favorite Linux flavor. 

**Note**: Make sure you can install Dotnet on the Linux flavor chosen!

Once Docker is installed, we can clone Covenant and build the docker image:

```
$ ~ > git clone --recurse-submodules https://github.com/cobbr/Covenant
$ ~ > cd Covenant/Covenant
$ ~/Covenant/Covenant > docker build -t covenant .
```

Now, run Covenant within the Docker container (be sure to replace the "</absolute/path/to/Covenant/Covenant/Data>" with your own absolute path!):

```
$ ~/Covenant/Covenant > docker run -it -p 7443:7443 -p 80:80 -p 443:443 --name covenant -v </absolute/path/to/Covenant/Covenant/Data>:/app/Data covenant
```

E.g.: 
```
$ ~/Covenant/Covenant > docker run -it -p 7443:7443 -p 80:80 -p 443:443 --name covenant -v /home/ryan/Covenant/Covenant:/app/Data covenant
```

The `-it` parameter is a Docker parameter that indicates that we should begin Covenant in an interactive tty, and can be excluded if you would not like to attach to the tty.

The `-p` parameters expose ports to the Covenant Docker container. You must expose port 7443 (admin UI) and any other ports you would like to start listeners on.

The `-v` parameter creates a shared Data directory between the host and the container. Be sure to specify an absolute path to your data directory, a relative path will not work.

Once Covenant has been started, you can disconnect from the interactive interface at any time by pressing `Ctrl+p` and `Ctrl+q` consecutively.

To stop the container, you can run:

```
$ ~/Covenant/Covenant > docker stop covenant
```

And to restart Covenant interactively (with all data saved), you can run:

```
$ ~/Covenant/Covenant > docker start covenant -ai
```

Alternatively, to remove all Covenant data and restart fresh, you can remove and run again (again, be sure to replace the "</absolute/path/to/Covenant/Covenant/Data>" with your own absolute path!):

```
$ ~/Covenant/Covenant > docker rm covenant
$ ~/Covenant/Covenant > docker run -it -p 7443:7443 -p 80:80 -p 443:443 --name covenant -v </absolute/path/to/Covenant/Covenant/Data>:/app/Data covenant --username AdminUser --computername 0.0.0.0
```

Docker is awesome when you don't want to build disposable infrastructure and you want to build a long-term C2 server. If you have reoccurring clients, in between engagements you just destroy and rebuild the container for a clean workspace.

## Installation Tips
- You will not have to register a user if one is specified on the docker command-line using the `--username` flag, you will also prompted for a password
- If the docker container crashes, try changing and eliminating optional command line arguments

<br />

# GUI Tasks
By default, the web interface for Covenant is started on [https://127.0.0.1:7443](https://127.0.0.1:7443). This means, if you are running this from a headless Linux/Windows server, you will need to proxy access to this port somehow. SSH can be used to do a local port forward to the C2 server.

```
$ ~/ > ssh -v -N -L 7443:127.0.0.1:7443 <USER>@<COVENANT-IP>
```

**NOTE**: If port 7443 is exposed on the SSH client machine, anyone can access the C2 server. This could be helpful of setting up a local port forward on a proxy or internal jump box for team members. This can be restricted to only allow localhost with the command below:

```
$ ~/ > ssh -v -N -L 127.0.0.1:7443:127.0.0.1:7443 <USER>@<COVENANT-IP>
```

## Registering a user
After starting Covenant and the GUI has been reached, you must register an initial user through the web interface. Navigating to the web interface at [https://127.0.0.1:7443](https://127.0.0.1:7443) will allow you to register the initial user:

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-1.png)

Once the initial user has been registered, open registration will be closed, and new users will have to be created by an Administrative user.

Upon logging into the application you are greeted with your new Covenant dashboard!

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-2.png)

## Creating a HTTP listener
Covenant makes creating a listener super easy. There are two types of listeners included with the framework, bridge and native. Bridge listeners are beyond the scope of this article and can be read about in the documentation [here](https://github.com/cobbr/Covenant/wiki/Bridge-Listeners). Currently, Covenant only has a native `HttpListener`, which is self-explanatory. To set up a listener, click on "Listeners" from the menu and then "Create". 

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-3.png)

We are dropped into the listener configuration screen where we can change the options applied to the listener.

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-4.png)

The documentation has a very clean description of each of the options so I will include it here for reference.

The following options will need to be configured when creating the listener:

-   **Name** - The `Name` of the listener that will be used throughout the interface. Pick something recognizable!
-   **BindAddress** - The `BindAddress` is the local ip address that the listener will bind to. This can be helpful in cases where the Covenant host has multiple nics. Usually, this value will be `0.0.0.0`.
-   **BindPort** - The `BindPort` is the local port that the listener will bind to.
-   **ConnectPort** - The `ConnectPort` is the _callback_ port that Grunts will be directly connecting to. This also represents the port portion of the `Urls`.
-   **ConnectAddresses** - The `ConnectAddresses` are the _callback_ addresses, and represents the hostname portion of the `Url`. You must specify at least one `ConnectAddress` and can specify as many as you would like. You can specify multiple `ConnectAddresses` for failure prevention. Grunts will attempt to connect to each of these `ConnectAddresses` and will use the first one that succeeds. If you are using redirectors, this should be the url that points to the external redirector.
-   **Urls** - The `Urls` are the _callback_ URLs, and are the urls that Grunts will be directly connecting to. The URLs are calculated based on a combination of the `ConnectAddresses`, `BindPort`, and `UseSSL` values, and should be of the form: `http(s)://<connectaddress>:<bindport>`. The `Urls` are calculated for you, you should not need to manually edit these values.
-   **UseSSL** - The `UseSSL` value determines if the listener should use the HTTPS or HTTP protocol. If `UseSSL` value is true, an `SSLCertificate` needs to be provided.
-   **SSLCertificate** - The `SSLCertificate` is the certificate used by the listener, if `UseSSL` is true. The certificate is expected be in PFX format.
-   **SSLCertificatePassword** - The `SSLCertificatePassword` is the password that is being used to protect the `SSLCertificate`.
-   **HttpProfile** - The `HttpProfile` determines the behavior of Grunt and Listener communication.
</end >

**Note**: We will be going over how to create a HTTPS listener in the next section.
	
Once all the required options have been set, clicking on "Create" will create, start the listener and add it to the listener dashboard.

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-5.png)

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-6.png)

## Creating a HTTPS listener
Creating a HTTPS listener is the same as creating the HTTP listener as above with a few small tweaks. The `UseSSL` value will need to be set as true and a `SSLCertificate` needs to be provided in a PFX format. There are a number of different ways to get one of these. Purchase a domain and get a certificate through LetsEncrypt or generate a self-signed certificate. Retrieving a certificate from LetsEncrypt is also beyond the scope of this post and will not be covered here. Google is your friend :).

As far as self signed certifications go, there are 2 options. Covenant is nice enough to generate self-signed certificates for us if they don't exist in the "Covenant Data" directory e.g. `~/Covenant/Covenant/Data`. It will generate `covenant-dev-public.cer` which will be used for the admin HTTPS connection and `covenant-dev-private.pfx` which can be used for HTTPS listener creation. Since these are generated server side and the admin panel isn't exposed to the internet by default (DON'T DO IT), option 1 is to somehow pull the `.pfx` file locally (ex. FTP, SCP, HTTP File Hosting). Option 2 is using a small CSharp program to generate a self-signed certificate locally. I dug into parts of the Covenant source [code](https://github.com/cobbr/Covenant/blob/c53155615563cf68979820356b8430e4eb01207d/Covenant/Covenant.cs#L201) and looked into how Covenant did the self-signed certificate generation. I then extracted the relevant code and wrapped it into a standalone C# project for download [here](https://github.com/active-labs/covenantCertGeneration). The source is small, simple and easy to modify if other attributes need to be added to the generated certificate. Compiling and running the program results in the `covenant-dev-public.cer` and `covenant-dev-private.pfx` files being created in the current working directory:

```
PS C:\source\repos\covenantCert\covenantCert\bin\Release> .\covenantCert.exe
Creating cert...
PS C:\source\repos\covenantCert\covenantCert\bin\Release> dir


    Directory: C:\source\repos\covenantCert\covenantCert\bin\Release


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         5/24/2021   1:13 PM           2414 covenant-dev-private.pfx
-a----         5/24/2021   1:13 PM            738 covenant-dev-public.cer
-a----         5/12/2021   3:53 PM           6656 covenantCert.exe
```

Now we can create a new listener with the same options as above, but with the addition of the `UseSSL` variable being `True`. 

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-7.png)

Now we have the option to upload a `SSlCertificate` and provide the `SSLCertificatePassword`. If you are using the self-signed certificate from Covenant or the standalone CSharp [program](https://github.com/active-labs/covenantCertGeneration), the `SSLCertificatePassword` should be left blank.

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-8.png)

Clicking on "Create" will start our HTTPS listener and now we can generate a launcher that will utilize it!

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-9.png)

## Launchers
"Launchers are used to generate, host, and download binaries, scripts, and one-liners to launch new Grunts." - Covenant [Documentation](https://github.com/cobbr/Covenant/wiki/Launchers)

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-10.png)

There is no reason to rewrite some of the launcher documentation that is provided in the Covenant wiki as it short, sweet and aptly describes the current launcher types so it has been duplicated here.

Launchers are named roughly by the system binary that will be used to execute the launcher. Currently, Covenant supports the following launchers:

-   **Binary** - The `Binary` launcher is used to generate custom binaries that launch a Grunt. This is currently the only launcher that does not rely on a system binary.
-   **ShellCode** - The `ShellCode` launcher converts a Grunt binary to ShellCode using Donut.
-   **PowerShell** - The `PowerShell` launcher is used to generate PowerShell code and/or a PowerShell one-liner that launches a Grunt using `powershell.exe`.
-   **MSBuild** - The `MSBuild` launcher is used to generate an MSBuild XML file that launches a Grunt using `msbuild.exe`.
-   **InstallUtil** - The `InstallUtil` launcher is used to generate an InstallUtil XML file that launches a Grunt using `installutil.exe`.
-   **Mshta** - The `Mshta` launcher is used to generate an HTA file and/or a mshta one-liner that launches a Grunt using `mshta.exe` that relies on DotNetToJScript.
-   **Regsvr32** - The `Regsvr32` launcher is used to generate an SCT file and/or regsvr32 one-liner that launches a Grunt using `regsvr32.exe` that relies on DotNetToJScript.
-   **Wmic** - The `Wmic` launcher is used to generate an xsl file and/or wmic one-liner that launches a Grunt using `wmic.exe` that relies on DotNetToJScript.
-   **Cscript** - The `Cscript` launcher is used to generate a JScript file a Grunt using `cscript.exe` that relies on DotNetToJScript.
-   **Wscript** - The `Wscript` launcher is used to generate a JScript file a Grunt using `wscript.exe` that relies on DotNetToJScript.

Please keep in mind that any of the launchers that rely on DotNetToJScript **may not work** on some of the latest versions of Windows 10 and Windows Server 2016 and/or may be signatured by some AMSI providers.

## Creating a binary launcher
We will use the binary launcher option to create a standalone compiled Dotnet binary as it is straight forward and perfect for an example. Navigating to Launchers > Binary displays a handful of configuration options.

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-11.png)

Other launchers may have some additional configuration options, but these options are common to all launcher types. The configuration options to consider are:

-   **Listener** - The `Listener` is the name of the listener that this Grunt should communicate with. If you have multiple active listeners, be sure to select the correct listener.
-   **ImplantTemplate** - The `ImplantTemplate` is the type of implant that the launcher will generate.
-   **DotNetVersion** - The `DotNetVersion` of the implant that will be generated. You'll be limited to a choice of the `DotNetVersion`s compatible with the chosen `ImplantTemplate`.
-   **Delay** - The `Delay` is the time that the Grunt will sleep in-between each poll of the server. A larger `Delay` value will result in stealthier communication, but increase the time it takes to task a Grunt.
-   **JitterPercent** - The `JitterPercent` is the percentage of variability in the `Delay` value.
-   **ConnectAttempts** - The `ConnectAttempts` is the number of consecutive times a Grunt will attempt to poll the listener before quitting. If a Grunt cannot reach the listener and fails to successfully poll the listener more times than the `ConnectAttempts` value, it will quit.
-   **KillDate** - The `KillDate` is the date at which a Grunt will quit and stop calling back to the listener.

Some other options may be displayed based upon the `ImplantTemplate` that has been selected. If you select an `ImplantTemplate` with an `HTTP` `CommType`:

-   **ValidateCert** - The `ValidateCert` option determines if the Grunt will validate the listener's SSL certificate to prevent MiTM attacks. There are scenarios where target network proxies can interfere with certificate validation, and it's preferable to _not_ validate the certificate. This option is only relevant when using the HTTP `CommType`, and will only be displayed if you have selected the HTTP `CommType`.
-   **UseCertPinning** - The `UseCertPinning` option determines if the Grunt will use cert pinning of the listener's SSL certificate to prevent MiTM attacks. There are scenarios where target network proxies can interfere with certificate pinning, and it's preferable to _not_ perform cert pinning. This option is only relevant when using the HTTP `CommType`, and will only be displayed if you have selected the HTTP `CommType`.

If you select an `ImplantTemplate` with an `SMB` `CommType`:

-   **SMBPipeName** - The `SMBPipeName` is the name of the named pipe that the Grunt will bind to and listen on. This option is only relevant when using the SMB `CommType`, and will only be displayed if you have selected the SMB `CommType`.

Since we are generating a Grunt for the HTTPS listener with a self-signed certificate and for the reasons listed above, we can set `ValidateCert` and `UseCertPinning` to false. Since this is an example, I will leave the other options  default, however, on an engagement, these should absolutely be changed. Clicking the "Generate" button will compile the Grunt.

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-12.png)

At the top of the launcher screen, there are 3 tabs "Generate", "Host" and "Code". We just utilized the generate tab to set the launcher options and generate the actual Grunt binary. The Host tab is used to host the launcher on the Covenant server. It has a "Url" option which can be set and then navigated to over the listener.

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-13.png)

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-14.png)

The Code tab can be used to view the source code of the launcher's stager, "StagerCode":

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-15.png)

<br />

# Executing a Grunt
Taking our created binary from above and running it on a system should give us an active Grunt connection. We are not delving into bypassing AV/EDR, obfuscating the Grunt binary, or getting the Grunt on the system as this is out of scope for this blog post. AV was disabled on this machine in order to get the default binary Grunt to execute. We use PowerShell to directly execute the binary, then verify a grunt has checked in using the Covenant GUI.

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-16.png)

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-17.png)

Once the Grunt checks in, we see a handful of information about the connection, can interact with the Grunt and issue tasks or commands through the web UI using the `Interact` tab.

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-18.png)

![](https://github.com/active-labs/ACTIVEBlog/blob/main/Red%20Team%20Infrastructure%20-%20C2/assets/Red_Team_Infrastructure_-_C2-19.png)

<br />

# Final Thoughts
Congratulations! You should now have a fully working Covenant server running and have the knowledge to get HTTP and HTTPS listeners running. Covenant is one of the best open source C2 frameworks available right now. Development is active and changes are being pushed to GitHub on a daily basis. If the project does not offer a specific capability, it is fairly straight forward to open the source and start modifying code. I hope this post is as beneficial to others as it was for me to do the research and document some notes.