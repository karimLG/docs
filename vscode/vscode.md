 # Visual Studio Code usage for Yocto / DC

[TOC]

# Presentation
*VSCode* can be used as development software in the *Yocto* environment.

# VSCode Installation
In this document, *VSCode* is used to illustrate the use of the development software in the *Yocto* environment.

*VSCode* can be installed from *Ubuntu Software*.\
![](/images/VSCode_000.png)

## C/C++ extensions
Add *C/C++* and *C/C++ Extension Pack* extensions.\
![](/images/VSCode_001.png)\
![](/images/VSCode_002.png)

# VSCode developpment
*VSCode* must be run from a [initialized SDK environment](/index.md/#initialize-sdk-environment):
[initialized SDK environment](/index.md)
[initialized SDK environment](/index.md/#yocto-project-documentation-for-dcs)
[initialized SDK environment](/index.md#yocto-project-documentation-for-dcs)
**![warning] TODO correct link**
>\<workspace>: project folder
```bash
. /opt/landis/0.1/environment-setup-cortexa8t2hf-neon-poky-linux-gnueabi
code <workspace>
```

## VSCode build
In the project folder, create or add build task configurations in *.vscode/tasks.json* file:
```json
{
	"version": "2.0.0",
	"tasks":
	[
		{
			"label": "make all",
			"command": "make",
			"args": ["all"],
			"problemMatcher": ["$gcc"],
			"group": {"kind": "build", "isDefault": true}
		},
		{
			"label": "make clean",
			"command": "make",
			"args": ["clean"],
			"problemMatcher": ["$gcc"],
			"group": {"kind": "build"}
		}
	]
}
```
Project can be build from *Terminal/Run Build Task... (Ctrl+Shift+B)* menu
and clean from *Terminal/Run Task...* menu (make clean).

## VSCode debug
A script file must be created to source the SDK before starting the debugger.
Create *~/.config/Code/User/YoctoSdkGdb.sh*:
```bash
. /opt/landis/0.1/environment-setup-cortexa8t2hf-neon-poky-linux-gnueabi && $GDB $@
```

In the project folder, create or add debug launch tasks configuration in *.vscode/launch.json* file:
```json
{
    "version": "0.2.0",
    "configurations":
    [     
        {
            "type": "gdb",
            "request": "attach",
            "name": "Attach to gdbserver",
            "executable": "helloworld",
            "target": "100.0.0.1:3389",
            "remote": true,
            "cwd": "${workspaceRoot}", 
            "gdbpath": "${userHome}/.config/Code/User/YoctoSdkGdb.sh"
        }
    ]
}
```

*gdbserver* must be run in the DC to launch the application to be debugged:
>\<application>: path to the application
```bash
gdbserver :3389 <application> --no-background --verbose
```

Application can be debugged from *Run/StartDebugging (F5)* menu
or *Run and Debug (Ctrl+Shift+D)* perspective.\
![](/images/VSCode_004.png)