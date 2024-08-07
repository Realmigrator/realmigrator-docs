# Group based actions

It is possible to determine a user´s group membership in **mainConfigCheck()** or any module config check function. To do so copy the following link:

```c++
def mainConfigCheck(globalSettings, config) {
    var group = globalSettings["Group"];
	if (type_name(group)=="Map") {
	    gkScript.logInfo("Group: " + group["DisplayName"] + "(" + group["ObjectID"] + ")");

        if ("Offenbach" == group["DisplayName"]) {
            // user is in group named "Offenbach"
		}
        else if ("b865c07b49ab454bbd1f41c7bb466d27" == group["ObjectID"]) {
            // user is in group with id b865c07b49ab454bbd1f41c7bb466d27
        }
    } else {
        // User is unassigned
    }
}
```
