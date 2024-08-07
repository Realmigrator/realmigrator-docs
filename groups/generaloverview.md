# Groups General Overview

![](../.gitbook/assets/groups1.PNG)

This menu shows:

* A summary of all registered groups - **Groups (Total xx)**
* Add new groups function
* A filter function - **Filter by Client** (name search)

## Groups Settings

With a click on a group, the following menu appears:

![](../.gitbook/assets/groups2.PNG)

This menu contains information about a group client:

* **Display Name** (Group Name)
* **Assigned Objects**
* Date selection
* Available modules
* Task of selected module (**Skip**, **Discover** and **Migrate** - explanation see table below)

| Command      | Description                                                                         | file module | PST module | inventory module |
| ------------ | ----------------------------------------------------------------------------------- | :---------: | :--------: | :--------------: |
| **skip**     | module is not executed                                                              |      +      |      +     |         +        |
| **discover** | module discovers items the client and sends a report, but does not migrate any data |      +      |      +     |         +        |
| **migrate**  | migrates data continuously                                                          |      +      |      +     |         -        |
