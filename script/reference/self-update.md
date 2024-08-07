# Self Update Reference

## Installation

gkScript methods:

* Install itself to the machine and set an auto-run key to start on every logon.
* Self-update an installed client.

gkScript also ensures that the installed client is running as a single instance per logon session.

| Script                                                   | Function                                                                                                                                                              |
| -------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `string gkScript::getVersion()`                          | Version string of the current client-xxx.exe / chaiCmd-xxx.exe module                                                                                                 |
| `string gkScript::moduleFilename(bool fullPath = false)` | Module file name of the current client-xxx.exe / chaiCmd-xxx.exe module.                                                                                              |
| `int gkScript::getPID()`                                 | Process ID of the current instance.                                                                                                                                   |
| `bool gkScript::isInstalled()`                           | <p>- Returns <strong>false</strong> for chaiCmd-xxx.exe<br>- Returns <strong>true</strong> if client-xxx.exe is present in the target path</p>                        |
| `bool gkScript::isInstalledInstance()`                   | <p>- Returns <strong>false</strong> for chaiCmd-xxx.exe<br>- Returns <strong>true</strong> if client-xxx.exe was started from install-path</p>                        |
| `bool gkScript::setupClient(bool install)`               | <p>- Will install itself or deinstall the client<br>- Will fail if current module is chaiCmd-xxx.exe<br>- Will fail of <code>isInstalledInstance() == true</code></p> |
| `string gkScript::installPath()`                         | Returns the installation path                                                                                                                                         |

Sample install code:

```c++
def checkInstallation(serverVersion, app) {
    if (gkScript.isInstalledInstance()) {
        gkScript.logDebug("we are the installed client, checking auto-update ...");

        if (gkScript.getVersion() != serverVersion) {
        var path = gkScript.expandString("%temp%\\" + gkScript.moduleFilename());
            if (app.loadClient(path)) {
                gkScript.logDebug("starting new client for self-update ...");
                gkScript.runProcess(path);

                exit(0);
            }
            else {
                gkScript.logError("Failed to load: " + path);

                exit(2);   // comment this line if the outdated version should run as fallback..
            }
        }
        return true;
    }
    else {
        if (gkScript.isClientInstalled()) {
            gkScript.logDebug("Client is installed, nothing to do.");
            exit (0);
        }
        else {
		    if ("chaicmd" == gkScript.toLower(gkScript.moduleFileName().substr(0,7))) {
                gkScript.logDebug("Running chaiCmd with bootstrapper.");
				return true;
			}

            gkScript.sleep(10000);  // wait for main process to end.
            if (!gkScript.installClient()) {
                gkScript.logDebug("Failed to install client!");
                exit(2);
            }
        }
    }

    gkScript.logDebug("starting new client for self-update ...");
    gkScript.runProcess(gkScript.installPath() + gkScript.moduleFilename());

    exit (0);
}
```
