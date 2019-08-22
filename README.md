:computer: System API
=====================

System API (sys-API) is a RESTful API to your computers hardware.

Sys-API publishes and monitors values from [OSHI](https://github.com/oshi/oshi) with the help of [Dropwizard](https://github.com/dropwizard/dropwizard). On Windows the information is supplemented with
[OpenHardwareMonitor](https://github.com/openhardwaremonitor/openhardwaremonitor) with a little help from [OhmJni4Net](https://github.com/Krillsson/ohmjni4net).

## What can it do?

- Threshold-based monitoring of system values
- List ongoing threshold violations (events)
- CPU usage & info
- Memory usage
- List running processes
- List all network interfaces
- Show info from sensors and fans
- Motherboard information

## Endpoints

```

    // Static information about the system
    GET     /system
    
    // Current load of the system
    GET     /system/load
    
    // An array of (date, system load) entries
    GET     /system/load/history

    // Currently configured monitors
    GET     /monitors
    
    // Configure a new monitor
    POST    /monitors
    
    // Remove an existing monitor
    DELETE  /monitors/{id}

    // Get currently ongoing threshold violations, if any.
    GET     /events

    // displays meta info about the Sys-API server
    GET     /

    GET     /cpu
    GET     /cpu/load
    GET     /cpu/load/history
    GET     /drives
    GET     /drives/loads
    GET     /drives/loads/history
    GET     /drives/loads/{name}
    GET     /drives/{name}
    GET     /gpus
    GET     /gpus/loads
    GET     /gpus/loads/history
    GET     /memory
    GET     /memory/history
    GET     /motherboard
    GET     /motherboard/health
    GET     /nics
    GET     /nics/loads
    GET     /nics/loads/history
    GET     /nics/loads/{id}
    GET     /nics/{id}
    GET     /pid
    GET     /processes
    GET     /processes/{pid}
    GET     /system/jvm
    GET     /system/uptime
    GET     /version
```

## Usage

The `/system` endpoint contains accumulated information about the whole system. Then if you want information for just a specific part of the system you can use the part specific endpoint.
So for example the cpu temperature and usage is under `/cpu`. Some things are excluded from `/system`, such as `/system/jvm` and `/processes` since they contain so much information.

### Building UIs

The general idea is that each item has a root endpoint that contain static information, like a processors name or the names of the disks. Then each and every item has a `/load` endpoint that gives you the current
load of that item and a `/load/history` that gives you the load history over time. Disks or network interfaces always have mappable names or ID's between the root and load/history content.

History endpoints also have the ability to limit it's output. 

Example: `v2/system/load/history?fromDate=2018-09-23T15:11:55.661&toDate=2018-09-23T15:21:25.659`

So if you are building a UI on top of this API you can query the root endpoint once or seldom. And then if you want to display load over time you can query the history endpoint once and
then continuously poll the load endpoint and append the results to the history collections.

### Monitoring

Currently, you can monitor

 - CPU load
 - CPU temperature
 - Memory usage
 - Network up or down
 
#### Configuring monitors
 
You add a monitor by doing a POST call to `/monitors` with the body of `com.krillsson.sysapi.dto.monitor.Monitor`:

```json
{
  "inertiaInSeconds": "10",
  "type": "CPU|CPU_TEMP|DRIVE|GPU|MEMORY|NETWORK_UP",
  "threshold": "123"
}
```

If you exclude the `"id"` field from the JSON the API will create a new monitor. If you include an ID the API will update the monitor of that ID if it exists or return NOT FOUND/404 if it does not exist. 

The response for this call will be the UUID of the monitor that was created:

```json
{
    "id": "4bd27b5a-b3a2-4716-a7bb-8f0bf9815849"
}
```

You can then use this ID to delete or update a monitor.

_Inertia_: the amount of time in seconds the threshold should be crossed for before triggering a event. The same goes for when
transitioning back to below the threshold.

_Note_: The `NETWORK_UP` ignores the threshold and uses 1 as value for up and 0 for down.

_Note_: The cpu, gpu and cpu temp monitors checks if the load goes _above_ the thresholds while the drive and memory onces
monitors considers a violation to be when the value drops below a threshold.

#### Querying for events

Call the `/events` endpoints to get the currently ongoing threshold violations (events).

The response will be an array of `com.krillsson.sysapi.dto.monitor.MonitorEvent`:

```json
[
    {
        "id": "9723c720-4fa0-4497-a08d-8117aa343713",
        "monitorId": "4bd27b5a-b3a2-4716-a7bb-8f0bf9815849",
        "time": 1532602649608,
        "type": "CPU|CPU_TEMP|DRIVE|GPU|MEMORY|NETWORK_UP",
        "monitorStatus": "START|STOP",
        "threshold": 123,
        "value": 122
    }
]
```

The `"id"` in the response above can be used to delete the events associated with that ID. If you remove a _START_ event
with a particular ID, the _STOP_ event for that event will be discarded. 

## Running
Download the [release](https://github.com/Droid-MAX/system-api/releases).

- Debian/Ubuntu: download the package and use dpkg to install it.

- Service Manage: 

Start: `sudo service system-api start`

Stop: `sudo service system-api stop`

Status: `sudo service system-api status`

Reload: `sudo service system-api reload`

Restart: `sudo service system-api restart`

and service default boot.

- Log File path: `/var/log/syslog`

## Configuration
The configuration.yml file is a [Dropwizard configuration file](https://dropwizard.github.io/dropwizard/manual/configuration.html).

configuration.yml path is `/etc/configuration.yml`

Basic Authentication:

    user:
        username: [change me]
        password: [change me]
        
Disable the OhmJni extension on Windows:

    windows:
      enableOhmJniWrapper: false

### Enable HTTPS

Forward HTTP to HTTPS:

    forwardHttps: true

License
-------

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
