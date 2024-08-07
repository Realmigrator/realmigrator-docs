# File

The File Module contains options to configure the searches of files which have to migrate. RealMigrator searches these files and copies them to a defined destination.

![](../.gitbook/assets/Files1.PNG)

## Upload Store

Upload stores always support basic file operations (upload file, create directory). They can support delete operations on files and directories to support proper synchronization.\
In addition, they can support blockmode uploads, which enable differential uploads based on hash maps.

![](../.gitbook/assets/Files2.PNG)

| Setting         | Explanation                                                                                                                                                                                                                                                                        |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Type            | <p><strong>File</strong>: Writes to a file back-end, either local or on a file server<br><strong>OneDrive</strong>: Stores to the user's OneDrive via a RealMigrator server Graph API<br><strong>Blob Storage</strong>: Stores a blob storage via RealMigrator server Blob API</p> |
| Root path       | Default root path                                                                                                                                                                                                                                                                  |
| Diff file size  | Default size are 4 MB. When a file exceeding this size it will be uploaded in blocks. It is possible to define a own size                                                                                                                                                          |
| Diff block size | When a file has to uploaded in blocks the default size is 1 MB                                                                                                                                                                                                                     |
| Path blacklist  | Paths which are listed in a blacklist are excluded from all catalogs and will not be part of migrations (see \[Well-known folder] (\{{< ref "modules/wellknownfolder/\_index.md" >\}}) for folder (paths) list)                                                                    |

## Catalog

A catalog defines which files RealMigrator has to migrate and where to find them.

![](../.gitbook/assets/Files3.PNG)

| Setting              | Explanation                                                                                                                                                                                                                                                                               |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name                 | Name of the catalog                                                                                                                                                                                                                                                                       |
| Path                 | Concrete path and place where RealMigrator will find the files (see \[Well-known folder] (\{{< ref "modules/wellknownfolder/\_index.md" >\}}) for folder (paths) list)                                                                                                                    |
| Remote path          | Defined destination for **Path**                                                                                                                                                                                                                                                          |
| Filter               | RealMigrator migrates the listed file formats                                                                                                                                                                                                                                             |
| Blacklist            | When activated RealMigrator will migrate all file formats except the formats from **Filter**                                                                                                                                                                                              |
| Synchronization Type | <p><strong>One-Move Sync</strong>: Copies remote storage and removes them if they are deleted in the source path<br><strong>Copy</strong>: Copies changes but does not remove deleted files<br><strong>Move</strong>: Removes local file after they were copied to the remote storage</p> |

### Catalog Order

Use the **up and down buttons** to change the order of catalogs

![](../.gitbook/assets/Files4.PNG)

### Create and Delete catalogs

At the bottom of the page, there are buttons to create new catalogs or delete existing ones

![](../.gitbook/assets/Files5.PNG)

* To add further catalogs click the button **Catalog**
* To delete the last created catalog click the button **Last Catalog**
* To delete all of them, click the button **All**

## Log level and Reset code

| Setting    | Explanation                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Log level  | Available log levels are **Error**, **Info** and **Debug**.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Reset Code | A client is always checking the own configurations after a defined period of time. This means that the client compares the selected client settings with the selected server settings. In addition the client connect to the server based on \[Module loop delay]\(\{{< ref "beginning/serverconfig/\_index.md#global-client-configuration" >\}}), a defined period of time. By this time value the client is starting a Module loop and the client is checking the Reset code. If there is a Reset code change the client will takes over the new configuration. If there is no change, the client will still use the existing configuration. |
| Power mode | If this mode is activated, power down is prevented during migration.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |

![](../.gitbook/assets/Files6.PNG)

## Script

RealMigrator offers a default script for File. But it is possible to edit the existing script or to write a new script.

![](../.gitbook/assets/Files7.PNG)

## Reset Button

![](../.gitbook/assets/resetbutton.PNG)

If you click the **Reset** button in the **Config** tab or in the **Script** tab, all your entries will be reset to the RealMigrator default values.

To accept the default values, you have to click **Save** at the top of the page.
