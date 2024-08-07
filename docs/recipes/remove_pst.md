# Remove PSTs

To remove PSTs from outlook usw the code below. Copy it to a global configuration script. In the **afterFinalize()** function, call **removePstFromOutlook(true)** or **removePstFromOutlook(false)**.\
If the boolean parameter is true, PST files are moved to the recycle bin.

```c++
def afterFinalize() {
    removePstFromOutlook(true);
}


def isVirtualOutlook() {
    var reg=gkScript.openRegistry();

    return (!reg.readString("HKEY_LOCAL_MACHINE\\SOFTWARE\\WOW6432Node\\Microsoft\\Office\\16.0\\Common\\InstallRoot\\Virtual\\VirtualOutlook").empty() || !reg.readString("HKEY_LOCAL_MACHINE\\SOFTWARE\\WOW6432Node\\Microsoft\\Office\\15.0\\Common\\InstallRoot\\Virtual\\VirtualOutlook").empty());
}

def removePstFromOutlook(bool trash) {
    var progid = "Outlook.Application";
    var ol;

    if (isVirtualOutlook()) {
        ol = gkScript.createComObject(progid, false);
    } else {
        ol = gkScript.getComObject(progid);
        if(!ol.connected()) {
            gkScript.logInfo("failed to get outlook application, starting outlook..");

            var reg=gkScript.openRegistry();

            var path = reg.readString("HKEY_CLASSES_ROOT\\CLSID\\{0006F03A-0000-0000-C000-000000000046}\\LocalServer32\\")
            if (path.empty()) {
                path = reg.readString("HKEY_CLASSES_ROOT\\Wow6432Node\\CLSID\\{0006F03A-0000-0000-C000-000000000046}\\LocalServer32\\")
            }
            if (path.empty()) {
                path = "C:\\Program Files (x86)\\Microsoft Office\\Root\\Office16\\OUTLOOK.EXE";            
            }

            var b = path.find("\"");
            var e = path.rfind("\"");
            if (b < e) {path = path.substr(b+1, e-b-1);}

            gkScript.logInfo("Found outlook at path '" + path + "'. Execute it...");

            var handle = gkScript.runProcess(path);
            gkScript.sleep(15 * 1000);

            var hNotepad = gkScript.runProcess("notepad.exe");
            gkScript.sleep(5 * 1000);

            ol = gkScript.getComObject(progid);
            gkScript.terminateProcess(hNotepad, 0);
        }        
    }

    if(!ol.connected()) {
        gkScript.logError("failed to get outlook application");
        return false;
    }

    var session = ol.getProp("Session");
    if(!session.connected()) {
        gkScript.logError("failed to get session");
        return false;
    }

    var stores = session.getProp("Stores");
    if(!stores.connected()) {
        gkScript.logError("failed to get stores");
        return false;
    }

    var count = stores.getProp("Count");

    for(var i = 1; i <= count; ) {
        var s = stores.invoke("Item", DISPATCH_PROPERTYGET, [i]);
        if(s.connected()) {
            if(!s.getProp("IsDataFileStore") || (to_string(s.getProp("ExchangeStoreType")) != "3")) {
                i += 1;
                continue;
            }

            var path = s.getProp("FilePath");
            if(path.find(".pst") == -1) {
                i += 1;
                continue;
            }

            gkScript.logInfo("Store " + to_string(i) + " is a data file store type 3 located at " + path + ". Remove it...");

            var folder = s.invoke("GetRootFolder", []);
            if(folder.connected()) {
                session.invoke("RemoveStore", [folder]);
                count -= 1;
                if (trash) {
                  gkScript.deleteFile(path, true);
                }
            } else {
                gkScript.logInfo("Failed to get root folder");
                i += 1;
            }
        }
    }

    ol.disconnect();

    return true;
}
```
