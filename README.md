# Lenovo Confluent Prometheus Exporter

This simple exporter uses the Lenovo Confluent and Prometheus python clients to query and export nodes sensors and health status.<br> 

## Installtion
Current exporter implementation assume it will run on locally on the confluent servers (either standalone or collective)<br>
<br>Install the prometheus-client (https://github.com/prometheus/client_python) 
```
pip install prometheus-client
```
  
<br>Run the script directly or as a systemd service (see example systemd unit file)
```
/usr/sbin/confluent_exporter -f /etc/confluent/exporter/config.yaml
```

<br>The behavior of the exporter is customized using a yaml configuration file. By default, the exporter will look for `etc/confluent/exporter/config.yaml`. <br>
<br>To specify different file:  
```
/path/to/confluent_exporter [-f | --configfile] <config file>
```
Default is /etc/confluent/exporter/config.yaml
<br>Exporter and configuration file should be owned by the Confluent user.<br><br>

## Configuration
### Sensors
Sensors are defined per group under the _groups_ statement. each group entry is defined by the group name and list of sensors to collect. While nodes can be in multiple groups, the exporter will perform only one collection per node per sensor 
```
groups:
 - name: "group1"
   sensors:
   - inlet_temp
   - avg_cpu_temp
   - health
   - total_power
   - dc_energy

 - name: "group2"  # useful for devices such as enclosure managers/SMMs
   sensors:
   - health
 ```

#### Current supported sensors:
**total_power** - Total node current power consumption (Watt)<br>
**avg_cpu_temp** - Average CPU temperature (Celsius)<br>
**inlet_temp** - Server inlet air temperature (Celsius)<br>
**health** - Node health status - OK (0) / Warning (1) / Critical or Failed (2) / Unknown, or Error in fetching (3)<br>
**dc_energy** - Total DC Energy consumed by the node (kWh)<br>
**cpu_power** - Node CPU power consumption (Watt)<br>
**mem_power** - Node Memory power consumption (Watt)<br>
Adding sensors is trivial. More sensors will be added soon.<br><br>

### Adding lables to nodes: 
**note** - It is better to minimize the number of labels.<br>
The exporter will add the 'node' label with the node name.
Additional labels can be added in the following way -<br><br>
add the labels section with list of labels
```
labels:
- rack
- nodetype
```

<br>Add corresponding attribute to node with the following format - 
`custom.labels.<label name>`
For example:
```
nodeattrib rack1 custom.lables.rack=rack1
nodeattrib service custom.lables.nodetype=service
```

### Port 
The default exporter port is 13101.<br>
To change the exporter port add the following to the configuration file: 
```
port: <port num>
```

### Collective
Setting collective to `True` will ensure that each confluent server will collect data only from nodes where the confluent server is the current active collective manager
```
collective: True
```
Default is to ignore collective (False)
