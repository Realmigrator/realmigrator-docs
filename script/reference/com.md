# COM

## Object creation

| Script                                                          | Function                                                                                                                                                                          |
| --------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ComObject gkScript.createComObject(string id, bool inProcess)` | <p>Will create a COM object.<br>- id can be a CLSID or a ProgID<br>- Set inProcess to <strong>true</strong> for in-process objects, <strong>false</strong> for local servers.</p> |
| `ComObject gkScript.getComObject(string id)`                    | <p>Will attach to a running COM server.<br>- id can be a CLSID, a ProgID or a Moniker</p>                                                                                         |

## ComObject class

```c++
enum InvokeTypes {
    DISPATCH_METHOD,
    DISPATCH_PROPERTYGET,
    DISPATCH_PROPERTYPUT,
    DISPATCH_PROPERTYPUTREF
};


class ComObject
{
    bool connected();          // return true if ComObject is connected to COM server
    void disconnect();

    value invoke(string name, vector params);              // invoke a function
    value invoke(string name, InvokeTypes type, vector params);              // invoke a function with specified invoke type
    value getProp(string prop);                            // read property
    bool setProp(string prop, val);                        // write property
}
```

Sample:

```shell
eval> var inet = gkScript.createComObject("InternetExplorer.Application", false);
eval> inet.connected();
true
eval> inet.setProp("Visible", true);
true
eval> inet.getProp("HWND");
3736394
eval> inet.invoke("Navigate", ["http://www.google.de"])
eval> inet.invoke("Close", [])
[2018-02-23 11:29:43.892] [9516] [error] [Default] gkScript::getProp can't get id for Close
eval> inet.invoke("Quit", []);
eval> inet.connected()
true
eval> inet.disconnect()
eval> inet.connected()
false
eval>
```
