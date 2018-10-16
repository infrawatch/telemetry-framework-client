# Variables

There are several variables which control what collectd + AMQP functionality is installed

## Glossary

 - QDR -- Qpid Dispatch Router
 - SA -- Service Assurance

## QDR Configuration
### router
The router variable allows the setting of general Qpid router parameters 

#### id_prefix
A prefix that is added to the beginining of the router name

#### mode
The mode that the QDR is to run in [*interior|edge*].  This should be *interior*.

    router:
        id_prefix: COLLECTD_ROUTER
        mode: interior

### listeners
#### name
#### host
#### port

    listeners:
        - name: cloud-node
          host: 0.0.0.0
          port: amqp

### connectors
Specify remote connections.  In the telemetry-framework context, the remote connector should specify the address of the primary QDR in the OpenShift cluster.  This address is exposed using the *Route* construct.  The default SA install creates a route with the name *blue.qdr-sa-telemetry.apps.nfvpe.site*.  

#### name
Name given to the connection

#### role
Type of connection.  Must be *inter-router*

#### host
IP Address or DNS name of the exposed, primary QDR

#### port
Which port to connect to.  Defaults to 20001 in standard SA install.

    connectors:
        - name: central-router
          role: inter-router
          host: blue.qdr-sa-telemetry.apps.nfvpe.site 
          port: 20001

## General Collecd/Barometer variables
### collectd_host
Address of the local QDR to connect to.  Should be 127.0.0.1
    collectd_host: 127.0.0.1

### collectd_port
Port to connect to for QDR.  Should be AMQP(5672)
     collectd_port: 5672

### collectd_interval
Default polling interval for all collectd plugins.  Specific plugins may override this value
    collectd_interval: 5

### collectd_image
Location of image on hub.docker.io    
    collectd_image: nfvpe/barometer-collectd

### collectd_image_tag
Tag of image
    collectd_image_tag: latest

### plugins
What plugins are to be included.  If a list is provided, only that plugins will be loaded.  If no list is provided. the following list will be enabled.

    plugins:
      - cpu
      - cpufreq
      - interface
      - disk
      - load
      - ovs_stats (if compute or controller)
      - ovs_events (if compute or controller)
      - hugepages (if virtualization present)
      - processes
      - uptime
      - df
      - virt (if virtualization present)
      - memory
      - intel_rdt

## Plugin specific variables
### collectd_plugin_connectivity_interfaces
The connectivity plugin monitors kernel network interfaces for link status changes.  The plugin is 
*event-driven* and reacts within 100's of milliseconds of an event occurring.  If an interface is 
specified, only that interface is monitored.  If no interface is specified, all interfaces are 
monitored.

    collectd_plugin_connectivity_interfaces:
      - eth0

### collectd_plugin_procevent_bufferlength
How many big is the event buffer?
    collectd_plugin_procevent_bufferlength: 1000
    
### collectd_plugin_procevent_processes
Which processes to monitor.  qemu-kvm should be included for virtual hosts.

    collectd_plugin_procevent_processes:
      - tuned
      - qemu-kvm
      
### collectd_plugin_ipmi_username/password
Login information for the host IPMI  

    collectd_plugin_ipmi_username: ADMIN
    collectd_plugin_ipmi_password: ADMIN

### collectd_plugin_virt_refreshinterval
How often to refresh the virt domain information.  Not recommended to go below 60 seconds.

    collectd_plugin_virt_refreshinterval: 60

### collectd_plugin_virt_hostnameformat
    collectd_plugin_virt_hostnameformat: hostname
    
### collectd_plugin_virt_plugininstanceformat    
    collectd_plugin_virt_plugininstanceformat: name
    
### collectd_plugin_virt_extrastats
What additional information should be included in virt collection.  A guest agent is necessary for fs_info.    

    collectd_plugin_virt_extrastats: "cpu_util disk_err domain_state fs_info job_stats_background perf vcpupin"

### collectd_plugin_cpu_interval
How often to sample the cpu information    

    collectd_plugin_cpu_interval: 5

### collectd_plugin_ceph
Setup monitoring of a ceph node.  Specify OSDs, MONs, and MDSs and there location.

    collectd_plugin_ceph:
      - daemon: "osd.0"
        socketpath: "/var/run/ceph/ceph-osd.0.asok"
      - daemon: "osd.1"
        socketpath: "/var/run/ceph/ceph-osd.1.asok"
      - daemon: "mon.a"
        socketpath: "/var/run/ceph/ceph-mon.ceph1.asok"
      - daemon: "mds.a"
        socketpath: "/var/run/ceph/ceph-mds.ceph1.asok"
