# Set bandwidth

Use **mainConfigCheck()** to change bandwidth settings.

To change some values use the following code:

```c++
def mainConfigCheck(globalSettings, config) {

    if (this or that) {
        config["bandwidthSettings"]["maxBandWidth"] = 200000;
        config["bandwidthSettings"]["minSleepTime"] = 10;
    }
}
```

To overwrite the complete bandwidthSettings object use the following code:

```c++
def mainConfigCheck(globalSettings, config) {

    if (this or that) {
        config["bandwidthSettings"] = [
            "maxBandWidth": 10000,
            "slowNetworkBarrier": 300,
            "slowNetworkUsage": 0.25,
            "unusableNetworkBarrier": 200,
            "unusableWaitTime": 120,
            "minSleepTime": 500,
            "useMeteredConnection":true
        ];
    }
}
```
