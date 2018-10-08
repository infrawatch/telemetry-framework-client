
# There are several variables which control what collectd + AMQP functionality is installed
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
        id_prefix: BAROMETER_ROUTER
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
### barometer_host
Address of the local QDR to connect to.  Should be 127.0.0.1
    barometer_host: 127.0.0.1
### barometer_port
Port to connect to for QDR.  Should be AMQP(5672)
     barometer_port: 5672
### barometer_interval
Default polling interval for all collectd plugins.  Specific plugins may override this value
    barometer_interval: 5
### barometer_image
Location of image on hub.docker.io    
    barometer_image: nfvpe/barometer-collectd
### barometer_image_tag
Tag of image
    barometer_image_tag: latest

## Plugin specific variables
### barometer_plugin_connectivity_interfaces
The connectivity plugin monitors kernel network interfaces for link status changes.  The plugin is *event-driven* and reacts within 100's of milliseconds of an event occurring.  If an interface is specified, only that interface is monitored.  If no interface is specified, all interfaces are monitored.
    barometer_plugin_connectivity_interfaces:
      - eth0
