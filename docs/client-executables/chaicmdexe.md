# chaiCmd.exe

* Command .exe file
* Suitable for testing configurations and settings
* Suitable for reporting

### A exemplary chaiCmd.exe usage

Create reporting with **server\_query.chai** (**carbon** => subfolder **testscripts**)

**chaiCmd.exe ..\testscripts\server\_query.chai**

Usage: server\_query.chai \[PROJECT] \[TOKENFILE] \[COMMAND] \[ARGS...]

Command can be one of the following:

| Command    | Result                                                              |
| ---------- | ------------------------------------------------------------------- |
| groups     | list groups                                                         |
| software   | list installed software (Additional args: \[GROUPID-optional])      |
| progress   | list client progress (Additional args: \[GROUPID-optional])         |
| errors     | list client errors (Additional args: \[GROUPID-optional])           |
| files      | get files report (Additional args: \[CLIENTID])                     |
| clientjson | clientjson download client json data (Additional args: \[CLIENTID]) |
| log        | download log file (Additional args: \[CLIENTID])                    |
| module     | download module data (Additional args: \[CLIENTID] \[MODULE])       |

### Report generation

> **chaiCmd.exe ..\testscripts\server\_query.chai \[projectname] token.txt>test.csv**

It creates a progress reporting. Optional use a GroupID to specify a report.

> **chaiCmd.exe ..\gkScriptModule\testscripts\server\_query.chai \[projectname] token.txt groups**

It creates a GroupID - name list
