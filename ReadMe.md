# Git Utilities #

Thor repository tools to make it easier to work with Git in a VFP development environment.  

### Prerequisites ###

The following items are required to take full advantage of the Git utilities:  

 - [Thor](http://vfpx.codeplex.com/wikipage?title=Thor) - The Thor framework must be loaded as part
   of the VFP development environment.  
 - The Thor Repository (this is a collection of tools, and not to be confused with a Git repository)  
 - [FoxBin2Prg](http://vfpx.codeplex.com/wikipage?title=FoxBin2Prg) - The minimum required version
   is 1.19.42.  For best results, FoxBin2Prg should be installed via Thor's "Check for Updates"
   feature.  If FoxBin2Prg is installed some other way, the Thor plug-in "Get FoxBin2Prg Folder"
   can be edited to tell Thor where you have FoxBin2Prg installed.  
 - [Git for Windows](http://git-scm.com/download/win) - When installing Git for Windows, at the step
   entitled **Adjusting your Path environment**, select the second option "Use Git from the Windows
   Command Prompt".  
   **NOTE:** Beginning with the 2.x versions of Git for Windows, there are 32- and 64-bit versions.
   You MUST have the 32-bit version of Git for Windows installed in order for these utilities to
   work, even on 64-bit Windows, because VFP is a 32-bit app, and is unable to run 64-bit Git from
   the command line.  If you need the 64-bit version of Git for Windows, it appears that both the
   32-bit and 64-bit versions can be installed on the same PC, but in my testing, the 64-bit install
   uninstalled the 32-bit version, so the 32-bit version must be installed LAST.
 - Windows Scripting Host - This is installed as part of Windows, but system administrators will
   sometimes disable it on the PCs on their networks.  

#### Getting Started with Git ####

If you are new to Git, I've created a
[setup guide](https://drive.google.com/file/d/0B1GXcfuc1fBubFpUS2VmSVNpUEk/view?usp=sharing)
for getting started with Git.  This guide includes:  

 - some basic instructions for installing Git and some useful third-party tools
 - some tips on configuring Git and the tools
 - information on SSH keys
 - a list of resources, including documentation, articles, and links to Git and all the tools
   referenced  

### Installation ###

#### Release ####

The Git Utilities are included in Thor's **Check for Updates**.  When installed, these tools will
appear in the Thor tools menus under `Applications > Git Utilities`.  

**NOTE:** If you previously installed Git Utilities in the `My Tools` folder, you need to delete all
`Thor_Proc_GitUtilities.*` and `Thor_Tool_GitUtilities_*.*` files from that folder.  If you don't,
the files in `My Tools` will be used instead of the files coming from _Check for Updates_.  

#### Beta ####

Occasionally I may make new features available via a beta version of Git Utilities before releasing
them in the regular version.  If you would like to try the beta version, you can manually add it to
the **Check for Updates** list by copying the file `Thor_Update_Git-Utils-Beta.PRG` to your
`My Updates` folder.  This folder can be easily accessed from the Thor menu by selecting
`Thor > More > Open Folder > Tools`.  From there you can navigate to `Updates\My Updates`.  You can
then run **Check for Updates**, and you will see a new item in the list named _Git Utilities (Beta)_.

**NOTE:** The beta version of Git Utilities completely replaces the release version, so you cannot
have both installed at the same time.  However, it is very easy to return to the release version
again by simply running **Check for Updates** and selecting Git Utilities to reinstall it.  

### Tool Options ###

The options for these tools are found under **Git Utilities** on the Options page of the Thor
Configuration form.  The same options can also be accessed from the _Options_ link on the tool
page of each tool that is affected by the options.

### Description of Tools ###

#### `Thor_Proc_GitUtilities.PRG` ####
This is the main program file, which defines the Git utilities class that contains all the features
used by the other tools in this collection.  If you want to access the features of this class
directly, you can do so using the following command:  

```
loGitUtils = EXECSCRIPT( _Screen.cThorDispatcher, "Thor_Proc_GitUtilities" )
```  

The code contains documentation for all of the properties, methods, parameters, and return values.

#### `Thor_Tool_GitUtilities_PrepareForCommit.PRG` ####
This tool appears under the Git Utilities menu as **Prepare for Git commit**.  It operates on either
an entire VFP project, or an individual Git repository folder.  It performs several operations to
prepare VFP files to be committed to their respective repositories.  

 - When a project is processed, a text version of the project file itself is generated.
 - A text file is generated for each VFP binary file which has been modified since the last commit.
   The FoxBin2Prg configuration settings determine which binary files have text files generated
   for them, and how the file is generated.  
   **Note:** Untracked files (those that are new and unstaged) are ignored.  For new binary files
   which have not been committed to the repository yet, a text file is only generated if the primary
   binary file (VCX, SCX, MNX, FRX, LBX, DBF, DBC) is *staged* in Git.     
 - For any binary file that has changed because VFP generated new object code, but the source code
   has *not* changed (the text file is still the same), the changes to the binary file will be
   reverted.
 - *(New - 2015-10-12)* If the primary binary file (VCX, SCX, MNX, FRX, LBX, DBF, DBC) is *staged*
   in Git, but the secondary file(s) (VCT, SCT, etc.) and/or the text file are still untracked
   (haven't been added to the repository yet), then those untracked files will be added to the
   repository automatically.  This causes those files to be staged for commit, and timestamps will
   be saved for them if you are using that option.
 - In the Git Utilities options, if you have specified that timestamp files are be generated,
   then the timestamp file for each repository being processed will be updated accordingly.  
   **Note:** Untracked files (those that are new and unstaged) are ignored.  For new files which
   have not been committed to the repository yet, timestamps are only saved for the new files that
   have been *staged* in Git.

**NOTE #1:** This tool assumes that both the VFP binary files and their corresponding text
files are being committed to the Git repositories (with the exception of the .PJX file).  If
only binary or only text files are being committed, then this tool will not be very helpful,
and may even undo some legitimate changes, and therefore should not be used.  

**NOTE #2:** When this tool reverts binary files that haven't really changed, Git is used to replace
those files.  That means that any databases, tables, or classes which are being reverted need to be
closed/released before running this utility.  Files that are still open can cause this utility to
"hang", preventing the process from completing.

#### `Thor_Tool_GitUtilities_PostCheckoutSynch.PRG` ####
This tool appears under the Git Utilities menu as **Post-checkout file synchronization**.  It
operates on either an entire VFP project (all the repositories in that project), or an individual
Git repository folder.  It performs several operations to synchronize binary files and object code
to files that have been checked out from a branch of a Git repository.  

 - If the selected repositories contain FoxBin2Prg text files for .PJX files, the .PJX files will
   be regenerated.  
   **Note:** This assumes you don't commit .PJX/.PJT files to your repositories.
 - For any binary menu files (.MNX) in the repositories, the code files (.MPR) will be regenerated
   and recompiled using whatever program is specified in the VFP `_GENMENU` system variable.  
   **Note:** This assumes you don't commit .MPR/.MPX files to your repositories.
 - All program files in the repositories will be recompiled.   
   **Note:** This assumes you don't commit .FXP files to your repositories.
 - In the Git Utilities options, if you have specified that timestamp files are be generated,
   then for each repository being processed that has a timestamp file, the file modification dates
   will be restored for all files in the repository.  

Since this tool finds and regenerates .PJX files for all the project text files in the selected
repositories, it is normally simpler and faster than running FoxBin2Prg directly.  In addition,
regenerating and recompiling just the menus and programs is faster than recompiling an entire
project to accomplish the same end result, and will save time when running the **Prepare for Git
commit** tool, since it won't change the object code in the other VFP binary files.  

#### `Thor_Tool_GitUtilities_SaveProjectTimeStamps.PRG` ####
This tool appears under the Git Utilities menu as **Save file timestamps**.  It operates on either
an entire VFP project, or an individual Git repository folder.  The purpose of this tool is to
preserve the modification date of each file being tracked in a repository, which Git does not do.
Whenever files are checked out of a Git repository, they are assigned the current system date and
time.  However, sometimes it is nice to know the last time a particular file was modified.  This
tool creates/updates a timestamp file in the root folder of each repository being processed, saving
the last modification date for each file being tracked in the repository.

#### `Thor_Tool_GitUtilities_RestoreProjectTimeStamps.PRG` ####
This tool appears under the Git Utilities menu as **Restore file timestamps**.  It operates on
either an entire VFP project, or an individual Git repository folder.  Its purpose is to restore
the modification dates for all files listed in the timestamp files created by the *Save file
timestamps* tool.

#### `Thor_Tool_GitUtilities_ShowReposInProject.PRG` ####
This tool appears under the Git Utilities menu as **Show Git repos in project**.  It operates on
the active VFP project, or if no project is open, it prompts for one.  This is strictly an
informational tool, which displays a cursor containing a list of all Git repositories which
include files from the project, the branch that is currently checked out for each project, and a few
stats about any changes or conflicts for each repository.

#### `Thor_Tool_FoxBin2Prg_DB2toDBF.PRG` ####
**PLEASE NOTE:**  As of FoxBin2Prg version 1.19.47, this tool is no longer needed.  There is a new
FoxBin2Prg.CFG setting `DBF_Conversion_Support: 8`, which provides bidirectional conversion of
.DBF/.DB2 files directly from FoxBin2Prg.  

Unlike the other Git Utilities, this tool only uses FoxBin2Prg and not Git, and therefore can be
used with any source control system where you are using FoxBin2Prg.  For that reason, this tool is
installed in the Thor tools menus under `Applications > FoxBin2Prg`, so that it is grouped with
other FoxBin2Prg tools in the Thor repository.  It appears in that menu as **Populate DBF from DB2**.  

The purpose of this tool is to allow a .DBF file to be created *and populated* from a FoxBin2Prg
text file that was created with the FoxBin2Prg.CFG setting `DBF_Conversion_Support: 4`, effectively
giving you bidirectional support with this setting.  I wrote this tool because I had some settings
tables which had been edited in two different branches of the same repository, and I needed to
merge those changes.  The changes could be merged in the .DB2 file, and this tool was used to
regenerate the .DBF with the combined record changes.  

This tool has a couple limitations:  

 - It was written with free tables in mind, and has not been tested on tables that are contained in
   a VFP database.  While it might work on contained tables, use it at your own risk.
 - This tool cannot populate Varbinary, Blob, or auto-increment fields.  If you run this on a table
   containing those data types, it will abort with an error and not convert anything.

If you want to add the DB2-to-DBF converter to the Finder context menu, go to
`Thor > More > Manage Plug-Ins`.  In the *Manage Plug-In PRGs* dialog, locate the
*Finder context menu* plug-in, and click the *Action* button (either *Create* or *Edit*), and insert
code like the following into the plug-in .PRG:  

```
  *-- Attempt to get a reference to the text-to-DBF converter.
  loDB2toDBF = EXECSCRIPT( _Screen.cThorDispatcher, ;
      "Class= ConvertFoxBin2PrgTextToTable from Thor_Tool_FoxBin2Prg_DB2ToDBF.PRG" )
  IF VARTYPE( m.loDB2toDBF ) = "O"
    *-- Check if the selected file has a DBF text extension, and get
    *-- the command to add to the context menu for this file.  If the
    *-- command comes back empty, the file is not a DB2 file.
    lcExec = m.loDB2toDBF.GetMenuItemExec( m.lcFileName )
    IF NOT EMPTY( m.lcExec )
      *-- Add the item to the context menu.
      loContextMenu.AddMenuItem( "Convert " + m.lcQuote + JUSTFNAME( m.lcFileName ) + m.lcQuote ;
        + " to DBF", m.lcExec )
    ENDIF
  ENDIF
```
