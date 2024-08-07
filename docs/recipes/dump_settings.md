# Dump Config

To log configuration data, to dump global settings or dump module configurations in order to know what to evaluate and what to change, it is possible to dump settings in **mainConfigCheck()** or any module configuration check function.\
To do so copy the following code:

```c++
def mainConfigCheck(globalSettings, config) {

    gkScript.logInfo("globalSettings: \n" + to_json(globalSettings));
    gkScript.logInfo("config: \n" + to_json(config));

}
```
