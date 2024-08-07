# Server Module Scripting

The Server Module offers a Script breakout when the client reaches finish state. In order to be called, you need to implement a _**onServerFinish(app)**_ function. The _**app**_ object is a _**ScriptApplication**_ object that offers methods to interact with the finishing client:

```c++
class ScriptApplication {
	bool loadModuleData(moduleName, jsonObject);    // load module data into jsonObject
    bool loadBlobData(path, data, offset, size);    // load a Blob store chunk into data
    bool getBlobSize(path, size);                   // get the size of a Blob

    string onedriveSession(path);                   // create an upload session url in the users onedrive
    bool onedriveUploadChunk(session, data, offset, totalsize);
                                                    // upload a chunk to the user's onedrive
    bool onedriveCancelUpload(session);             // cancel an upload

    setProgress(progressData);                      // Report progress. The progressData Map object should
                                                    // have "total" and "completed" int members and a "message" string.
    setError(msg);                                  // Set an error message
}
```

The following script will load the user's PST module data, iterate through the uploaded PSTs and copy them into its Onedrive.

```c++
//
// config_server script
//

def copyBlobToOnedrive(app, file) {
    var size = 0ull;
  	var blockSize = 4 * 1024 * 1024;
  	var uploadSize;
  	var offset;
  	var data;
  
    if (!app.getBlobSize(file["blobPath"], size))  {
      app.setError("Failed to get Blob size for '" + file["blobPath"] + "'");
  	  file["status"] = "failed";
      return false;
    }
  	file["size"] = size;
  
  	var session = app.onedriveSession(file["onedrivePath"]);
  	if (session.empty()) {
      app.setError("Failed to open Onedrive Session for '" + file["onedrivePath"] + "'");
  	  file["status"] = "failed";
      return false;
    }

  	for (offset = 0ull; offset < size; offset += uploadSize) {
      	if (offset + blockSize <= size) {uploadSize = blockSize;}
      	else {uploadSize = size - offset;}

   		data = "";
      	if (!app.loadBlobData(file["blobPath"], data, offset, uploadSize)) {
          	app.setError("Failed to read chunk '" + file["blobPath"] + "' (" + to_string(offset) + ":" + to_string(uploadSize) + ")");
          	app.onedriveCancelUpload(session);
            file["status"] = "failed";
          	return false;
        }
      
      	if (!app.onedriveUploadChunk(session, data, offset, size)) {
          	gkScript.logInfo("Failed to upload chunk - retrying after sleep.");
          	gkScript.sleep(30000);
            if (app.onedriveUploadChunk(session, data, offset, size)) {
              	gkScript.logInfo("Retry succeeded.");
            } else {
                app.setError("Failed to upload chunk '" + file["onedrivePath"] + "# (" + to_string(offset) + ":" + to_string(data.size()) + "/" + to_string(size) + ")");
                app.onedriveCancelUpload(session);
                file["status"] = "failed";
                return false;
            }
        }
    }
  
  	file["status"] = "finished";
  
  	return true;
}

def onServerFinish(app) {
    var pst;
  	var moduleName = "pst";
  	var progress = ["total": 0, "completed": 0, "message": "Initializing PST copy...", "files": Map()];
  	var &files = progress["files"];
	app.setProgress(progress);

    if (!app.loadModuleData(moduleName, pst)) { return false; }

    for (c:pst["catalogs"]) {
        for (f:c.second["files"]) {
            if (f.second["state"] != 4) {
                gkScript.logError("Cannot migrate PST '" + f.first + "': not synced (" + to_string(f.second["state"]) + ")");
                continue;
            }

          	var file = Map();
          	file["blobPath"] = moduleName + f.second["remotePath"];
            file["blobSize"] = f.second["size"];
            file["blobDate"] = f.second["modified"];
          	file["onedrivePath"] = f.second["remotePath"] + "_backup";
          	files[f.first] = file;
            gkScript.logInfo("PST: " + to_json(file));
        }
    }

	progress["total"] = files.size()

	for (f:files) {
      	progress["message"] = "Copying " + f.second["blobPath"];
		app.setProgress(progress);

        if (copyBlobToOnedrive(app, f.second)) {
        	progress["completed"] += 1;
        }
    }

   	progress["message"] = "Finished PST copy.";
	app.setProgress(progress);

	return true;
}
```
