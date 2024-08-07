# rmService.exe

* It is a background service
* No automatic installation

### Service Installation

To install **rmService.exe** store it on the computer. Run the .exe file with a command-line interpreter.\
Enter the following command:

```
install <Project> [<UPN>]
```

A **UPN** (User Principal Name) is optional, but a **project name** is essential for the installation command.

![Command line options for rmService.exe](<../.gitbook/assets/Screenshot 2024-01-15 131700.png>)

### Upload Agent

To install the service as an upload agent, you need to provide a **project name** and a **token**. You can generate a token on the server configuration page. Both parameters are required.

```
install-uploadagent <Project> <Token>
```
