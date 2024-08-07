# Customize Reports

Reports can be customized (added, removed or changed) by uploading a custom `reports.js` to [Binaries menu](../beginning/binaries.md).

## Reference

The JavaScript file must consist of an object, where each key is a data source for the report. The value for each data source is an array consisting of reports. Each array item is an object that needs to have the following properties:

| Property    | Required | Description                                 |
| ----------- | -------- | ------------------------------------------- |
| `key`       | yes      | The unique key for the report               |
| `run`       | yes      | The run function to fetch the data          |
| `text`      | no       | The text display in the selector            |
| `extractor` | no       | The function to extract the returned data   |
| `headers`   | no       | An array of headers to display in the table |

The following sample shows the absolute minimal example for a report named **Report** in an own data source **datasource**. It just returns **Jon Doe** as data and displays it in the table.

```javascript
{
    datasource: [{
        key: 'report',
        text: 'Report',
        run: function(fetcher, groupId, extractor) {
            return new Promise(function(resolve, reject) {
                const result = extractor({ Name: 'Jon Doe' });
                resolve(result);
            });
        },
        extractor: function(json) {
            return [{ name: json.Name }];
        },
        headers: [{
			name: "Name",
			key: "name"
        }]
    }]
}
```

### Run

**run: function** is used to return the data and has the following signature: `function(fetcher, groupId, extractor)`.

**Parameters:**

* fetcher: The fetcher object. See below for more details.
* groupId: The GroupID on which to operate. An empty string if called for **all** clients.
* extractor: The extractor function that is defined for the report. See below for more details.

**Returns:** A promise that will eventually resolve to an array of objects.

#### Extractor

**extractor: function** is used to return the data and has the following signature: `function(json)`.

**Parameters**\
json: Some json data from which to extract the needed information.

**Returns**\
An array of objects.

#### Fetcher

All functions return a promise.

|                              |                                                              |
| ---------------------------- | ------------------------------------------------------------ |
| fetch(path)                  | Fetch data from path                                         |
| getModuleData (module, user) | Get module for "module" data for the specified user          |
| fetchGroups()                | Return all groups                                            |
| fetchGroup(group\_id)        | Return group with `group_id`. Use `*` for unassigned clients |
| fetchClients(group\_id)      | Fetch all clients which belong to group `group_id`.          |

#### Headers

Headers is an array of objects. Each object has the following members:

| Key          | Description                                          |
| ------------ | ---------------------------------------------------- |
| `name`       | String. Test to display in the column header         |
| `key`        | String. Key used to access the value to be displayed |
| `width`      | Number. Width of the column                          |
| `formatter`  | Function. Formatter to use                           |
| `sortable`   | Boolean. If this field is sortable                   |
| `resizable`  | Boolean. If this field is resizable                  |
| `filterable` | Boolean. If this field is filterable                 |

**Sample:**

```javascript
{
    name: "Group ID",
    key: "group_id",
    width: 250,
    formatter: GroupIDFormatter,
    sortable: true,
    resizable: true,
    filterable: true
}
```

#### Formatter

| Key                 | Description                                                                                       |
| ------------------- | ------------------------------------------------------------------------------------------------- |
| `ClientIDFormatter` | Formats a client ID to a link                                                                     |
| `GroupIDFormatter`  | Formats a group ID to a link                                                                      |
| `ProgressFormatter` | Creates a progress bar                                                                            |
| `SoftwareNameLink`  | Generates a link for a search on https://developer.microsoft.com/en-us/windows/ready-for-windows/ |
| `DateTimeFormatter` | Formats a unix time stamp to a localized string                                                   |
| `BooleanFormatter`  | Formats a boolean to either **yes** or **no**                                                     |

#### Utilities

**DataTime(unix\_timestamp)** = Creates a data time object from a unix time stamp.

### Samples

The following samples provide some common patterns that have been useful so far. Of course, these patterns can be combined with each other.

#### Remove Data Source

Included data source can be removed by overwriting the data source with a `null` value.

To remove PST data source, use `reports.js`:

```javascript
{
	pst: null
}
```

#### Remove Report

Included reports can be removed by providing a report with the same key but a `null` value for the run function.

To remove File state report, use `reports.js`:

```javascript
{
	file: [{
		key: 'file-state',
		run: null
	}]
}
```

## Included Reports

### Data Sources

* `progress`
* `inventory`
* `file`
* `inventory`

### Reports

| Data Source | Key                                 | Name                    |
| ----------- | ----------------------------------- | ----------------------- |
| `progress`  | `get-client-state`                  | Client State            |
| `progress`  | `get-client-module-state`           | Module State per Client |
| `progress`  | `get-group-assignments`             | Group Assignments       |
| `progress`  | `group-commands`                    | Group Commands          |
| `inventory` | `system-summary`                    | System details          |
| `inventory` | `software-installed-on-client`      | Software summary        |
| `inventory` | `software-installed-on-each-client` | Software details        |
| `inventory` | `drive-mappings-per-client`         | Drive mappings          |
| `inventory` | `printers`                          | Printers                |
| `file`      | `file-state`                        | File state              |
| `file`      | `file-state-anonymous`              | File state anonymized   |
| `pst`       | `pst-state`                         | PST state               |
| `pst`       | `pst-detail`                        | PST details             |

### Source

These reports are already included in RealMigrator. They are provided here as a reference.

```javascript
{
	progress: [{
		key: 'get-client-state',
		text: 'Client State',
		run: function(fetcher, groupId, extractor) {
			return fetcher.fetchClients(groupId).then((clients) => {
				var progress = [];
				for (var i in clients) {
					progress.push(clients[i]);
				}
				return progress;
			}).then((progress) => {
				var arr = [];
				for(var i in progress) {
					const x = extractor(progress[i]);
					arr = arr.concat(x);
				}
				return arr;
			});
		},
		extractor: function(json) {
			const progressData = (progress) => {
				var percent = 0.0;
				var errors = 0;

				var keys = progress ? Object.keys(progress) : [];
				if(keys.length > 0) {
					keys.forEach(key => {
						var element = progress[key];
						if((element.state !== 'skip') && element.total) {
							percent += element.count / element.total;
							if (element.errors) {errors += element.errors.length}
						} else {
							percent += 1.0;
						}
					});
					percent /= keys.length;
				}
			
				return [percent, errors];
			};

			var p = json["Status"] || { };
			var state = '';

			if (p.state === 0) {
				state = 'in progress';
			} else if(p.state === 1) {
				state = 'finished';
			} else if(p.state === 2) {
				state = 'error';
			}
			var percent = 0.0, errors = 0;
			if (p.progress) {
				[percent, errors] = progressData(p.progress)
			}
			return [{
				group_id: json.Group ? json.Group.ObjectID : '',
				group_name: json.Group ? json.Group.DisplayName : '',
				client_id: json["ObjectID"],
				display_name: json["DisplayName"],
				state: state,
				message: p ? p.message : "",
				percent: percent,
				errors: errors,
				timestamp: json.Timestamp,
				upn: json.UPN,
		}];
		},
		headers: [{
			name: "Group ID",
			key: "group_id",
			width: 250,
			formatter: GroupIDFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Group",
			key: "group_name",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Client ID",
			key: "client_id",
			formatter: ClientIDFormatter,
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "UPN",
			key: "upn",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "State",
			key: "state",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Message",
			key: "message",
			width: 400,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Complete",
			key: "percent",
			formatter: ProgressFormatter,
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Errors",
			key: "errors",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Last seen",
			key: "timestamp",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}]
	},{
		key: 'get-group-assignments',
		text: 'Group Assignments',
		run: function(fetcher, groupId, extractor) {
			return fetcher.fetchClients(groupId).then((clients) => {
				return extractor(clients);
			});
		},
		extractor: function(clients) {
			var map = {};

			for(var i in clients) {
				const client = clients[i];
				const memberId = client.GroupID || client.ObjectID;

				if (!map[memberId]) {
					const groupId = client.Group ? client.Group.ObjectID : '';
					const groupName = client.Group ? client.Group.DisplayName : '';

					map[memberId] = {
						GroupID: groupId,
						GroupName: groupName,
						MemberID: memberId,
						count: 1
					};
				} else {
					map[memberId].count += 1;
				}
			}

			var arr = []
			for(var key in map) {
				arr.push(map[key]);
			}
			return arr;
		},
		headers: [{
			name: "Group ID",
			key: "GroupID",
			formatter: GroupIDFormatter,
			width: 400,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Group Name",
			key: "GroupName",
			width: 400,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Member ID",
			key: "MemberID",
			width: 400,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Count",
			key: "count",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}]
	}, {
		key: 'group-commands',
		text: 'Group Commands',
		run: function(fetcher, groupId, extractor) {
			return fetcher.fetchModuleConfiguration(groupId).then((group_configs) => {
				return extractor(group_configs);
			});
		},
		extractor: function(group_configs) {
			var arr = []

			for(var i in group_configs) {
				arr.push({
					ObjectID: group_configs[i].ObjectID,
					DisplayName: group_configs[i].DisplayName,
					Inventory: group_configs[i].Config.inventory,
					File: group_configs[i].Config.file,
					PST: group_configs[i].Config.pst,
					Custom: group_configs[i].Config.custom
				});
			}

			return arr;
		},
		headers: [{
			name: "Group ID",
			key: "ObjectID",
			formatter: GroupIDFormatter,
			width: 400,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Group Name",
			key: "DisplayName",
			width: 400,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Inventory",
			key: "Inventory",
			width: 100,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "File",
			key: "File",
			width: 100,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "PST",
			key: "PST",
			width: 100,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Custom",
			key: "Custom",
			width: 100,
			sortable: true,
			resizable: true,
			filterable: true
		}]
    }],
	inventory: [{
		key: 'system-summary',
		text: 'System details',
		run: function(fetcher, groupId, extractor, progressCB) {
			return fetcher.getModuleDataForGroup(groupId, 'inventory', progressCB).then((inventories) => {
				var arr = [];
				for(var i in inventories) {
					const x = extractor(inventories[i], inventories[i].ObjectID, inventories[i].DisplayName);
					arr = arr.concat(x);
				}
				return arr;
			});
		},
		extractor: function(json, client_id, display_name) {
			var arr = [];
			var system_ram = "";

			var disk_model = "", disk_size = "";
			var os_type = "", os_architecture = "";
			var pc_model = "", pc_manufacturer = "", pc_serial = "";
			var processor_name = "", processor_speed = 0, processor_id = "";
			var video_processor = "", video_ram = 0;
			var tpm_name = "", tpm_status = "";

			if (json.wmiData) {
				if (json.wmiData.ComputerSystem && json.wmiData.ComputerSystem.length) {
					system_ram = json.wmiData.ComputerSystem[0].TotalPhysicalMemory;
				}
				if (json.wmiData.ComputerSystemProduct && json.wmiData.ComputerSystemProduct.length) {
					pc_model = json.wmiData.ComputerSystemProduct[0].Version.length > json.wmiData.ComputerSystemProduct[0].Name.length ?
						json.wmiData.ComputerSystemProduct[0].Version : json.wmiData.ComputerSystemProduct[0].Name;
				}
				if (json.wmiData.DiskDrive && json.wmiData.DiskDrive.length) {
					disk_model = json.wmiData.DiskDrive[0].Model;
					disk_size = json.wmiData.DiskDrive[0].Size;
				}
				if (json.wmiData.OperatingSystem && json.wmiData.OperatingSystem.length) {
					os_type = json.wmiData.OperatingSystem[0].Caption;
					os_architecture = json.wmiData.OperatingSystem[0].OSArchitecture;
				}
				if (json.wmiData.SystemEnclosure && json.wmiData.SystemEnclosure.length) {
					pc_manufacturer = json.wmiData.SystemEnclosure[0].Manufacturer;
					pc_serial = json.wmiData.SystemEnclosure[0].SerialNumber;
				}
				if (json.wmiData.Processor && json.wmiData.Processor.length) {
					processor_name = json.wmiData.Processor[0].Name;
					processor_speed = json.wmiData.Processor[0].CurrentClockSpeed;
					processor_id = json.wmiData.Processor[0].ProcessorId;
				}
				if (json.wmiData.VideoController && json.wmiData.VideoController.length) {
					video_ram = json.wmiData.VideoController[0].AdapterRAM;
					video_processor = json.wmiData.VideoController[0].VideoProcessor;
				}
				if (json.wmiData.TPM && json.wmiData.TPM.length) {
					tpm_name = json.wmiData.TPM[0].Name;
					tpm_status = json.wmiData.TPM[0].Status;
				}
			}

			arr.push({
				client_id: client_id,
				display_name: display_name,
				system_ram: system_ram,
				disk_model: disk_model,
				disk_size: disk_size,
				os_type: os_type,
				os_architecture: os_architecture,
				pc_model: pc_model,
				pc_manufacturer: pc_manufacturer,
				pc_serial: pc_serial,
				processor_name: processor_name,
				processor_speed: processor_speed,
				processor_id: processor_id,
				video_processor: video_processor,
				video_ram: video_ram,
				tpm_name: tpm_name,
				tpm_status: tpm_status
			})
			return arr;
		},
		headers: [{
			name: "Client ID",
			key: "client_id",
			width: 250,
			formatter: ClientIDFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Display Name",
			key: "display_name",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "RAM size",
			key: "system_ram",
			width: 140,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Disk",
			key: "disk_model",
			width: 200,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Disk size",
			key: "disk_size",
			width: 140,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Operating System",
			key: "os_type",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Architecture",
			key: "os_architecture",
			width: 140,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "PC manufacturer",
			key: "pc_manufacturer",
			width: 200,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "PC model",
			key: "pc_model",
			width: 200,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Serial #",
			key: "pc_serial",
			width: 140,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Processor",
			key: "processor_name",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Speed",
			key: "processor_speed",
			width: 140,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Processor id",
			key: "processor_id",
			width: 200,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Video Processor",
			key: "video_processor",
			width: 200,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Video RAM",
			key: "video_ram",
			width: 140,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "TPM type",
			key: "tpm_name",
			width: 200,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "TPM status",
			key: "tpm_status",
			width: 140,
			sortable: true,
			resizable: true,
			filterable: true
		}]
	}, {
		key: 'software-installed-on-client',
		text: 'Software summary',
		run: function(fetcher, groupId, extractor, progressCB) {
			return fetcher.getModuleDataForGroup(groupId, 'inventory', progressCB).then((inventories) => {
				return extractor(inventories);
			});
		},
		extractor: function(inventories) {
			var map = {};

			for(var i in inventories) {
				const json = inventories[i];
				if(json) {
					for (var x in json.installerRegistry) {
						for (var id in json.installerRegistry[x]) {
							if(!map[id]) {
								map[id] = {
									packageCode: json.installerRegistry[x][id].packageCode,
									name: json.installerRegistry[x][id].name,
									version: json.installerRegistry[x][id].version,
									count: 0
								};
							}
		
							map[id].count += 1;
						}
					}
				}
			}

			var arr = []
			for(var key in map) {
				arr.push(map[key]);
			}
			return arr;
		},
		headers: [{
			name: "Name",
			key: "name",
			width: 400,
			formatter: SoftwareNameLink,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Version",
			key: "version",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Package Code",
			key: "packageCode",
			width: 300,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Install Count",
			key: "count",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}]
	}, {
		key: 'software-installed-on-each-client',
		text: 'Software details',
		run: function(fetcher, groupId, extractor, progressCB) {
			return fetcher.getModuleDataForGroup(groupId, 'inventory', progressCB).then((inventories) => {
				var arr = [];
				for(var i in inventories) {
					const x = extractor(inventories[i], inventories[i].ObjectID, inventories[i].DisplayName);
					arr = arr.concat(x);
				}
				return arr;
			});
		},
		extractor: function(json, client_id, display_name) {
			var arr = [];
			for (var x in json.installerRegistry) {
				for (var y in json.installerRegistry[x]) {
					json.installerRegistry[x][y].group_id = json.GroupID;
					json.installerRegistry[x][y].group_name = json.GroupName;
					json.installerRegistry[x][y].client_id = client_id;
					json.installerRegistry[x][y].display_name = display_name;
					arr.push(json.installerRegistry[x][y])
				}
			};
			return arr;
		},
		headers: [{
			name: "Group ID",
			key: "group_id",
			width: 250,
			formatter: GroupIDFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Group",
			key: "group_name",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Client ID",
			key: "client_id",
			width: 250,
			formatter: ClientIDFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Name",
			key: "name",
			width: 400,
			formatter: SoftwareNameLink,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Package Code",
			key: "packageCode",
			width: 300,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Version",
			key: "version",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Flags",
			key: "flags",
			width: 100,
			sortable: true,
			resizable: true,
			filterable: true
		}]
	}, {
		key: 'drive-mappings-per-client',
		text: 'Drive mappings',
		run: function(fetcher, groupId, extractor, progressCB) {
			return fetcher.getModuleDataForGroup(groupId, 'inventory', progressCB).then((inventories) => {
				var arr = [];
				for(var i in inventories) {
					const x = extractor(inventories[i], inventories[i].ObjectID, inventories[i].DisplayName);
					arr = arr.concat(x);
				}
				return arr;
			});
		},
		extractor: function(json, client_id, display_name) {
			var arr = [];
			if (json.wmiData && json.wmiData.MappdedDrives) {
				for (var x in json.wmiData.MappdedDrives) {
					arr.push({
						group_id: json.GroupID,
						group_name: json.GroupName,
						client_id: client_id,
						name: display_name,
						drive: json.wmiData.MappdedDrives[x].Caption,
						path: json.wmiData.MappdedDrives[x].ProviderName
					})
				}
			}
			return arr;
		},
		headers: [{
			name: "Group ID",
			key: "group_id",
			width: 250,
			formatter: GroupIDFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Group",
			key: "group_name",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Client ID",
			key: "client_id",
			width: 250,
			formatter: ClientIDFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Name",
			key: "name",
			width: 400,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Drive",
			key: "drive",
			width: 100,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Path",
			key: "path",
			width: 400,
			sortable: true,
			resizable: true,
			filterable: true
		}]
	}, {
		key: 'printers',
		text: 'Printers',
		run: function(fetcher, groupId, extractor, progressCB) {
			return fetcher.getModuleDataForGroup(groupId, 'inventory', progressCB).then((inventories) => {
				var arr = [];
				for(var i in inventories) {
					const x = extractor(inventories[i], inventories[i].ObjectID, inventories[i].DisplayName);
					arr = arr.concat(x);
				}
				return arr;
			});
		},
		extractor: function(json, client_id, display_name) {
			var arr = [];
			if (json.wmiData && json.wmiData.Printer && json.wmiData.Printer.length) {
				for (var x in json.wmiData.Printer) {
					json.wmiData.Printer[x].group_id = json.GroupID;
					json.wmiData.Printer[x].group_name = json.GroupName;
					json.wmiData.Printer[x].client_id = client_id;
					json.wmiData.Printer[x].display_name = display_name;
					arr.push(json.wmiData.Printer[x]);
				}
			}
			return arr;
		},
		headers: [{
			name: "Group ID",
			key: "group_id",
			width: 250,
			formatter: GroupIDFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Group",
			key: "group_name",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Client ID",
			key: "client_id",
			width: 250,
			formatter: ClientIDFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Name",
			key: "Name",
			width: 400,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Printer Name",
			key: "Caption",
			width: 300,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Port",
			key: "PortName",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Local",
			key: "Local",
			width: 100,
			formatter: BooleanFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}]
	}],
	file: [{
		key: 'file-state',
		text: 'File state',
		run: function(fetcher, groupId, extractor, progressCB) {
			return fetcher.getModuleDataForGroup(groupId, 'file', progressCB).then((inventories) => {
				var arr = [];
				for(var i in inventories) {
					const x = extractor(inventories[i], inventories[i].ObjectID, inventories[i].DisplayName);
					arr = arr.concat(x);
				}
				return arr;
			});
		},
		extractor: function(json, client_id, display_name) {
			var arr = [];
			const states = ["Check", "Incomplete", "Exclude", "Delete", "In Sync"];
			var walk = function (dir) {
				for (var f in dir.files) {
					dir.files[f].catalog = cat;
					dir.files[f].name = f;
					dir.files[f].path = dir.path;
					dir.files[f].client_id = client_id;
					dir.files[f].display_name = display_name;
					dir.files[f].state_cooked = states[dir.files[f].state];
					dir.files[f].modified = DateTime(dir.files[f].modified);
					if (dir.files[f].state === 4) {
						dir.files[f].size_synced = dir.files[f].size;
					} else if ((dir.files[f].state === 1) && dir.files[f].hashMap) {
						dir.files[f].size_synced = dir.files[f].hashMap.size;						
					} else {
						dir.files[f].size_synced = 0;						
					}
					delete dir.files[f].hashMap;
					arr.push(dir.files[f])
				}
				for (var d in dir.dirs) {
					walk(dir.dirs[d]);
				}
			};

			for (var cat in json.catalogs) {
				walk(json.catalogs[cat]);
			};
			return arr;
		},
		headers: [{
			name: "Client ID",
			key: "client_id",
			width: 250,
			formatter: ClientIDFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "File",
			key: "name",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Path",
			key: "path",
			width: 400,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Size",
			key: "size",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Size Synced",
			key: "size_synced",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "State",
			key: "state_cooked",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Modified",
			key: "modified",
			width: 150,
			formatter: DateTimeFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}]
	},
	{
		key: 'file-state-anonymous',
		text: 'File state anonymized',
		run: function(fetcher, groupId, extractor, progressCB) {
			return fetcher.getModuleDataForGroup(groupId, 'file', progressCB).then((inventories) => {
				var arr = [];
				for(var i in inventories) {
					const x = extractor(inventories[i], inventories[i].ObjectID, inventories[i].DisplayName);
					arr = arr.concat(x);
				}
				return arr;
			});
		},
		extractor: function(json, client_id, display_name) {
			var arr = [];
			const states = ["Check", "Incomplete", "Exclude", "Delete", "In Sync"];
			var d = new Date();
			var t = Math.round(d.getTime()/1000);

			function rounder(num, div) {
				return Math.round(num/div) * div;
			}
			function namer(name) {
				var dot = name.lastIndexOf(".");
				if (dot > 0) { return name.substring(dot+1);}
				else return "";
			}
			function sizer(size) {
				return (rounder(size, 1000000000) ||
					rounder(size, 1000000) ||
					rounder(size, 1000) ||
					rounder(size, 10));
			}
			function dater(date) {
				return Math.floor((t-date)/(60*60*24*30));
			}
			function walk(dir) {
				for (var f in dir.files) {
					arr.push({name: namer(f), size: sizer(dir.files[f].size), 
						months_unchanged: dater(dir.files[f].modified), 
						state: states[dir.files[f].state]});
				}
				for (var d in dir.dirs) {
					walk(dir.dirs[d]);
				}
			}

			for (var cat in json.catalogs) {
				walk(json.catalogs[cat]);
			};
			return arr;
		},
		headers: [{
			name: "File",
			key: "name",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Size",
			key: "size",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Months unchanged",
			key: "months_unchanged",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "State",
			key: "state",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}]
	}],
	pst: [{
		key: 'pst-state',
		text: 'PST state',
		run: function(fetcher, groupId, extractor, progressCB) {
			return fetcher.getModuleDataForGroup(groupId, 'pst', progressCB).then((inventories) => {
				var arr = [];
				for(var i in inventories) {
					const x = extractor(inventories[i], inventories[i].ObjectID, inventories[i].DisplayName);
					arr = arr.concat(x);
				}
				return arr;
			});
		},
		extractor: function(json, client_id, display_name) {
			var arr = [];
			const states = ["Check", "Incomplete", "Exclude", "Delete", "In Sync"];
			for (var cat in json.catalogs) {
				for (var f in json.catalogs[cat].files) {
					json.catalogs[cat].files[f].catalog = cat;
					json.catalogs[cat].files[f].path = f;
					json.catalogs[cat].files[f].client_id = client_id;
					json.catalogs[cat].files[f].display_name = display_name;
					json.catalogs[cat].files[f].state_cooked = states[json.catalogs[cat].files[f].state];
					json.catalogs[cat].files[f].modified = DateTime(json.catalogs[cat].files[f].modified);
					if (json.catalogs[cat].files[f].state === 4) {
						json.catalogs[cat].files[f].size_synced = json.catalogs[cat].files[f].size;
					} else if ((json.catalogs[cat].files[f].state === 1) && json.catalogs[cat].files[f].hashMap) {
						json.catalogs[cat].files[f].size_synced = json.catalogs[cat].files[f].hashMap.size;						
					} else {
						json.catalogs[cat].files[f].size_synced = 0;						
					}
					delete json.catalogs[cat].files[f].hashMap;
					delete json.catalogs[cat].files[f].summary;
					arr.push(json.catalogs[cat].files[f])
				}
			};
			return arr;
		},
		headers: [{
			name: "Client ID",
			key: "client_id",
			width: 250,
			formatter: ClientIDFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "PST",
			key: "name",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Path",
			key: "path",
			width: 400,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Size",
			key: "size",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Size Synced",
			key: "size_synced",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "State",
			key: "state_cooked",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Modified",
			key: "modified",
			width: 150,
			formatter: DateTimeFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}]
	},{
		key: 'pst-detail',
		text: 'PST details',
		run: function(fetcher, groupId, extractor, progressCB) {
			return fetcher.getModuleDataForGroup(groupId, 'pst', progressCB).then((inventories) => {
				var arr = [];
				for(var i in inventories) {
					const x = extractor(inventories[i], inventories[i].ObjectID, inventories[i].DisplayName);
					arr = arr.concat(x);
				}
				return arr;
			});
		},
		extractor: function(json, client_id, display_name) {
			var arr = [];
			const states = ["Check", "Incomplete", "Exclude", "Delete", "In Sync"];
			for (var cat in json.catalogs) {
				for (var f in json.catalogs[cat].files) {
					json.catalogs[cat].files[f].catalog = cat;
					json.catalogs[cat].files[f].path = f;
					json.catalogs[cat].files[f].client_id = client_id;
					json.catalogs[cat].files[f].display_name = display_name;
					json.catalogs[cat].files[f].state_cooked = states[json.catalogs[cat].files[f].state];
					json.catalogs[cat].files[f].modified = DateTime(json.catalogs[cat].files[f].modified);
					if (json.catalogs[cat].files[f].state === 4) {
						json.catalogs[cat].files[f].size_synced = json.catalogs[cat].files[f].size;
					} else if ((json.catalogs[cat].files[f].state === 1) && json.catalogs[cat].files[f].hashMap) {
						json.catalogs[cat].files[f].size_synced = json.catalogs[cat].files[f].hashMap.size;	
					} else {
						json.catalogs[cat].files[f].size_synced = 0;						
					}
                  	if (json.catalogs[cat].files[f].hashMap) {
						json.catalogs[cat].files[f].blockmap_length = json.catalogs[cat].files[f].hashMap.blockmap ? json.catalogs[cat].files[f].hashMap.blockmap.length : 0;
						json.catalogs[cat].files[f].hash = json.catalogs[cat].files[f].hashMap.hash;
						json.catalogs[cat].files[f].hashmap_index = json.catalogs[cat].files[f].hashMap.nextIndex;
                    } else {
						json.catalogs[cat].files[f].blockmap_length = 0;
						json.catalogs[cat].files[f].hash = '';
						json.catalogs[cat].files[f].hashmap_index = 0;
					}
					if (json.catalogs[cat].files[f].summary) {
						json.catalogs[cat].files[f].encryption = json.catalogs[cat].files[f].summary.Encryption;
						json.catalogs[cat].files[f].format = json.catalogs[cat].files[f].summary.Format;
						json.catalogs[cat].files[f].summary_size = json.catalogs[cat].files[f].summary.Size;
						json.catalogs[cat].files[f].version = json.catalogs[cat].files[f].summary.Version;
					}
					delete json.catalogs[cat].files[f].hashMap;
					delete json.catalogs[cat].files[f].summary;
					arr.push(json.catalogs[cat].files[f])
				}
			};
			return arr;
		},
		headers: [{
			name: "Client ID",
			key: "client_id",
			width: 250,
			formatter: ClientIDFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "PST",
			key: "name",
			width: 250,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Path",
			key: "path",
			width: 400,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Size",
			key: "size",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Header Size",
			key: "summary_size",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Format",
			key: "format",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Encryption",
			key: "encryption",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Version",
			key: "version",
			width: 80,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Size Synced",
			key: "size_synced",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Blockmap length",
			key: "blockmap_length",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Next Index",
			key: "hashmap_index",
			width: 80,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Hash",
			key: "hash",
			width: 270,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "State",
			key: "state_cooked",
			width: 150,
			sortable: true,
			resizable: true,
			filterable: true
		}, {
			name: "Modified",
			key: "modified",
			width: 150,
			formatter: DateTimeFormatter,
			sortable: true,
			resizable: true,
			filterable: true
		}]
	}]
}
```
