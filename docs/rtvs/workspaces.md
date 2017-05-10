---
title: "Workspaces in R Tools for Visual Studio | Microsoft Docs"
ms.custom: ""
ms.date: 4/27/2017
ms.prod: "visual-studio-dev15"
ms.reviewer: ""
ms.suite: ""
ms.technology:
  - "devlang-r"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: d610279c-d6c3-4084-939a-bf042f64d4dd
caps.latest.revision: 1
author: "kraigb"
ms.author: "kraigb"
manager: "ghogen"
translation.priority.ht:
  - "cs-cz"
  - "de-de"
  - "es-es"
  - "fr-fr"
  - "it-it"
  - "ja-jp"
  - "ko-kr"
  - "pl-pl"
  - "pt-br"
  - "ru-ru"
  - "tr-tr"
  - "zh-cn"
  - "zh-tw"
---


## Controlling where R code runs with workspaces

A workspace in R Tools for Visual Studio (RTVS) lets you configure where an R session runs, which can happen on both local and remote computers. The goal is to allow you to work on either with a comparable user experience, which gives you the ability to take advantage of potentially more powerful cloud-based computers.

To open the **Workspaces** window, use the **R Tools > Windows > Workspaces** command or press Ctrl+9.

![Workspaces window in R Tools for Visual Studio (VS2017)](media/workspaces-window.png)

In this window, the green checkmark indicates the active workspace to which RTVS is bound. Selecting a blue arrow sets the active workspace. The settings (gear) icon to the right of each workspace allows you to change its name, location, and command line arguments. The red X removes a manually-added workspace.

In this topic:

- [Saving and resetting a workspace](#saving-and-resetting-a-workspace)
- [Local workspaces](#local-workspaces)
- [Remote workspaces](#remote-workspaces)
- [Switching between workspaces](#switching-between-workspaces)
- [Directories on local and remote computers](#directories-on-local-and-remote-computers)
- [Copying project files to a remote workspace](#copying-project-files-to-a-remote-workspace)
- [Copying files from a remote workspace](#copying-files-from-a-remote-workspace)

## Saving and resetting a workspace

By default, RTVS doesn't save workspace state when you close and reopen a project. You can change this behavior, however, through the [Workspace options](options.md#workspace).

The **R Tools > Session > Reset** command and the reset toolbar button in the interactive window also reset workspace state at any time. In the case of remote workspaces, a reset has the effect of deleting the user profile created when first connecting to the remote server, which effectively deletes any files that have accumulated there.

## Local workspaces

The Local workspaces list displays all the R interpreters that you have installed on your computer. 

RTVS attempts to automatically detect all of the versions of R that you have installed by looking through the `HKEY_LOCAL_MACHINE\Software\R-Core\` registry key when Visual Studio starts up. Because this check is done only at startup, you'll need to restart Visual Studio if you install a new R interpreter.

RTVS might not detect an R interpreter that's installed in a non-standard way (for example, when simply copying files to a folder instead of running an installer). In this case, manually create a new local R Workspace as follows:

1. Select the **Add** button in the Workspaces window.
1. Enter a name for the new Workspace.
1. Enter the path to the R root folder, which is the one that contains the `bin` folder with the interpreter, and any optional command line arguments to pass to the interpreter when RTVS starts it up.
1. Select **Save** when you're done.

![Adding a new workspace](media/workspaces-add-new.png)

## Remote workspaces

Remote workspaces let you connect to an R session on a remote computer. (See [Setting up remote workspaces](workspaces-remote-setup.md) for how to configure a computer for this purpose.)

Remote workspaces aren't automatically detected by RTVS, so you must add them manually using the **Add** button in the Workspaces window as described in the previous section. In this case, enter the remote computer's URI rather than a local path.

> [!Important]
> Remote workspaces are identified by a URI that *must use the HTTPS protocol* to ensure the privacy and the integrity of communication with the remote computer. RTVS will not connect to a remote computer that does not support HTTPS.

> [!Note]
> Remote workspaces are effectively in preview. We are working on a better implementation of the file sync problem for a future release and welcome your ideas and feedback about how to make that a better experience for you.


## Switching between workspaces

RTVS is bound to only a single workspace at a time, which is indicated by a small green checkmark by that workspace in the Workspaces window. By default, RTVS binds to the last local Workspace that you had open in a previous session.

To change the active workspace, select the blue arrow next to the desired workspace. Doing so prompts you to save your session, terminates the current workspace, then switches to the new one.

> [!Tip]
> To disable the save prompt, select the **R Tools > Options** command and set the **Show confirmation dialog before switching workspaces** option to `No`. See [Workspace options](options.md#workspace).

If you attempt to switch to a local workspace that's been uninstalled or a remote workspace that it unavailable, you might encounter situations where an RTVS project is not bound to any workspace. As a result, you might see an error like the one below when you enter code in the interactive window or try to run code otherwise. To correct this, simply switch to another workspace in the Workspaces window. If none are available, you'll need to install an R interpreter. You can also try restarting Visual Studio you've installed an interpreter while Visual Studio running.

![Error when no workspace is bound to RTVS](media/workspaces-disconnected-interactive-window.png)

### Switching to a remote workspace

RTVS prompts you for credentials when you first connect to a remote workspace, then caches those credentials (using the secure Windows Credential Locker) for later sessions. Communication with the remote server is then done securely over HTTPS (which is required).

Depending on the server's configuration, you may see a certificate warning when connecting, "The security certificate presented by the Remote R Services does not allow us to prove that you are indeed connecting to the machine (name)."

![Self-signed certificate warning when connecting to a remote workspace](media/workspaces-remote-self-signed-certificate-warning.png)

The certificate is a document that is presented to RTVS by the machine that you are trying to connect to, which contains a field that identifies the URI of that machine. The warning appears when RTVS detects a mismatch between the URI in the certificate and the URI used to connect to the machine, indicating that the server's security may have been compromised.

However, this warning will also appear if a *self-signed certificate* was used to enable HTTPS on the remote machine instead of using one from a trusted provider. For more details, see [Setting up remote workspaces](workspaces-remote-setup.md).

## Directories on local and remote computers

By default, when you start a new R interpreter in a local workspace, your current working directory is `%userprofile%\Documents`. You can change this at any time using the **R Tools > Working Directory** commands, or by right-clicking a project in Visual Studio Solution Explorer and selecting commands like **Set Working Directory Here**.

On remote machines, RTVS automatically creates a user profile based on your credentials when you first connect to that server, so the working directory is the `Documents` folder under that profile. This will be used for all subsequent remote sessions that use the same credentials. 

As a result, the exact location where your code runs can differ between local and remote workspaces. In your code, then, avoid using absolute paths to data files and such, because your code won't likely be portable across workspaces. Use relative paths instead.

Note also that with remote workspaces, all files in the working directory remain in place across sessions for the same user profile. As noted earlier, you can delete these by using the  **R Tools > Session > Reset** command (or the reset button in the interactive window) when using a remote workspace. This again deletes the user profile from the server, which is recreated when you reconnect.

## Copying project files to remote workspaces

When working with R projects in Visual Studio, the local machine always has the latest project files even when you're using a remote workspace. That is, when you open a project in Visual Studio (which typically means opening a solution containing that project), RTVS assumes that the project's contents reside wholly on the local machine. The remote workspace is, in effect, just a temporary host for the project's files and any output from the code. This means, for example, that when loading a file using `source` in the interactive window, that file must already be on the remote machine in the path you provide, or it must be in the current working directory of the remote R interpreter (set with the `setwd()` function).

Files are copied to the remote server as follows:

- To work with files remotely through the interactive window, you must first copy them manually by right-clicking those files (or the project) in Solution Explorer and selecting **Source Selected **. For individual files, they'll be copied to the working directory on the server; when copying a project, RTVS will create a folder for the project.

- You can also copy files by selecting then in Solution Explorer and then selecting **Source Selected Files(s)**. This loads them into the interactive window and runs them there. If the session is connected to a remote computer, the files are copied there first.

- When RTVS is bound to a remote workspace and you press F5, select **Debug > Start Debugging** or otherwise start running your code, RTVS will by default copy the project's file to the remote workspace automatically (see below for how to control this).

- Any files that already exist on the server will be overwritten.

> [!Note]
> Because RTVS cannot reliably intercept all R function calls, calling functions such as `source()` or `runApp()` (for Shiny applications) from within the interactive window will *not* copy files froto the remote workspace.

Whether RTVS copies files when a project is run, and exactly which files are copies, is controlled through the [project properties](projects.md#project-properties). To open this page, select the **Project > (name) Properties...** menu command, or right-click the project in Solution Explorer and select **Properties...**.

![Project properties run tab with file transfer settings](media/workspaces-remote-file-transfer-filter-settings.png)

Here, **Transfer files on run** determines whether RTVS copies project files automatically. The **Files to transfer** value then filters exactly which files are transferred. The default is to copy only `.R`, `.Rmd`, `.sql`, `.md`, and `.cpp` files. This is done to avoid inadvertently copying large data files to the server with every run. 

## Copying files from a remote workspace

If your R script generates files on the server, you can copy those files back to the client using the `rtvs::fetch_file` function. This function accepts, at a minimum, the remote path to the file that you want to copy to your machine, and optionally the path on your machine where you want that file to be copied to. If you don't specify a path, the file will be copied into your `%userprofile%\Downloads` folder.