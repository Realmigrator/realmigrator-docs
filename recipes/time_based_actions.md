# Time based actions

Copy the function below to the Global Configuration script. Call it in **getGroupId()**, **mainConfigCheck()** or any module config check function.

```c++
def isNight() {
    var d=60*60*24;
    var t = gkScript.time();
    t += gkScript.utcOffset();
    var localTime = t-t/d*d;

    // night is before 28.800 = 8 * 60 *60 = 8:00 local time
    // and after = 75.600 = 21 * 60 * 60 = 21:00 local time
    var night = ((localTime < 28800) || (localTime > 75600)) 
    return night;
}
```

The following script is an example to detect if it is a weekday on the client.

```c++
def isWeekend() {
	var isWeekend = false;
  	try
    {
  		var query = gkScript.wmiQuery("ROOT\\CIMV2", "SELECT * FROM win32_localtime");
  		var day = query[0]["DayOfWeek"];
  		gkScript.logDebug("Current day of week: " + to_string(day));
      	// day 0 = sunday; day 6 = saturday
  		if (day == 0 || day == 6)
  		{
    		isWeekend = true;
  		}
    } catch (e)
    {
    	gkScript.logError("Failed to retrieve day of week: " + e.what());
    }
  	return isWeekend;  
}
```
