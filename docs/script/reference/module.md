# Module Reference

gkScript is a chai module that offers different features:

* Subscripts
* [HTTP Reference](http.md)
* [Web browser](browser.md)
* [File system](filesystem.md)
* [Graph Reference](graph-reference.md)
  * [MS Graph Authentication (and token acquisition)](graph-auth.md)
* [Hash Reference](hash.md)
* [Registry Reference](registry.md)
* [COM](com.md)
* [Self Update Reference](self-update.md)

## gkScript main object

All gkScript module features are available through a global gkScript object:

```c++
class gkScript
{
    string executionType();    // returns "exe", "service" or "docker"
    // Helper functions
    time_t time();
    time_t utcOffset();
    string timeString(time_t val, bool convertToLocalTime = true);
    int tick();    // tick-count in milliseconds
    string expandString(string);
    string getKnownFolder(string);

    sleep(milliseconds);
    string getCommandLine();
    vector<string> getArgList();
    string getNetworkId(bool appendGateway);
    int getConnectionCost(string host);
    string getUserSID();
    string getUserUPN();
    string getMailAddress();
    bool waitForNetwork(milliseconds);

    map<string, string> getUrlParameter(string url);
    string urlEncode(string);
    string pathEncode(string);    // same as urlEncode, but does not encode '/'
    string hexEncode(string);
    string base64Encode(string);
    string base64Decode(string);
    string toLower(string);
    string utf16ToUtf8(string);
    string utf8ToUtf16(string);
    string jScriptStringEncode(string);
    bool matchWildCard(string pattern, string test);
    int getCodePage();

    string randomFill(size);

    bool cryptProtect(string in, bool toLocalMachine, string &out);
    bool cryptUnprotect(string in, string &out);
    bool exportCertStore(string path, string password);

    // Settings store
    bool setString(Key, Value);
    string getString(Key);
    bool setInt(Key, Value);
    integer getInt(Key);

    // Logging functions
    LogLevel getLogLevel();
    setLogLevel(level);

    logError(string);
    logInfo(string);
    logDebug(string);

    bool exportLog(string);
    setPowermode(bool on);     // on: prevent idle power off

    // WMI
    map<string Name, Value> wmiQuery(string Path, string Query);

    // Processes and Threads
    bool shellExecute(hwnd, string verb, string path);
    int executeProcess(string commandLine, string stdin, string &stdout, string &stderr, unsigned int timeout);
    handle runProcess(string commandLine);
    handle openProcess(int processId);
    bool wait(handle, milliseconds);             // can wait for processes, threads, events
    bool terminateProcess(handle, int exitCode);
    wmQuitProcess(int processId);    // send WM_QUIT to all windows of that process
    int getExitCode(handle);
    handle runThread(string chaiCommand);
    bool terminateThread(handle, int exitCode);

    // Events
    handle createEvent();
    setEvent(handle);
    resetEvent(handle);

    // script tool
    ScriptTool ScriptTool(bool with_gkScript);
};
```

## Helper functions

Most helpers are easy to understand. See below for some samples:

### urlEncode - getUrlParameter

```shell
eval> var url = "https://localhost/response?company=" + gkScript.urlEncode("Glück & Kanja") + "&message=" + gkScript.urlEncode("Schöne Grüße");
https://localhost/response?company=Gl%81ck+%26+Kanja&message=Sch%94ne+Gr%81%E1e
eval> var param=gkScript.getUrlParameter("https://localhost/response?company=Gl%81ck+%26+Kanja&message=Sch%94ne+Gr%81%E1e");
[<company, Glück & Kanja>, <message, Schöne Grüße>]
eval> for (p : param) {print(p.first + ": " + p.second);}
company: Glück & Kanja
message: Schöne Grüße
```

### matchWildCard

| Script                                                      | Function                                                                                                         |
| ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `bool gkScript::matchWildCard(string pattern, string test)` | <p>Tests for a case insensitive match of test against pattern.<br>Pattern may contain wild cards '*' and '?'</p> |

### Network helper functions

| Script                                   | Function                                                                                                                                                       |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gkScript::waitForNetwork(milliseconds)` | <p>Wait until an internet connection is established and returns <strong>true</strong>.<br>If a timeout occurs, the function returns <strong>false</strong></p> |
| `gkScript.waitForNetwork(0);`            | Query network status                                                                                                                                           |
| `gkScript::getNetworkId()`               | Returns the hex encoded Id of the network (the MAC-Address of the Gateway)                                                                                     |
| `gkScript::getNetworkId(true)`           | Append the IP address of the Gateway                                                                                                                           |

```shell
eval> gkScript.getNetworkId()
802AA8F1C3BD
eval> gkScript.getNetworkId(true)
802AA8F1C3BD@172.27.0.1
```

| Script                              | Function                                                                                                                                                                                           |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gkScript::getConnectionCost(host)` | Returns the cost flags for a connection to the specified host. See https://msdn.microsoft.com/de-de/library/windows/desktop/hh437608(v=vs.85).aspx for a definition of WCM\_CONNECTION\_COST flags |

## Logging

Logging amount is controlled by LogLevel values:

```c++
enum LogLevel
{
    ll_ERROR,       // Log errors
    ll_INFO,        // Log error + info
    ll_DEBUG,       // Log error + info + debug
    ll_OFF,         // Log nothing
};
```

Logging functions are part of the gkScript class:

```c++

var logLevel = gkScript.getLogLevel()   // gets current LogLevel
gkScript.setLogLevel(ll_ERROR)          // sets the LogLevel to Error

gkScript.logError(string)               // Log an error message
gkScript.logInfo(string)                // Log an info
gkScript.logDebug(string)               // Log a debug string

bool gkScript.exportLog(string path)    // Export the log to a file
```

## Tray Icon

gkScript offers the following functions to control the tray icon **(client.exe only)**:

```c++
class callbackObject
{
    void onTrayClick();
}

bool showTrayIcon(callbackObject);
hideTrayIcon();
```

## WMI

| Script                            | Function                                                                                                                                                                            |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gkScript::wmiQuery(Path, Query)` | <p>Executes WMI queries.<br><code>Path</code> is the object path of the WMI namespace to query, e.g. <code>ROOT\CIMV2</code><br><code>Query</code> is the WQL query to execute.</p> |

The following code is a WMI query sample:

```c++
def printResults(wmi) {
	var count = 0;
	for (i:wmi) {
		print ("    Item " + to_string(count));
		++count;
		for (j:i) {
			if (j.first[0] == '_') {continue;}
			if (is_var_undef(j.second)) {print("       " + j.first + ": undefined");}
			else{print("       " + j.first + ": " + to_string(j.second));}
		}
	}
}


print("Win32_DiskDrive");
var r=gkScript.wmiQuery("ROOT\\CIMV2", "SELECT Manufacturer, Model, Size FROM Win32_DiskDrive where Size>0");
printResults(r);
```

## Script Tool

The scriptTool class is a sandbox for ChaiScripts. You can call scripts without messing up your own environment...

| Script                       | Function                                  |
| ---------------------------- | ----------------------------------------- |
| `gkScript.ScriptTool(true)`  | gkScript object inside the sandbox        |
| `gkScript.ScriptTool(false)` | Creates a sandbox without gkScript object |

{

} ScriptTool instances do not have an `exit()` command {}

```c++
class ScriptTool
{
    bool addScript(string script);
    object callFunction(string func);
    void reset();  // delete all defined variables and functions
};
```

The following code is a ScriptTool sample:

```shell
eval> var script=gkScript.ScriptTool(true);
eval> script.callFunction("gkScript.getVersion()");
0.2.0.0
eval> script.addScript("def testFunction(val) {return \"testFunction - \" + val;}");
true
eval> script.callFunction("testFunction(\"hello world\")");
testFunction - hello world
eval> script.reset();
eval> script.callFunction("testFunction(\"hello world\")");
[2017-12-01 18:26:11.946] [1668] [error] [Default] ScriptTool::callFunction: Eval exception thrown: Error: "Can not find object: test" during evaluation at (__EVAL__ 1, 1)
eval>
```
