# <div align="center"><img src="./fig.svg" width="48"> FIG stack</div>

<p align="center">A minimal logging architecture in Kubernetes
<br>
<strong>FluentBit + InfluxDB + Grafana ⎈ FIG</strong>
</p>

![FIG Data Flow](./FIG-Diagram.drawio.png)


#Below URL you can validate & visualize the configMap file.

    https://cloud.calyptia.com/visualizer

##COMMANDS:

    1. kubectl create ns monitoring
    2. git clone https://github.com/iqrabismi/FIG.git
    3. helm upgrade --install influxdb influxdb -n monitoring (Install Influx DB)
    4. helm upgrade --install fluent-bit fluent-bit -n monitoring (Install Fluent Bit)
    5. helm upgrade --install grafana grafana -n monitoring (Install Grafana)

** Try to use latest verison of Fluent Bit. Here it is used as  version: 1.8

How to exclude any path's  log (pod) not to send to influx DB. Example: Any pod in the namespace of kube-system & monitoring.
Specify in Values.yaml:

    path: /var/log/containers/*.log
    excludepath: /var/log/containers/*_monitoring_*.log, /var/log/containers/*_kube-system_*.log
    
NOTE: Make sure K8S-Logging.Exclude is On, else exclude path will not work.

How to specify any Influx DB host details which is not running inside cluster.
Specify in Values.yaml:
    
    type: Influx
    influx:
     host:  [Host IP Address or DNS]  
     port:  8086  
     bucket:  [Influx DB Name]  
     org:  [Influx DB Organization Name]  
     user:  fluentuser  
     token:  [Your Influx DB Token]  
     sequence_tag:  _seq
  
  
  
 AND in configMap.yaml:
 
  fluent-bit-input.conf: |
  
    [INPUT]
        Name             tail
        Path             {{ .Values.input.tail.path }}
        Exclude_Path     {{ .Values.input.tail.excludepath }}
        Parser           {{ .Values.input.tail.parser }}
        Tag              {{ .Values.filter.kubeTag }}
        Refresh_Interval 5
        Mem_Buf_Limit    {{ .Values.input.tail.memBufLimit }}
        Skip_Long_Lines  On
  
  NOTE: Make sure that, in Values.Yaml file, under filter section, kubeTag should contain the part of the text of those POD's name, whoes log fluent-bit will collect.

 fluent-bit-output.conf: |
 
    [INPUT]
        Name cpu
        Tag  <your desired tag name>
    [OUTPUT]
        Name          influxdb
        Match         *
        Host          {{ .Values.backend.influx.host }}
        Port          {{ .Values.backend.influx.port }}
        #Database     {{ .Values.backend.influx.database }}
        Bucket        {{ .Values.backend.influx.bucket }}
        Org           {{ .Values.backend.influx.org }}
        Sequence_Tag  {{ .Values.backend.influx.sequence_tag }}
        HTTP_User     {{ .Values.backend.influx.user }}
        #HTTP_Passwd  {{ .Values.backend.influx.password }}
        HTTP_Token    {{ .Values.backend.influx.token }}
        Tag_Keys      method path


In case of Influx DB & Grafana Service,
You can specify service Type as : LoadBalancer if you want to access these in public ip.


## Prerequisites
* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* [Helm 3](https://helm.sh/docs/intro/install/)
