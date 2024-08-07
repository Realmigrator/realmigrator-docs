# Graph Reference

### GraphConnection class

All Graph operations are done with the GraphConnection class.

| Script                        | Function                 |
| ----------------------------- | ------------------------ |
| `gkScript::GraphConnection()` | Creates a class instance |

**GraphConnection** offers the following features:

* [Authentication, token acquisition](graph-auth.md)
* OneDrive file operations

```c++
class GraphConnection
{
    // connection state
    bool isConnected();
    string getUserId();

    // Authentication
    string getAuthorizeUrl(string login_hint);
    string getResponseUrl();
    bool setAuthorizationResponse(string ResponseUrl);

    bool hasAdminScope();
    string getAdminConsentUrl();
    string getAdminResponseUrl();
    bool checkAdminConsentResponse(string ResponseUrl);

    // OneDrive
    class Drive
    {
        string id;

        string name;
        string description;
        string url;

        int64 sizeTotal, sizeFree;
        time_t created;
    };

    class DriveItem
    {
        string id;
        string driveId;

        bool isFolder;
        string name;
        string path;
        string downloadUrl;

        int64 size;
        time_t created, modified;
    };

    bool getPersonalDrive(Drive &);
    bool getDriveItem(DriveItem &, string driveId, string path);
    vector<DriveItem> getChildItems(DriveItem);
    bool createChildFolder(DriveItem &newFolder, DriveItem parent);
    bool renameMoveItem(DriveItem &, string newName, string newParentItemId);
    bool deleteItem(DriveItem);

    bool uploadFile(DriveItem &, DriveItem parent, string fileName, string localFilePath);
    bool uploadFile(DriveItem &, string localFilePath);
    bool uploadFile(DriveItem &, string localFilePath, string sessionUrl);

    bool setCallback(callbackObject);
};
```

### OneDrive file operations

In order to work with OneDrive, you need a drive object.

| Script                                       | Function                                      |
| -------------------------------------------- | --------------------------------------------- |
| `GraphConnection::getPersonalDrive(Drive &)` | Get the primary OneDrive assigned to the user |

All further file operations work on DriveItems, which could be either folders or files.

| Script                                                                     | Function                                                                             |
| -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| `GraphConnection:: getDriveItem(DriveItem &, string driveId, string path)` | <p>Opens a DriveItem.<br>To open the root directory of a drive, use the path "/"</p> |
| `GraphConnection:: getChildItems(DriveItem &)`                             | Get directory contents                                                               |

**List directory sample**

```c++
def listDirectory(path) {
    var graph=gkScript.GraphConnection();

	if (!graph.isConnected()) {
	    print("no graph connection.");
		return;
	}

	var drive = graph.Drive();
	if (!graph.getPersonalDrive(drive)) {
	    print("no personal drive.");
		return;
	}

	print("Drive info:");
	print("   id:          " + drive.id);
	print("   name:        " + drive.name);
	print("   description: " + drive.description);
	print("   url:         " + drive.url);
	print("   sizeTotal:   " + to_string(drive.sizeTotal));
	print("   sizeFree:    " + to_string(drive.sizeFree));
	print("   created:     " + gkScript.timeString(drive.created));

	var root = graph.DriveItem();
	if (!graph.getDriveItem(root, drive.id, path)){
	    print("Could not open path " + path);
		return;
	}

	var list = graph.getChildItems(root);

	print("Content of '" + root.path + "':");

	for (entry : list){
	    var str = "   ";
		if (entry.isFolder) {str += "<DIR> ";}
		else                {str += "      ";}
		str += entry.name;
		print (str);
    	print("            id:       " + entry.id);
   	    print("            size:     " + to_string(entry.size));
	    print("            created:  " + gkScript.timeString(entry.created));
	    print("            modified: " + gkScript.timeString(entry.modified));
	}
	print(to_string(list.size()) + " items total.");
}

listDirectory("/");
```

| Script                                                                                 | Function                          |
| -------------------------------------------------------------------------------------- | --------------------------------- |
| `GraphConnection::renameMoveItem(DriveItem &, string newName, string newParentItemId)` | Rename and/or move DirectoryItems |
| `GraphConnection::renameMoveItem(item, "newName", "")`                                 | Will rename item to "newName"     |
| `GraphConnection::renameMoveItem(item, "", folderXY.id)`                               | Will move item to folderXY        |
| `GraphConnection::renameMoveItem(item, "newName", folderXY.id)`                        | Will move and rename item.        |

**Basic file operations sample**

```c++
var graph=gkScript.GraphConnection();

// check connection
if (!graph.isConnected()) {
	print("no graph connection.");
	return;
}

// get onderive drive object
var drive = graph.Drive();
if (!graph.getPersonalDrive(drive)) {
	print("no personal drive.");
	return;
}

// get onedrive root folder
var root = graph.DriveItem();
if (!graph.getDriveItem(root, drive.id, "/")){
	print("no root.");
	return;
}
print("Opened root " + root.path);

var folderA = graph.DriveItem();
folderA.name = "folderA";
if (graph.createChildFolder(folderA, root)){
    print("Created folder " + folderA.path);
}

var folderB = graph.DriveItem();
folderB.name = "the folder b";
if (graph.createChildFolder(folderB, folderA)){
   print("Created folder " + folderB.path);
}

// move folderB to root
if (graph.renameMoveItem(folderB, "", root.id)){
   print("Moved folderB to " + folderB.path);
}

// cleanup
graph.deleteItem(folderA);
graph.deleteItem(folderB);

```

### File upload

| Script                                                                                              | Function                                                      |
| --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| `GraphConnection::uploadFile(DriveItem &, DriveItem parent, string fileName, string localFilePath)` | Will upload new files                                         |
| `GraphConnection::uploadFile(DriveItem &, std::string localFilePath)`                               | Will upload existing files                                    |
| `GraphConnection::uploadFile(DriveItem &, std::string localFilePath, std::string sessionUrl)`       | Will upload files with an externally generated Upload session |

All upload methods overwrite existing content, and can use a callback object to report upload progress.

| Script                                         | Function                   |
| ---------------------------------------------- | -------------------------- |
| `GraphConnection::setCallback(callbackObject)` | Will set a callback object |

```c++
class callbackObject
{
    void progressCallback(int sizeCompleted, int sizeTotal);
}
```

**File upload sample**

```c++
var graph=gkScript.GraphConnection();

// check connection
if (!graph.isConnected()) {
	print("no graph connection.");
	return;
}

class Progress
{
    def Progress(int Sleeptime, name) {
	    this.sleepTime = Sleeptime;
		this.name = name;
	}

	def progressCallback(int sizeCompleted, int sizeTotal){
	    print(this.name + "   [ " + to_string(sizeCompleted) + " / " + to_string(sizeTotal) + " ]");

		// sleep if we need to upload more than 4 k...
		if (sizeTotal - sizeCompleted > 4096) {gkScript.sleep(this.sleepTime);}
	}
	var sleepTime;
	var name;
}

// create local file
var file = gkScript.expandString("%temp%\\upload.txt");
gkScript.saveFile(file, "hello world!", true);

// get onderive drive object
var drive = graph.Drive();
if (!graph.getPersonalDrive(drive)) {
	print("no personal drive.");
	return;
}

// get onedrive root folder
var root = graph.DriveItem();
if (!graph.getDriveItem(root, drive.id, "/")){
	print("no root.");
	return;
}
print("Opened root " + root.path);

// upload...

var progress = Progress(2000, file + " -> /uploadtest.txt")
graph.setCallback(progress);

var upload = graph.DriveItem();

if (graph.uploadFile(upload, root, "uploadtest.txt", file)) {
    print("Successfully uploaded ");// + file + " to " + upload.path);
}

// cleanup
graph.deleteItem(upload);
gkScript.deleteFile(file);
```
