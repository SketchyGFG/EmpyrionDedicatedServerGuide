# EmpyrionDedicatedServerConfigurationGuide.txt

# 2024-08-18

The knowledge contained within this guide was collected thru the contribution of many people.  This guide offers a single perspective for running an Empyrion server on a headless, Windows server.  Some effort has been made to allow the reader to enter this document at any point without having to read everything, but please be mindful that it is impossible to provide everything needed at every given point without being terribly redundant.

While maintenance to this article may be given for errors, clarity and facts, the author has no interest in discussing the contents of this article or variations of it.  The author encourages others to build a server and learn from the experience.  Do it your way.  It is expected that at some point in time this document, in whole or in part, will no longer be accurate.

The contents of this guide was developed while running a large game server on recent hardware.  The situation was somewhat ideal and so we don't mention many problems that can be had.

In this guide we are making use of software built by other people, some of them are members of the community around this game.

- Empyrion Dedicated Server
- Empyrion Admin Helper (EAH)
- Empyrion Server Manager(ESM)
- Empyrion Web Access(EWA)

See this guide for further research: [Guide](https://empyrion.fandom.com/wiki/Guide/Setting_Up_Dedicated_Server)

************************************************************************

## Instructions regarding a Dedicated Empyrion Server:
```
CONTENTS:
1 - Initial installation
	1.1 - Configure CMD
	1.2 - Install SteamCMD
	1.3 - Install PeaZip
	1.4 - Install OSFMount
	1.5 - Place ESM files
	1.6 - Install base game
	1.7 - Install required code libraries
	1.8 - Install NSSM
	1.9 - Install scenario
	1.10 - Game configuration
	1.11 - Game ports
	1.12 - Backups
	1.13 - Ramdisk first time setup
	1.14 - Initial game startup
	1.15 - Non-Initial game startup
 
2 - Updates to base game
	2.1 - Using SteamCMD to update game
	2.2 - Server game cache folder
 
3 - Updates to scenario

4 - Server management
	4.1 - OS restarts
	4.2 - Starting the EAH
	4.3 - Stopping the EAH
	4.4 - Starting the server
	4.5 - Stopping the server
	4.6 - Automating restarts using EAH
	4.7 - Verify base game files
	4.8 - Erase existing game to start new game

5 - SteamCMD

6 - Empyrion Admin Helper (EAH)

7 - Empyrion Server Manager(ESM)

8 - Empyrion Web Access(EWA)

9 - CMD

10 - Telnet
```

## 1 - INITIAL INSTALLATION

This documents was assembled while using Windows Server 2022.  We currently expect the version of windows server will never matter.  We see no reason why this installation cannot be replicated on a Windows 10 or 11 machine.

Hardware, generally speaking:
- 256 GB of Ram
- Many cores
- 1TB of disk space for drive D:
- Disable 8dot3 naming to the target drive: [Guide](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/fsutil-8dot3name)
- Disable windows indexing to the target drive
- Disable windows security to the target drive
- open ports: 27740, 30000-30003

We assume we are installing this game server to a clean drive named D: and we plan to arrive at the following folder layout:
- d:\backups
- d:\empyrion
- d:\esm
- d:\installers
- d:\OSFMount
- d:\PeaZip
- d:\steamcmd
- d:\nssm

Folders briefly explained:
- `d:\backups` -  primary backup folder
- `d:\empyrion` - base game folder
- `d:\esm` - location for esm and most related scripts
- `d:\installers` - place all installers here after use, keeping only the latest version that is in use or any new versions yet to be installed
- `d:\OSFMount` - installed location of OSFMount
- `d:\PeaZip` - installed location of PeaZip
- `d:\steamcmd` - installed location of steamcmd
- `d:\nssm` - installed location for the Non-Sucking Service Manager (NSSM)

### 1.1 - Configure CMD

When using the basic command line window in Windows you will have at least one bad quirk that can make a mess of things, `Quick Edit Mode`.  Disable this option in the Default settings for all commands lines.  You won't need it for anything, and not disabling it can allow scripts to pause forever when the command window is clicked.

Steps:
1. Click the icon on the top left of the command windows and select `Default`.
2. Clear the option for `Quick Edit Mode`.
3. Click 'OK' at the bottom right and close the current command window.

### 1.2 - Install SteamCMD

Steps:
1. Download SteamCMD for Windows: [Download](https://steamcdn-a.akamaihd.net/client/installer/steamcmd.zip)
2. Create the folder `d:\steamcmd` if it doesn't already exist.
3. Extract the contents of the zip to the folder.
4. Place the installer archive into the following location: `d:\installers`

### 1.3 - Install PeaZip

PeaZip is a utility used by the ESM software for archive management. Remove any other previously install zip compression software as they can and will conflict with PeaZip.  PeaZip has everything so that you don't need anything else.

Steps:
1. Download PeaZip for windows from the following page: [Download](https://peazip.github.io/peazip-64bit.html)
2. Activate the installer and select `Custom installation` so that you can specify the install location.
3. Install PeaZip to the following location: `d:\PeaZip`
4. After install completes place the installer into the following location: `d:\installers`

### 1.4 - Install OSFMount

OSFMount is a utility used be the ESM software for ramdisk creation.  It is one of the fastest free ramdisk solutions available.  DO NOT EVER make any kind of symlink or hard link from the ramdisk back to the main drive as this will reduce the speed of the ramdisk to be the same speed of the hard disk!  You can link to it, but never from it.

Steps:
1. Download OSFMount from the following page: [Download](https://www.osforensics.com/tools/mount-disk-images.html)
2. Activate the installer and install OSFMount to the following location: `d:\OSFMount`
3. After install completes place the installer into the following location: `d:\installers`

### 1.5 - Place ESM files

ESM software is a set of scripts that will modify the base game to run in a ramdisk and help you manage the game from that point on.

Steps:
1. Download the latest ESM software from the following location: [Here](https://github.com/Vollinger0/esm/releases)
2. Extract the contents of the software archive into the following location: `d:\esm`
3. Place the software archive into the following location: `d:\installers`
4. Test the esm by running the following command from an administrative command line: `d:\esm\esm check-requirements`
5. Review test results and follow any guidance the esm gives you.

Optional:
We tend to create a folder named 'examples' and push any file containing the word 'example' into that folder.  This helps to de-clutter the contents of the main esm folder, and keeps the examples file sane(untouched).

### 1.6 - Install base game

To install the base game we must invoke the SteamCMD utility and download the game directly from Steam.  The EAH will be installed at the same time and we will using it to start and stop the server.  It is important to use the validation feature whenever possible to insure fewer issues occur.  This is accomplished by adding the 'validate' option to the command AND invoking the SteamCMD utility a second time successively.  The following command is used for everything related to SteamCMD.  The order of command options matters:
```
d:\steamcmd\steamcmd.exe +force_install_dir d:\empyrion\ +login anonymous +app_update 530870 validate +quit
```

Steps:
1. Open a command line with administrative access.
2. Use the following command to install the base game:
```
d:\steamcmd\steamcmd.exe +force_install_dir d:\empyrion\ +login anonymous +app_update 530870 validate +quit
```

3. Use the following command *again* to validate the base game:
```
d:\steamcmd\steamcmd.exe +force_install_dir d:\empyrion\ +login anonymous +app_update 530870 validate +quit
```

### 1.7 - Install required C++ code libraries

With the game installed there should now be a folder located at: `D:\empyrion\_CommonRedist\vcredist\2019`.  In this folder there should be the files VC_redist.x64.exe and VC_redist.x86.exe.  These installers need to be executed (just double click them) before the base game will run.  It is possible to install more recent versions of these executables as well.

Executables:
```
D:\empyrion\_CommonRedist\vcredist\2019\VC_redist.x64.exe
D:\empyrion\_CommonRedist\vcredist\2019\VC_redist.x86.exe
```

### 1.8 - Install NSSM

The Non-Sucking Service Manager (NSSM) provides and easy way to create and manage services under windows.  It is possible to install this utility anywhere so long that you can remember the path to it.  The game does not need to know where this utility is installed.  You will use this utility to install and manage windows services.

Steps:
1. Download the NSSM software from the following page: [Download](https://nssm.cc/download).  Using the pre-release version is preferred.
2. Extract the contents of the software archive into the following location: d:\nssm  The folders for src, win32, and win64 should be directly under the top folder.
3. Place the software archive into the following location: d:\installers


### 1.9 - Install scenario

Since there is usually no way to download the scenario directly from steam without providing personal information it is recommended to update the scenario from a personal account using something like https://justbeamit.com/  The scenario you are uploading may come as a folder with a number for a name like: 3143225812.  Zip this folder like it is and beam it up to the server.  It's best that you do NOT use that number as the same name as the scenario folder on the server, so change it when you move the folder into position.

Steps:
1. Upload the scenario archive to the server.
2. Decompress that archive to a location outside of the game folder, like `d:\installers`.
3. Check that the file `gameoptions.yaml` is visible just under the folder or un-nest folders until it is.
4. Since this guide expects to be dealing with the RE2 Beta scenario, we will name the folder RE2Beta.
4. Check that the a scenario folder with the same name does not exist in: `d:\empyrion\Content\Scenarios`.
5. Migrate the scenario folder to: `d:\empyrion\Content\Scenarios`.
7. Apply any changes to the game the are desired.
6. Initial game defaults are stored in: `gameoptions.yaml` and should be set before starting the game for the first time.  It is possible to use the EAH to apply these settings without starting the game server.  Use the EAH and check/set these now.


### 1.10 - Game configuration
	1.10.1 - ESM
	1.10.2 - EAH
	1.10.3 - Base game
	1.10.4 - NSSM
	1.10.5 - Scenario

#### 1.10.1 - ESM

Configuration and files to be found in the location:

	d:\esm

Files:
- `callesm-backup-sync.bat` - call this file to make game backups from the ramdisk
- `callesm-backup-async.bat` - call this file to make asynchronously backups from the ramdisk
- `esm-custom-config.yaml` - esm configuration
- `SqlQueries_disabled.txt` - file to disable rank queries


FILE: callesm-backup-sync.bat, top only
```
@echo off
setlocal enabledelayedexpansion
REM just a wrapper script to be called by EAH that will start esm synchronously (blocking the caller until it ends)
REM
REM by vollinger 20231102

REM ################################################################################################################
REM ## CONFIGURATION

REM path to esm tool installation
REM ************** UNCOMMENT THE FOLLOWING LINE AND MAKE SURE THE PATH POINTS TO THE ESM INSTALLATION **************
set "esmPath=d:\esm"

REM esm command to execute synchronously, blocking the caller
set "esmCommand=esm backup-create"

REM ################################################################################################################
REM ## script start
```

FILE: callesm-backup-async.bat, top only
```
@echo off
setlocal enabledelayedexpansion
REM just a wrapper script to be called by EAH that will start esm asynchronously (not blocking the caller)
REM
REM by vollinger 20231102

REM ################################################################################################################
REM ## CONFIGURATION

REM path to esm tool installation
REM ************** UNCOMMENT THE FOLLOWING LINE AND MAKE SURE THE PATH POINTS TO THE ESM INSTALLATION **************
set "esmPath=d:\esm"

REM esm command to execute asynchronously in the background
set "esmCommand=esm backup-create"

REM ################################################################################################################
REM ## script start
```

FILE: esm-custom-config.yaml, whole file
```
general:
  useRamdisk: True # True|False. if True, use a ramdisk for the savegame. Requires the user to call ramdisk-install and ramdisk-setup to work. Will completely solve your server performance issues.

ramdisk:
  drive: "R:" # string. the drive letter to use for the ramdisk, e.g. "R:"
  size: "50G" # string. ramdisk size to use, e.g. '5G' or '32G', etc. If you change this, the ramdisk needs to be re-mounted, and the setup needs to run again.

server:
  dedicatedYaml: 'esm-dedicated.yaml' # string. path to dedicated yaml, make sure this is the one defined in EAH if you use that

backups:
  additionalBackupPaths: # list of full paths to source files or directories to backup additionally. Those will all end up in the folder "Additional" in the backup.
    - 'd:/empyrion/Content/Scenarios/RE2Beta' # the whole modpack
    - 'd:/esm/esm-custom-config.yaml' # our own configuration

updates: 
  additional: # additional stuff to copy when calling the esm game-update command
    # If a path is relative, it will be assumed to be relative to the installation path.
    # source path globs (wildcards) are supported
    # if dst does not exist and is not a directory, a directory with that name will be created and used as target for the source.
    # if src is a directory, it will be copied recursively into the target folder.
    # existing files/directories in the target will be overwritten!
    #
    # copy all item icons from the scenario to EAH
    - {src: 'd:/empyrion/Content/Scenarios/RE2Beta/SharedData/Content/Bundles/ItemIcons/*.*', dst: 'd:/empyrion/DedicatedServer/EmpyrionAdminHelper/Items'}
    # copy custom sqlqueries to the game config
    - {src: 'd:/esm/SqlQueries_disabled.txt', dst: 'd:/empyrion/Content/Configuration/SqlQueries.txt'}

paths:
  install: 'd:/empyrion/' # the games main installation location
  osfmount: 'd:/OSFMount/osfmount.com' # path to osfmount executable needed to mount the ram drive
  peazip: 'd:/PeaZip/res/bin/7z/7z.exe' # path to peazip used for the static backups
  epmremoteclient: 'd:/esm/emprc/EmpyrionPrime.RemoteClient.Console.exe' # path to emprc, used to send commands to the server
  eah: 'd:/empyrion/DedicatedServer/EmpyrionAdminHelper' # path to EAH, for backing up its data
  steamcmd: 'd:/steamcmd/steamcmd.exe' # path to steamcmd for installs and updates of the game server

downloadtool:   # configuration for the shared data download tool
  customExternalHostNameAndPort: 'http://x.x.x.x:27440'   # if set, this will be used as the host instead of the automatically generated host-part of the url. must be something like: 'https://my-server.com:12345'. The path/name of the files will be appended.
  useSharedDataURLFeature: true                                  # if true, a zip for the SharedDataURL feature will be created, served and the dedicated yaml will be automatically edited.

communication:
  announceSyncEvents: false                                      # if true, sync events (syncStart, syncEnd) will be announced in the chat
```

FILE: SqlQueries_disabled.txt, top only
```
# -------------------- Configuration of the ranking boxes, RankingBoxX,Y

#- RankingBox1,1 = RkgPlayerKills
#- RankingBox2,1 = RkgNPCKills
#- RankingBox3,1 = RkgPlayerDied
#- RankingBox4,1 = RkgPlaytime
#- RankingBox5,1 = RkgBlocksPlaced

#- RankingBox1,2 = RkgVisitedStars
#- RankingBox2,2 = RkgVisitedPlanets
#- RankingBox3,2 = RkgVisitedMoons
#- RankingBox4,2 = RkgDiscoveredPOIs
#- RankingBox5,2 = RkgDiggedOres

#- RankingBox1,3 = RkgTravelledKM
#- RankingBox2,3 = RkgTravelledKMAir
#- RankingBox3,3 = RkgTravelledAU
#- RankingBox4,3 = RkgTravelledLY
#- RankingBox5,3 = RkgTradeVolume

# -------------------- RkgPlayerKills
```

#### 1.10.2 - EAH

EAH configuration and files to be found in the location: `d:\empyrion\DedicatedServer\EmpyrionAdminHelper\Config`. Do not rename these files except to make backups. Setup of the EAH is best done from within the EAH where settings are consolidated into interface.

Files:
- `Settings.XML` - Primary EAH configuration.
- `AdminProfile.XML` - EAH users with admin access.  Helps with warping and impersonation.
- `TimeTable.XML` - Automation provided thru EAH.
- `Chatbot_Options.XML` - Text sources for some CB: options.

#### 1.10.3 - Base game

Configuration and files to be found in the location `d:\empyrion`

Files:
- `esm-dedicated.yaml` - base game configuration
- `esm-starter-for-eah.cmd` - file used by EAH to start game

FILE: esm-dedicated.yaml, Vanilla server, whole file
```
ServerConfig:
  Srv_Port: 30000
  Srv_Name: This is the name of the vanilla server that will appear publicly!
  Srv_MaxPlayers: 128
  Srv_ReservePlayfields: 2
  Srv_Description: 'Wiped on mm.dd.yyyy'
  AdminConfigFile: adminconfig.yaml
  Tel_Enabled: true
  Tel_Port: 30004
  Tel_Pwd: ThisIsAPassword!
  SaveDirectory: Saves
  EACActive: true
  MaxAllowedSizeClass: 25
  AllowedBlueprints: All
  HeartbeatServer: 15
  KickPlayerWithPing: 3000
  TimeoutBootingPfServer: 300
  PlayerLoginParallelCount: 5
GameConfig:
  GameName: Default Multiplayer
  Mode: Survival
  Seed: 12345678
  CustomScenario: Default Multiplayer
```

FILE: esm-dedicated.yaml, RE2 Beta server, whole file
```
ServerConfig:
  Srv_Port: 30000
  Srv_Name: This is the name of the RE2Beta server that will appear publicly!
  Srv_MaxPlayers: 128
  Srv_ReservePlayfields: 2
  Srv_Description: 'Wiped on mm.dd.yyyy'
  AdminConfigFile: adminconfig.yaml
  Srv_Public: true
  Tel_Enabled: true
  Tel_Port: 30004
  Tel_Pwd: ThisIsAPassword!
  SaveDirectory: Saves
  EACActive: true
  MaxAllowedSizeClass: 25
  AllowedBlueprints: All
  HeartbeatServer: 15
  KickPlayerWithPing: 3000
  TimeoutBootingPfServer: 300
  PlayerLoginParallelCount: 5
GameConfig:
  GameName: RE2Beta
  Mode: Survival
  Seed: 12345678
  CustomScenario: RE2Beta
```

FILE: esm-starter-for-eah.cmd, top only, copied from d:/esm/esm-starter-for-eah.example.cmd
```
@echo off
setlocal enabledelayedexpansion
REM just a wrapper script to be called by EAH that will start esm asynchronously
REM
REM by vollinger 20231102

REM ################################################################################################################
REM ## CONFIGURATION

REM path to esm tool installation
REM ************** UNCOMMENT THE FOLLOWING LINE AND MAKE SURE THE PATH POINTS TO THE ESM INSTALLATION **************
set "esmPath=d:\esm"

REM ################################################################################################################
REM ## script start
```

#### 1.10.4 - NSSM

The NSSM must be called from an administrative command line.  We are only providing settings that matter, and so anything else does not matter.  Create the services below using provided information.  Do not start these services at this time.

1. Empyrion_StarterService:

Commands: `d:\nssm\win64\nssm.exe install Empyrion_StarterService` or `d:\nssm\win64\nssm.exe edit Empyrion_StarterService`

Settings that matter:
```
Service name		Empyrion_StarterService
Display name		Empyrion_StarterService
Description		Empyrion_StarterService
Path			d:\empyrion\DedicatedServer\EmpyrionAdminHelper\EmpAdminHelper_StarterService.exe
Startup directory	d:\empyrion\DedicatedServer\EmpyrionAdminHelper
```

Do not start these services at this time.

2.) Empyrion_SharedDataServer:

Commands: `d:\nssm\win64\nssm.exe install Empyrion_SharedDataServer` or `d:\nssm\win64\nssm.exe edit Empyrion_SharedDataServer`

Settings that matter:
```
Service name		Empyrion_SharedDataServer
Display name		Empyrion_SharedDataServer
Description		Empyrion_SharedDataServer
Path			d:\esm\esm.exe
Startup directory	d:\esm
Arguments		tool-shareddata-server
```

Do not start these services at this time.


#### 1.10.5 - Scenario

The following setting must be set before starting the game for the first time.  It is possible to use the EAH to apply these settings without starting the game server.

FILE: gameoptions.yaml, RE2 Beta server, whole file
```
Options:
- ValidFor:
  - MP
  - Survival
  DecayTime: 0
  MaxStructures: 2000
  AntiGriefDistancePvE: 0
  AntiGriefDistancePvP: 0
  AntiGriefOresDistance: 0
  EnableVolumeWeight: true
  EnableCPUPoints: true
  AutoMinerDepletion: false
  GroundedStructureSpawn: false
  TurretUndergroundCheck: true
  OriginAccessOthers: false
  OriginAutoAlliance: false
  MaxSpawnedEnemies: 60
  DespawnEscapePod: true
  RegeneratePOIs: true
  CustomPlayerTab: Scenario Change Log;https://steamcommunity.com/sharedfiles/filedetails/changelog/3143225812
  DiffAmountOfOre: Rich
  DiffNumberOfDeposits: Plenty
  AntiGriefDistancePvE: 0
  AntiGriefDistancePvP: 0
  GroundedStructureSpawn: False
```

### 1.11 - Games ports

Incoming game ports that matter:
- Empyrion: `30000 - 30004`
- ESM: `27440`


### 1.12 - Backups

When setting up backup routines we prefer to combine folders into one game backup location to reduce the load on management and allow for relocation if desired.  The base game wants to use a specific backup folder by default so I will link that folder to the real backup location.  Next I will create a task in the timetables of the EAH to backup structures.  Last I will create a windows scheduling task to automate save folders dumps to the backup location.

Steps:
1. Remove the folder at the location: `d:\empyrion\Backup`
2. From an administrative command line execute the following command: `mklink /D "d:\empyrion\Backup" "D:\backups"`
3. Add the following line to the EAH timetables, the EAH will replace the timestamp when processed:
```
<TT Ld="2024-07-23T03:17:41.5493315-04:00" W="13" A="17" P="" P2="" DZ="" CC="DFA531" AC="true" />
```
4. Create the following windows scheduling task:
```
	Create New Task...
	General tab:
		Name									EmpyrionGameBackup
		Location								\
		user account								SYSTEM
		Run with highest privileges						true
	Triggers tab (make one trigger):
		Begin the task								On a schedule
		Settings								Daily
		Start									enter today's date @ 12:30 AM
		Recur every								1 days
		Repeat task every							6 hours
		for a duration of							1 day
		Stop task if it runs longer than					4 hours
		Enabled									true
	Actions tab (make one action):
		Action									Start a program
		Program.script								D:\esm\callesm-backup-async.bat
		Start in (! this is NOT optional)					D:\esm\
	Settings tab (clear is not mentioned)
		Allow task to be run on demand						true
		Stop the task if it runs longer than					4 hours
		If the running task does not end when requested, force it to stop	true
		Does not start a new instance						selected
```

### 1.13 - Ramdisk first time setup

Steps:
1. From an administrative command line execute the following command: `d:\esm\esm.exe check-requirements`. Review feedback and follow any guidance.
2. From an administrative command line execute the following command: `d:\esm\esm.exe ramdisk-install`. This process may start the game momentarily to create save game files, then reorganize them for migration to the ramdisk.
3. From an administrative command line execute the following command: `d:\esm\esm.exe ramdisk-setup`. This process creates the ramdisk and migrates the save game to it.
4. From an administrative command line execute the following command: `d:\esm\esm.exe check-requirements`. Review feedback and follow any guidance.

### 1.14 - Initial game startup

Initial game start must be manual due to windows requiring human approval of running applications.  In this case we will start the game using the EAH, giving focus to the command windows that will appear and watching for system warnings.  When the 'Open File - Security Warning' dialog appears it is important un-check the option 'Always ask before opening this file'.  You should see 3 dialogs during the startup process.  It is important to uncheck the option 'Always ask before opening this file' from each dialog before the game will fully run in the background.

Steps:
1. Stop any related services (`Empyrion_SharedDataServer` service, `Empyrion_StarterService` service, etc.)
2. Start the EAH locally from path: `D:\empyrion\DedicatedServer\EmpyrionAdminHelper\EmpyrionAdminHelper.exe`, or using a link you may have created to your desktop.
3. Functions (sidebar) - Server group: Hit the button 'Start'.
4. You will see commands windows appear and you will want the top-most command window to remain in focus.
5. Look for the `Open File - Security Warning` dialogs to appear and un-check the option `Always ask before opening this file` for each one.  There should be 3 in all.  If something about this does not look right, stop the server and try again from step 3.
6. If all goes well and all 3 dialogs are un-checked, you should then consider testing that the server is fully functional.  Connecting to it via client is usually the best way.
7. Stop the server and proceeding to `section 1.15`.

### 1.15 - Non-Initial game startup

The following process will work once the process from `section 1.14`: Initial game startup, has been fullfilled.  If you have any problems you might consider revisiting that section. Do not start the EAH from the server or the EAH may not be running as the correct user (SYSTEM), so when you logout from the server the EAH and anything running from it will die.

Steps:
1. Start the `Empyrion_StarterService` service and check to see if a process for the `EmpAdminHelper_StartService.exe` is running.  We have seen this process sleep, so restart this service if you are not able to bring the EAH online in the following steps.
2. Start the EAH slave client on the remote machine.  The client will connect to the starter service.  You may not see any indication that this has occurred.
3. Functions (sidebar) - Master/Slave group: Hit the button `Start Master`
4. Check the process list from the server that a running process exists for `EmpAdminHelper_NoGUI.exe`.  If not, repeat step 1.
5. Exit the EAH slave client !!!
6. Start the EAH slave client again.  The client should now connect to the `EmpAdminHelper_NoGUI.exe` process.  You should see a green circle in the orange connect button.
7. Functions (sidebar) - Server group: Hit the button 'Start'.  The EAH will attempt to start the server process.  It is useful to watch the process list at the server and note what is happening.  The game process will start first, then load processes for at least 2 playfields.
8. Logout from server and check that served game continues to run.

************************************************************************
## 2 - Updates to base game

### 2.1 - Using SteamCMD to update game

When an update to the base game comes out you will usually find out from all the players who will no longer be able to connect to the server.  This is an important aspect of the survival game player experience and should not be avoided.  Try and wait until at least 5 members complain, then notice the update.  If you don't have 5 players on your server you may have to create a couple and pretend.

Steps:
1. Exit the game.
2. From an administrative command line execute the following command:
```
d:\steamcmd\steamcmd.exe +force_install_dir d:\empyrion\ +login anonymous +app_update 530870 validate +quit
```
It's important to verify the game files whenever possible.  You can't know when a problem will occur and it may not yet be obvious that there is an active problem.

3. From an administrative command line execute the following command 2-3 times:
```
d:\steamcmd\steamcmd.exe +force_install_dir d:\empyrion\ +login anonymous +app_update 530870 validate +quit
d:\steamcmd\steamcmd.exe +force_install_dir d:\empyrion\ +login anonymous +app_update 530870 validate +quit
```
4. Start game normally.

### 2.2 - Server game cache folder

Following some suggestion from other server operators we have on occasion attempted to remove the cache contents created by the server.  This is possible to do successfully under certain circumstances, and with a long running game can help to reduce overall game file size, and backup time.

That said, we feel that something is almost always broken when this is done.

Don't do this with any other action like a server move (ho ho, great times!).

Steps:
1. Take the server down.
2. Clear the cache folder.
3. Start the server.
4. Listen for players with missing ships.

************************************************************************
## 3 - Updates to scenario

When updates for scenarios are made available you will need to upload them to the server since there is usually no way to download them directly from steam without providing personal information.  Use https://justbeamit.com/ to migrate scenario files to the server from a local machine.  The scenario you are uploading may come as a folder with a number for a name like: 3143225812.  Zip this folder like it is and beam it up to the server.  It's best that you do NOT use that number as the same name as the scenario folder on the server, so change it when you move the folder into position.  If you make a mistake here with scenario folder placement you can and will corrupt the save game.

Steps:
1. Upload the scenario archive to the server.
2. Decompress that archive to a location outside of the game folder, like `d:\installers`.
3. Check that the file `gameoptions.yaml` is visible just under the folder or un-nest folders until it is.
4. Check that the scenario update does not have the same folder name as the scenario currently in use.
5. Migrate the scenario update folder to: `d:\empyrion\Content\Scenarios`
6. Apply any settings that are scenario specific.
7. Bring down the server.
8. Record the name of the current scenario folder and discard the folder into Trash.
9. Rename the updated scenario folder to the old scenario folder name.
10. Restart the service `Empyrion_SharedDataServer` and wait a minute for service startup to complete.
11. Continue with section 1.15 - Non-Initial game startup

************************************************************************
## 4 - Server management

### 4.1 - OS restarts

With operating system restarts you will lose the ramdisk and it's contents unless properly saved back to the game folders.  The ESM will dump the ramdisk back to the hard drive every hour so rollback time is minimized.

Steps:
1. From an administrative command line execute the following command: `d:\esm\esm.exe ramdisk-setup`
2. From an administrative command line execute the following command: `d:\esm\esm.exe check-requirements`
3. Continue with section 1.15 - Non-Initial game startup

### 4.2 - Starting the EAH

Steps:
1. Start the `Empyrion_StarterService` service and check to see if a process for the `EmpAdminHelper_NoGUI.exe` is running.  We have seen this process sleep, so restart this service if you are not able to bring the EAH online in the following steps.
2. Start the EAH slave client.  The client will connect to the start server.  You may not see any indication that this has occurred.  Do not start the EAH from the server or the EAH may not be running as the correct user (SYSTEM), so when you logout from the server the EAH and anything running from it will die.
3. Functions (sidebar) - Master/Slave group: Hit the button `Start Master`
4. Check the process list from the server that a running process exists for `EmpAdminHelper_NoGUI.exe`.  If not, repeat step 1.
5. Exit the EAH slave client !!!
6. Start the EAH slave client again.  The client should now connect to the `EmpAdminHelper_NoGUI.exe` process.  You should see a green circle in the orange connect button.

### 4.3 - Stopping the EAH

Steps:
1. Stop the `Empyrion_StarterService` service.
2. Watch the process list to ensure that both the `EmpAdminHelper_StartService.exe` and `EmpAdminHelper_NoGUI.exe` exit.  You may need to exit the process if that doesn't happen.

### 4.4 - Starting the server

Steps:
1. Start the Empyrion_StarterService service and check to see if a process for the `EmpAdminHelper_StartService.exe` is running.  We have seen this process sleep, so restart this service if you are not able to bring the EAH online in the following steps.
2. Start the EAH slave client.  The client will connect to the start server.  You may not see any indication that this has occurred.  Do not start the EAH from the server or the EAH may not be running as the correct user (SYSTEM), so when you logout from the server the EAH and anything running from it will die.
3. Functions (sidebar) - Master/Slave group: Hit the button `Start Master`
4. Check the process list from the server that a running process exists for `EmpAdminHelper_NoGUI.exe`.  If not, repeat step 1.
5. Exit the EAH slave client!!!
6. Start the EAH slave client again.  The client should now connect to the `EmpAdminHelper_NoGUI.exe` process.  You should see a green circle in the orange connect button.
7. Functions (sidebar) - Server group: Hit the button 'Start'.  The EAH will attempt to start the server process.  It is useful to watch the process list at the server and note what is happening.  The game process will start first, then load processes for at least 2 playfields.

### 4.5 - Stopping the server

We will usually stop the server from the EAH using one of the available buttons in the EAH, `Stop in 5`, `Stop now`, or `Restart in 5`.  Or we may stop the server by entering the command in EAH's CMD box `saveandexit 10` and pressing the black arrow.  You must press the black arrow for the command to process.

It may be important to watch the process list at the server to make sure that the game does indeed exit.  The game process can refuse to exit and the EAH will interpret this as a game restart, and, after a short time, try and stand everything back up.  Not sure what state the game would be in here, so lets avoid this if possible by force quitting the process when this appears to be the case.  Try and wait for the Robocopy process to finish before you do this, but not vital.

It may be useful to stop the Empyrion_StarterService service which will cause the EAH to exit and prevent restart.  Careful with this since the EAH creates additional timeline tasks during restart and those tasks may be re-processed for the event once the EAH is brought back online.  We have not seen an easy way to negate this so be prepared to watch the whole event happen again.

### 4.6 - Automating restarts using EAH

We are using the EAH timetables to handle most automation of the game.  We will do this with two lines.  The first line will warn players of the coming restart.  The second will perform the restart as well as a wiping deposits for selected starter playfields.  To handle Thursday player structure wipes without issue we will make a restart schedule for each day of the week.  This allows us to change the schedule for one day without affecting other days.
```
    <TT Ld="2024-07-24T14:03:09.5457359-04:00" W="17" A="0" P="***** RE2: DAILY REBOOT in 10 minutes *****" P2="" DZ="13:50" CC="DFCE31" AC="true" />
    <TT Ld="2024-07-24T14:03:09.5457359-04:00" W="17" A="5" P="***** RE2: DAILY REBOOT NOW *****" P2="" DZ="13:55" CC="DFA531" AC="true">
      <TTS A="32" P="Akua#Tallodar#Roggery#Uai#Omicron#Ningues#Oscutune#Pandora#Sienna#Maspero#Dread#Purgator#Rogue PX-37" P2="" AC="true" Delay="0" />
    </TT>
```
The EAH creates additional timeline tasks during restart so be careful with this since those tasks may be re-processed for the event once the EAH is brought back online.  We have not seen an easy way to negate this so be prepared to watch the whole event happen again.

### 4.7 - Verify base game files

It's important to verify the game files whenever possible.  You can't know when a problem will occur and it may not yet be obvious that there is an active problem.

From an administrative command line execute the following command 2-3 times:
```
d:\steamcmd\steamcmd.exe +force_install_dir d:\empyrion\ +login anonymous +app_update 530870 validate +quit
d:\steamcmd\steamcmd.exe +force_install_dir d:\empyrion\ +login anonymous +app_update 530870 validate +quit
d:\steamcmd\steamcmd.exe +force_install_dir d:\empyrion\ +login anonymous +app_update 530870 validate +quit
```
The verify fuction may not activate on the first use so it's important to execute the command multiple times until it does.


### 4.8 - Erase existing game to start new game

Steps:
1. From an administrative command line execute the following command: `d:\esm\esm delete-all --doit` This will remove the ramdisk and delete any data from the previous save.  This can fail at times.
2. Use the EAH to apply desired game settings.
3. From an administrative command line execute the following command: `d:\esm\esm.exe ramdisk-install` This process may start the game momentarily to create save game files, then reorganize them for migration to the ramdisk.  Also, you may have to manually help the game exit for the install process to continue.  Look for a button on the game server screen to exit the game.  The rest will usually take care of itself.
4. From an administrative command line execute the following command: `d:\esm\esm.exe ramdisk-setup` This process creates the ramdisk and migrates the save game to it.
5. From an administrative command line execute the following command: `d:\esm\esm.exe check-requirements` Review feedback and follow any guidance.
6. Continue with section 1.15 - Non-Initial game startup

************************************************************************
## 5 - SteamCMD

SteamCMD is used to install and maintain the base game.  One command is all you need for most everything.  This command is used to install, update, and fix the base game.  When used to install or update the game it is important to run the command multiple times until validation occurs.  Not doing so can result in a broken base game.  Validating the base game is recommended for all occasions.  Please know that validating can and will overwrite changed game settings so it always important to use settings files that are not originated in the base game(ergo: rename you settings files for everything but EAH).

Documentation:
https://developer.valvesoftware.com/wiki/SteamCMD

Command:
```
d:\steamcmd\steamcmd.exe +force_install_dir d:\empyrion\ +login anonymous +app_update 530870 validate +quit
d:\steamcmd\steamcmd.exe +force_install_dir d:\empyrion\ +login anonymous +app_update 530870 validate +quit
d:\steamcmd\steamcmd.exe +force_install_dir d:\empyrion\ +login anonymous +app_update 530870 validate +quit
```
************************************************************************
## 6 - Empyrion Admin Helper (EAH)

The EAH is an admin utility (client/server) provided by the game developer.  It runs along side the game and provides features for game management.  The EAH will maintain a database outside of the game and use that data to avoid taxing the game.

Documentation:
https://eah.empyrion-homeworld.net/

Notes:
- While the EAH can change game parameters and effect game play the game is not dependent on the EAH for anything and so what happens in the EAH stays in the EAH.
- The game does not like to wait to for the EAH to do it's thing and so you can have serious problems if the EAH creates some conflict with the game.
- For large games this utility can become extremely slow to use, and so using something like the EWA will make admin life of a large server MUCH more fun.
- This admin tool offers the most admin functions of any other tool.  Doesn't mean it's easier to use, but does do things the EWA cannot(like impersonation).
- It is possible to wipe the db files under the config folder of the EAH.  This will not impact the game.  They will be rebuilt during game operation albeit you will lose the ability to restore things from any history that the EAH recorded.
- It's important to have the correct password with EAH client.  It can act somewhat likes it is fully connected (double green) even when it doesn't have correct access.
- The EAH does have buttons to help fix problems.  You can only use these buttons from the Master EAH, not the client.
  - `Check Database`: this will check the game database for any problems.  Do this when the game is offline.
  - `Reset Red Counter`: clears any data related to the Cheater Check option.  Can be done anytime. EAH data only.
  - `Reset last checked PF ID`: My guess is that playfields are check in some sequence.  Can be done anytime. EAH data only.
  - `Delete all Playfields (EAH)`: Clears playfield data from the EAH.  You will know when to use this.  Can be done anytime. EAH data only.
  - `Check corrupt player files`: Looks for problems with player data and dumps a list of any problems.  Can be done anytime. EAH data only.

************************************************************************
## 7 - Empyrion Server Manager(ESM)

ESM software is a set of scripts that will modify the base game to run in a ramdisk and help you manage the game in that configuration from that point on.  This is an incredibly powerful toolset to elevate game performance and reduce problems.

Documentation:
https://github.com/Vollinger0/esm

Use the following command to see a list of possible functions:

	d:\esm\esm.exe

************************************************************************
## 8 - Empyrion Web Access(EWA)

The EWA is a game mod and web frontend for managing an Empyrion server similar to the EAH.  This mod keeps a database much like EAH, but that database is located on the ramdisk, and over time it can and will take up a good amount of space.  The largest file will likely become HistoryBooks.db and can be removed at any time.  This tool uses port 80.  We don't know of a way to change this.  This is important since upon initial use the first admin to login into the EWA becomes the owner, so don't open port 80 if you can avoid doing so.

We do not know of a way to start the EWA without first starting the game so there may not be an easy way to manage the game without EAH using the EWA alone.  However using the esm to start the game is possible.

DO NOT EVER make any kind of symlink or hard link from the ramdisk back to the main drive as this will reduce the speed of the ramdisk to be the same speed of the hard disk!

Documentation:
https://github.com/GitHub-TC/EmpyrionWebAccess

Steps:
1. Download the latest EWA software archive from the following location: [Here](https://github.com/GitHub-TC/EmpyrionWebAccess/releases)
2. Place the software archive into the following location: `d:\installers`
3. Double click the software archive so that it opens with PeaZip
4. Drag the folder named `EWALoader` to the following location: `d:\empyrion\Content\Mods`
5. Start the game using the Master EAH.
6. Watch for a warning ask you to approve access for the `EmpyrionModWebHost` process.  It is critical that you grant approval.  You may see this multiple times.  At some point it will go away forever.
7. Stop the game.
8. Enter the folder located at `r:\Default Multiplayer\Mods\EWA` and rename the file `xstart.txt` to `start.txt`.  This can be done while the game is running, but may not have full access.
9. Start the game server again using the Master EAH and look into the process manager for a running instance of `EmpyrionModWebHost.exe`.
10. Again watch for a warning ask you to approve access for the `EmpyrionModWebHost` process.  It is critical that you grant approval.  You may see this multiple times.  At some point it will go away forever.
11. Stop the game.
12. Start the game server normally and look into the process manager for a running instance of `EmpyrionModWebHost.exe`.
13. Open a browser on that server to the URL: `http://127.0.0.1/`
14. This will be the first access of the EWA so enter a user name and password for the first user to have access to the EWA.

************************************************************************
## 9 - CMD

While using the command line in Windows you will have at least one bad quirk that can make a mess of things, `Quick Edit Mode`.  Disable this option in the Default settings for all commands lines.  You won't need it for anything, and not disabling it can allow scripts to pause forever when the command window is clicked.

Steps:
1. Click the icon on the top left of the command windows and select `Default`.
2. Clear the option for `Quick Edit Mode`.
3. Click `OK` at the bottom right and close the current command window.

************************************************************************
## 10 - Telnet

Telnet is important for admin tools to function. For this guide the telnet server resides on port 30004.  For the sake of security, we are not openning this port to the world.

Using `Putty` to connect to the server via telnet can provide yet another way to administer an empyrion server.  Once connected an admin can type `HELP` to see a list of available commands.  The list of commands here will be smaller than the list found in-game.

Using the `plys` command will return a list of known and active players.  For the list of active players, the `C-Id` column provides the session id(cl) to address an active play with when using remoteex commands.

There are some commands that cannot be performed on a multi-player server without causing game corruption.  Don't try and change game mode(cm).  Do your research.

Example commands:

	help
	remoteex cl=70 'fbp'
	remoteex cl=51 'give item Token 4013'
	remoteex cl=188 'level u+ 4000'
	remoteex cl=2 'faction rep local Colonists 17500'
	stoppf 'Akua Moon 1'
	saveandexit 15
	saveandexit cancel
	help saveandexit
