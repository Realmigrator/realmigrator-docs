# Registry Reference

## gkScript Main Object

| Script                     | Function                      |
| -------------------------- | ----------------------------- |
| `gkScript::openRegistry()` | Will create a registry object |

## gkScript Registry Object

```c++
enum RegistryTypes {
    REG_DWORD,
    REG_BINARY,
    REG_SZ,
    REG_EXPAND_SZ,
    REG_MULTI_SZ,
    REG_LINK,
    REG_NONE,
    REG_DWORD_LITTLE_ENDIAN,
    REG_DWORD_BIG_ENDIAN,
    REG_QWORD,
    REG_QWORD_LITTLE_ENDIAN,
};

class Registry
{
    int readDWORD(string path);                     // returns 0 on error
    bool writeDWORD(string path, int val);

    string readString(string path);                 // returns "" on error
    bool writeString(string path, string val);

    bool readValue(string path, string &value, RegistryTypes &type);
    bool writeValue(string path, string value, RegistryTypes type);

    bool deleteValue(string path);
    bool createKey(string path);
    bool deleteKey(string path);

    vector<string> enumSubkeys(string path);
    vector<string> enumValues(string path);
};
```

| Registy paths and root keys                                                                                                                                                                                                                                            |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| For all operations, use fully qualified path strings.                                                                                                                                                                                                                  |
| Paths strings to values include the value name.                                                                                                                                                                                                                        |
| To address the the key's unnamed or default value, e.g. "HKLM\Software\\" for the default value of the key "HKEY\_LOCAL\_MACHINE\Software".                                                                                                                            |
| <p>Paths must start with one of the following default keys, or it's appreviation:<br>HKCR or HKEY_CLASSES_ROOT<br>HKCU or HKEY_CURRENT_USER<br>HKLM or HKEY_LOCAL_MACHINE<br>HKU or HKEY_USERS<br>HKCC or HKEY_CURRENT_CONFIG<br>HKPD or HKEY_PERFORMANCE_DATA<br></p> |
| The Registry always opens the 64-bit registry paths. To use the 32-bit path you need explicit addressing, e.g. HKEY\_LOCAL\_MACHINE\SOFTWARE\Wow6432Node                                                                                                               |

#### Strings

| Script                 | Function                                                                                            |
| ---------------------- | --------------------------------------------------------------------------------------------------- |
| `Registry::readString` | Will return UTF8 strings                                                                            |
| `Registry::readValue`  | Will return UTF16 strings for all registry string types (REG\_SZ, REG\_EXPAND\_SZ, REG\_MULTI\_SZ). |

Convert the string returned by `Registry::readValue` with `gkScript.utf16ToUtf8`

#### Registry sample

```c++
// Dump installed software from registry

def dumpSoftware(string user) {
    var reg=gkScript.openRegistry();

	var path = "HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Installer\\UserData\\" + user + "\\Products";
	var software  = reg.enumSubkeys(path);

	for (s:software) {
	   var sPath = path + "\\" + s + "\\InstallProperties\\"
	   var name = reg.readString(sPath + "DisplayName");
	   var version = reg.readString(sPath + "DisplayVersion");

	   var t=REG_NONE;
	   var uninst = "";
	   reg.readValue(sPath + "UninstallString", uninst, t);

	   if (!name.empty()) {
	      print("    " + name);
		  print("        version: " + version);
		  print("        uninstall info: " + to_string(t) + " '" + gkScript.utf16ToUtf8(uninst) + "'");
	   }
	}
}


var reg=gkScript.openRegistry();

var users = reg.enumSubkeys("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Installer\\UserData")


for (u:users) {
  print("Software installed for user " + u);
  dumpSoftware(u);
}

```
