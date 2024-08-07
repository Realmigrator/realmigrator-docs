# Folder Lists

The following tables contain paths which are part of Upload Stores and Catalogs in the [File module](file.md) and in the [PST module](pst.md). Depending on Custom module settings (see [Module Configuration](module-configuration.md)) the paths are a part of [Custom module](custom.md) as well.

## General Folder List

This list contains paths for general folders, like Documents, Music or Desktop.

| Name                     | Path                                                                 | ID                             |
| ------------------------ | -------------------------------------------------------------------- | ------------------------------ |
| Documents                | %USERPROFILE%\Documents                                              | FOLDERID\_Documents            |
| Links                    | %USERPROFILE%\Links                                                  | FOLDERID\_Links                |
| Music                    | %USERPROFILE%\Music                                                  | FOLDERID\_Music                |
| Contacts                 | %USERPROFILE%\Contacts                                               | FOLDERID\_Contacts             |
| Desktop                  | %USERPROFILE%\Desktop                                                | FOLDERID\_Desktop              |
| Camera Roll              | %USERPROFILE%\Pictures\Camera Roll                                   | FOLDERID\_CameraRoll           |
| Pictures                 | %USERPROFILE%\Pictures                                               | FOLDERID\_Pictures             |
| Account Pictures         | %APPDATA%\Microsoft\Windows\AccountPictures                          | FOLDERID\_AccountPictures      |
| Videos                   | %USERPROFILE%\Videos                                                 | FOLDERID\_Videos               |
| Downloads                | %USERPROFILE%\Downloads                                              | FOLDERID\_Downloads            |
| Favorites                | %USERPROFILE%\Favorites                                              | FOLDERID\_Favorites            |
| Cookies                  | %APPDATA%\Microsoft\Windows\Cookies                                  | FOLDERID\_Cookies              |
| Temporary Internet Files | %LOCALAPPDATA%\Microsoft\Windows\Temporary Internet Files            | FOLDERID\_InternetCache        |
| History                  | %LOCALAPPDATA%\Microsoft\Windows\History                             | FOLDERID\_History              |
| Libraries                | %APPDATA%\Microsoft\Windows\Libraries                                | FOLDERID\_Libraries            |
| Network Shortcuts        | %APPDATA%\Microsoft\Windows\Network Shortcuts                        | FOLDERID\_NetHood              |
| Printer Shortcuts        | %APPDATA%\Microsoft\Windows\Printer Shortcuts                        | FOLDERID\_PrintHood            |
| Programs                 | %APPDATA%\Microsoft\Windows\Start Menu\Programs                      | FOLDERID\_Programs             |
| Quick Launch             | %APPDATA%\Microsoft\Internet Explorer\Quick Launch                   | FOLDERID\_QuickLaunch          |
| Recent Items             | %APPDATA%\Microsoft\Windows\Recent                                   | FOLDERID\_Recent               |
| Ringtones                | %LOCALAPPDATA%\Microsoft\Windows\Ringtones                           | FOLDERID\_Ringtones            |
| Searches                 | %USERPROFILE%\Searches                                               | FOLDERID\_SavedSearches        |
| SendTo                   | %APPDATA%\Microsoft\Windows\SendTo                                   | FOLDERID\_SendTo               |
| Templates                | %APPDATA%\Microsoft\Windows\Templates                                | OLDERID\_Templates             |
| Start Menu               | %APPDATA%\Microsoft\Windows\Start Menu                               | FOLDERID\_StartMenu            |
| Startup                  | %APPDATA%\Microsoft\Windows\Start Menu\Programs\StartUp              | FOLDERID\_Startup              |
| Administrative Tools     | %APPDATA%\Microsoft\Windows\Start Menu\Programs\Administrative Tools | FOLDERID\_AdminTools           |
| Application Shortcuts    | %LOCALAPPDATA%\Microsoft\Windows\Application Shortcuts               | FOLDERID\_ApplicationShortcuts |
| Temporary Burn Folder    | %LOCALAPPDATA%\Microsoft\Windows\Burn\Burn                           | FOLDERID\_CDBurning            |

## Common Folder List

This list contains paths which related to all user profiles, like templates or administrative tools.

| Name                        | Path                                                                         | ID                         |
| --------------------------- | ---------------------------------------------------------------------------- | -------------------------- |
| Common Administrative Tools | %ALLUSERSPROFILE%\Microsoft\Windows\Start Menu\Programs\Administrative Tools | FOLDERID\_CommonAdminTools |
| Common OEM Links            | %ALLUSERSPROFILE%\OEM Links                                                  | FOLDERID\_CommonOEMLinks   |
| Common Programs             | %ALLUSERSPROFILE%\Microsoft\Windows\Start Menu\Programs                      | FOLDERID\_CommonPrograms   |
| Common Start Menu           | %ALLUSERSPROFILE%\Microsoft\Windows\Start Menu                               | FOLDERID\_CommonStartMenu  |
| Common Startup              | %ALLUSERSPROFILE%\Microsoft\Windows\Start Menu\Programs\StartUp              | FOLDERID\_CommonStartup    |
| Common Templates            | %ALLUSERSPROFILE%\Microsoft\Windows\Templates                                | FOLDERID\_CommonTemplates  |

## OneDrive Folder List

The following list contains paths for OneDrive migrations.

| Name               | Path                             | ID                            |
| ------------------ | -------------------------------- | ----------------------------- |
| OneDrive           | %USERPROFILE%\OneDrive           | FOLDERID\_SkyDrive            |
| OneDrive Documents | %USERPROFILE%\OneDrive\Documents | FOLDERID\_SkyDriveDocuments}, |
| OneDrive Pictures  | %USERPROFILE%\OneDrive\Pictures  | FOLDERID\_SkyDrivePictures    |
