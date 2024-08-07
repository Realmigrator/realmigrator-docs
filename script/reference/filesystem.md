# File System

## gkScript main object

The global gkScript object offers the following filesystem functions:

```c++
class gkScript
{
    ...

    // Local filesystem
    bool loadFile(string path, string content);
    bool loadFile(string path, string content, size_t offset, size_t size);
    string readStdin();
    bool saveFile(string path, string content, bool overwrite = false);
    bool saveFile(string path, string content, size_t offset);
    bool setFileSize(string path, size_t size);
    bool setFileAttributes(string path, int attributes);
    bool copyFile(string sourcePath, string targetPath, bool overwrite);
    bool moveFile(string sourcePath, string targetPath);
    bool deleteFile(string path, bool recycleBin = false);
    bool createDirectory(string path);
    bool deleteDirectory(string path);

    vector<DirectoryEntry> listDirectory(string path);
    bool getFileInformation(string path, DirectoryEntry &info);


    map<string, value> getFileSummary(string path);
    map<string, string> enumDrives();

    bool addNetworkConnection(string path, string user, string password, string localDrive = "");
    bool deleteNetworkConnection(string path, bool force = false);
    ...
};
```

## Directory Content

| Script                                 | Function                                      |
| -------------------------------------- | --------------------------------------------- |
| `gkScript::listDirectory(string path)` | Returns an array of **DirecotyEntry** objects |

DirectoryEntry:

```c++
class DirectoryEntry
{
    isFolder;             // bool
    name;                 // string
    path;                 // string

    attributes;           // uint32_t
    size;                 // int64_t
    created, modified;    // time_t

}
```

**listDirectory** sample:

```shell
eval> var list=gkScript.listDirectory("c:");
eval> for (entry : list){print(entry.name);}
$Recycle.Bin
$WINDOWS.~BT
bootmgr
BOOTNXT
Config.Msi
Documents and Settings
MATS
OneDriveTemp
pagefile.sys
PerfLogs
Program Files
Program Files (x86)
ProgramData
Recovery
swapfile.sys
System Volume Information
Users
Windows
```

## Ranged read/write operations

| Script                                            | Function                                                                                             |
| ------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `gkScript::loadFile(path, content, offset, size)` | Can be used to load a part of a file                                                                 |
| `gkScript::getFileInformation(path, &info)`       | Can be used to retrieve the file size                                                                |
| `gkScript::saveFile(path, content, offset)`       | <p>Can be used to write a part of the file.<br>The write size is the size of the content string.</p> |

## File summary

| Script                                  | Function                                                                                                                                                                                                                   |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gkScript::getFileSummary(string path)` | <p>Returns a map of key:value pairs with summary information about the file.<br>Supported file types are:<br>- Structured storage file with summary information (.msi, .msg, .doc, .xls, thumps.db,...)<br>- PST files</p> |

Dump MSI installer cache:

```c++
var dir=gkScript.listDirectory(gkScript.expandString("%windir%\\installer"));

var i=0;

for (entry : dir) {
	if (!entry.isFolder) {
		var info=gkScript.getFileSummary(entry.path);
		if (!is_var_undef(info["MSI Package Code"])) {
			++i;
			print(entry.name + "   (" + entry.path + ")");
			print("    Package Code " + to_string(info["MSI Package Code"]));
			if (!is_var_undef(info["Subject"])) {print("    " + to_string(info["Subject"]));}
		}
	}
}


print(to_string(i) + " Packages found.");
```

## Read console input

| Script                         | Function                                      |
| ------------------------------ | --------------------------------------------- |
| `string gkScript::readStdin()` | Reads from stdin until end of file is reached |

Console input sample:

```
>echo Hello %username%|chaiCmd.exe -c "var in=gkScript.readStdin(); print(\"input: \" + in);"
input: Hello christoph
```

## Read console input

| Script                                                             | Function                                                                                                                                                                                                                                      |
| ------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gkScript::addNetworkConnection(path, user, password, localDrive)` | <p>Creates a network connection.<br>If user and password are empty, the current user context is used to connect.<br>If localDrive is present, the connection is permanent and mapped to the drive.<br>Use the format "X:" for localDrive.</p> |
| `gkScript::deleteNetworkConnection(path, force)`                   | <p>Removes a network connection.<br>Path can be eithe a UNC path or a local drive.</p>                                                                                                                                                        |
