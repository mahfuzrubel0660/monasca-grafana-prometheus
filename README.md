# monasca-grafana-prometheus



                **Monasca deployment on Kubernetes Instructions**
Introduction
This chart bootstraps Monasca deployment on a Kubernetes cluster using the Helm Package manager.
Prerequisites
Kubernetes 1.4+ with 1 master + 3 worker nodes
Openstack
1.	Create monasca user in keystone
openstack user create --domain default --password-prompt monasca
2.	Add monasca user to service tenant with admin role
openstack role add --project service --user monasca admin
3.	Create monasca service
openstack service create --name monasca --description "monasca" monitoring
4.	Create monasca endpoint
openstack endpoint create --region RegionOne monitoring public http://mon-api.brilliant.com.bd openstack endpoint create --region RegionOne monitoring admin http://mon-api.brilliant.com.bd openstack endpoint create --region RegionOne monitoring internal http://mon-api.brilliant.com.bd
1. Kafka Installation :
This chart will do the following:
Implement a dynamically scalable kafka cluster using Kubernetes StatefulSets
Implement a dynamically scalable zookeeper cluster as another Kubernetes StatefulSet required for the Kafka cluster above
Installing the Chart
To install the chart with the release name standalone-kafka in the monasca namespace:
# helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
# helm repo add incubator https://charts.helm.sh/incubator

# helm repo add stable https://charts.helm.sh/stable

# mkdir monasca-install
# cd monasca-install
# git clone https://github.com/cloud-operation/monasca-helm.git
# cd monasca-helm/charts
# helm install -f kafka_values.yaml --name standalone-kafka --namespace monasca incubator/kafka
# kubectl -n monasca get sts
NAME                         READY   AGE
standalone-kafka             3/3     5m26s
standalone-kafka-zookeeper   3/3     5m26s
# kubectl -n monasca get jobs
NAME                               COMPLETIONS   DURATION   AGE
standalone-kafka-config-8145cacc   0/1           5m44s      5m44s
2. Kafka manager Installation :
# helm install -f kafka-manager-values.yaml --name kafka-manager --namespace monasca stable/kafka-manager
# kubectl -n monasca get pods
NAME                                                   READY   STATUS      RESTARTS   AGE
kafka-manager-cf9d4bd4b-rm55z                          1/1     Running     0          53s
kafka-manager-kafka-manager-bootstrap-09113f4d-hnx2b   0/1     Completed   3          53s
standalone-kafka-0                                     2/2     Running     1          11m
standalone-kafka-1                                     2/2     Running     0          9m49s
standalone-kafka-2                                     2/2     Running     0          8m12s
standalone-kafka-config-8145cacc-5cl7z                 0/1     Completed   5          11m
standalone-kafka-zookeeper-0                           1/1     Running     0          11m
standalone-kafka-zookeeper-1                           1/1     Running     0          11m
standalone-kafka-zookeeper-2                           1/1     Running     0          9m58s
This will host kafka manager on http://kafka-manager.brilliant.com.bd using ingress routing. The default seven monasca topics and one default topic (consumer_offset) should also have been created.
3. Monasca Installation :
This chart bootstraps a Monasca deployment on a Kubernetes cluster using the Helm Package manager.
Installing the Chart:
# helm repo add monasca http://monasca.io/monasca-helm
# helm install -f mysql/values.yaml --name standalone-monasca-mysql --namespace monasca mysql/
# helm install -f influxdb/values.yaml --name standalone-monasca-influxdb --namespace monasca influxdb/
# helm install -f monasca/values.yaml --name monasca --namespace monasca monasca/

Or,

helm install monasca --namespace monasca --set api.keystone.username=monasca,api.keystone.password=Enp129s0f0,api.keystone.tenant_name=service,api.keystone.identity_url="http://192.168.40.10:5000",api.keystone.auth_url="http://192.168.40.10:5000",client.keystone.username=monasca,client.keystone.password=Enp129s0f0,client.keystone.user_domain_name=Default,client.keystone.project_name=service,client.keystone.project_domain_name=Default,client.keystone.url=http://192.168.40.10:5000/v3

# 

helm install monasca --namespace monasca -f monasca/values.yaml monasca/ --set api.keystone.username=monasca,api.keystone.password=Enp129s0f0,api.keystone.tenant_name=service,api.keystone.identity_url="http://118.67.209.114:5000",api.keystone.auth_url="http://118.67.209.114:5000",client.keystone.username=monasca,client.keystone.password=Enp129s0f0,client.keystone.user_domain_name=Default,client.keystone.project_name=service,client.keystone.project_domain_name=Default,client.keystone.url="http://118.67.209.114:5000/v3"


Check the deployment:
kubectl -n monasca get deploy
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
kafka-manager       1/1     1            1           22h
monasca-api         3/3     3            3           15d
monasca-client      1/1     1            1           15d
monasca-grafana     1/1     1            1           15d
monasca-influxdb    1/1     1            1           15d
monasca-memcached   1/1     1            1           15d
monasca-mysql       1/1     1            1           15d
monasca-persister   9/9     9            9           15d

helm ls –n monasca
NAME                            REVISION        UPDATED                         STATUS          CHART                   APP VERSION   NAMESPACE
kafka-manager                   1               Mon Mar  9 17:10:43 2020        DEPLOYED        kafka-manager-2.2.1     1.3.3.22      monasca
monasca                         1               Sun Feb 23 17:13:00 2020        DEPLOYED        monasca-0.6.4                         monasca
nginx-ingress                   1               Sun Jan 12 13:52:25 2020        DEPLOYED        nginx-ingress-1.28.2    0.26.2        kube-system
standalone-kafka                1               Mon Mar  9 17:02:51 2020        DEPLOYED        kafka-0.20.8            5.0.1         monasca
standalone-monasca-influxdb     1               Sun Feb 23 16:26:37 2020        DEPLOYED        influxdb-0.6.2-0.0.2                  monasca
standalone-monasca-mysql        1               Sun Feb 23 16:32:09 2020        DEPLOYED        mysql-0.2.4                           monasca



Deploying Ingress for Monasca API:
# kubectl apply -f monasca-api-ingress.yaml

apiVersion: extensions/v1beta1

kind: Ingress

metadata:

annotations:

nginx.ingress.kubernetes.io/rewrite-target: /

name: monasca-api-ingress

namespace: monasca

labels:

app: monasca-monasca

chart: monasca-0.6.4

component: api

spec:

rules:

- host: mon-api-test.brilliant.com.bd

http:

paths:

- backend:

serviceName: monasca-api

servicePort: 8070

path: /
This will host monasca api on http://mon-api.brilliant.com.bd using ingress routing
Modify monasca-client deployment to point to external keystone endpoint:
# kubectl -n monasca edit deploy monasca-api
...
        env:
        - name: OS_AUTH_URL
          value: http://cloud.brilliant.com.bd:5000/v3
        - name: OS_AUTH_URL
          value: http://118.67.209.114:5000/v3

...
This will point monasca-client to retrive token from http://cloud.brilliant.com.bd:5000/v3 (external keystone)
4. Patch Persistent Volumes to retain: It is very important to update the PVs created and set them to "Retain" from "Delete". Delete setting will delete the PVs once the pod gets deleted.

# kubectl patch pv -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}' pvc-372af896-6818-11e9-8d26-02c0777b771b
persistentvolume/pvc-372af896-6818-11e9-8d26-02c0777b771b patched
# ... (Repeat for all other PVs)
5. Deploy grafana dashboard for monasca
# kubectl apply -f grafana/configmap.yaml
# kubectl apply -f grafana/deployment.yaml
# kubectl apply -f grafana/service.yaml
# kubectl apply -f grafana/grafana-init.yaml
# kubectl apply -f grafana/grafana-ingress.yaml

# kubectl -n monasca get svc
kafka-manager                         ClusterIP   10.102.130.56    <none>        9000/TCP                     94m
monasca-api                           ClusterIP   10.111.198.134   <none>        8070/TCP                     92m
monasca-grafana                       ClusterIP   10.108.32.60     <none>        3000TCP                      28m
monasca-influxdb                      ClusterIP   10.97.0.249      <none>        8086/TCP                     92m
monasca-memcached                     ClusterIP   10.98.28.187     <none>        11211/TCP                    92m
monasca-mysql                         ClusterIP   10.105.211.107   <none>        3306/TCP                     92m
standalone-kafka                      ClusterIP   10.98.138.137    <none>        9092/TCP                     101m
standalone-kafka-headless             ClusterIP   None             <none>        9092/TCP                     101m
standalone-kafka-zookeeper            ClusterIP   10.108.75.69     <none>        2181/TCP                     101m
standalone-kafka-zookeeper-headless   ClusterIP   None             <none>        2181/TCP,3888/TCP,2888/TCP   101m

# kubectl -n monasca get ing
Open grafana on http://grafana-mon.brilliant.com.bd with keystone user "monasca" and password.

New Grafana Installation:
# helm repo add grafana https://grafana.github.io/helm-charts
# helm repo update
# 
# helm upgrade --install  grafana-monitoring --namespace=monasca grafana/grafana \
--set service.type=NodePort \
--set persistence.enabled=true \
--set persistence.size=5Gi

# kubectl get secret --namespace monasaca grafana-monitoring -o jsonpath="{.data.admin-password}" | base64 --decode ; echo  ------------- to find the password.


6. Deploy monasca-agent on all compute nodes
# yum install epel-release
yum install python-pip gcc python-devel

pip install --upgrade pip
pip install virtualenv
pip install zipp==1.2.0 # if required


mkdir monasca-agent
cd monasca-agent/

cp -R /usr/lib64/python2.7/site-packages/ujson* lib/python2.7/site-packages/

virtualenv  .
source bin/activate

pip install --no-cache-dir --upgrade monasca-agent==2.8.0

cp /usr/lib64/python2.7/site-packages/libvirt.py lib/python2.7/site-packages/
cp /usr/lib64/python2.7/site-packages/libvirt.pyc lib/python2.7/site-packages/
cp /usr/lib64/python2.7/site-packages/libvirt.pyo lib/python2.7/site-packages/
cp /usr/lib64/python2.7/site-packages/libvirtmod* lib/python2.7/site-packages/
cp /usr/lib64/python2.7/site-packages/libvirt_python-4.5.0-py2.7.egg-info lib/python2.7/site-packages/

pip install monasca-agent[libvirt]==2.8.0

monasca-setup -u monasca -p ******* --check_frequency 120 --project_name service --project_domain_name Default --region_name RegionOne --user_domain_name Default -s monitoring --endpoint_type adminURL --keystone_url http://cloud.brilliant.com.bd:5000 --monasca_url http://mon-api.brilliant.com.bd/v2.0 --config_dir /etc/monasca/agent --log_dir /var/log/monasca/agent --log_level INFO --overwrite

monasca-setup -d libvirt --overwrite

usermod -aG root mon-agent

vi /etc/monasca/agent/agent.yaml

Api:
  amplifier: 0
  backlog_send_rate: 1000
  ca_file: null
  endpoint_type: adminURL
  insecure: false
  keystone_url: http://cloud.brilliant.com.bd:5000
  max_batch_size: 0
  max_buffer_size: 1000
  max_measurement_buffer_size: -1
  password: monasca-openstack-user-password
  project_domain_id: null
  project_domain_name: Default
  project_id: null
  project_name: service
  region_name: RegionOne
  service_type: null
  url: http://mon-api.brilliant.com.bd/v2.0
  user_domain_id: null
  user_domain_name: Default
  username: monasca
Logging:
  collector_log_file: /var/log/monasca/agent/collector.log
  enable_logrotate: true
  forwarder_log_file: /var/log/monasca/agent/forwarder.log
  log_level: INFO
  statsd_log_file: /var/log/monasca/agent/statsd.log
Main:
  check_freq: 120
  collector_restart_interval: 24
  dimensions:
    service: monitoring
  hostname: compute3.brilliant.com.bd
  num_collector_threads: 1
  pool_full_max_retries: 4
  sub_collection_warn: 6
Statsd:
  monasca_statsd_port: 8125



vi /etc/monasca/agent/conf.d/libvirt.yaml

init_config:
  auth_url: http://cloud.brilliant.com.bd:5000
  cache_dir: /dev/shm
  customer_metadata:
  - scale_group
  disk_collection_period: 0
  max_ping_concurrency: 8
  metadata:
  - scale_group
  nova_refresh: 14400
  password:  nova-password-here
  ping_check: true
  project_name: service
  project_domain_id: default
  region: RegionOne
  username: nova
  user_domain_id: default
  vm_cpu_check_enable: true
  vm_disks_check_enable: true
  vm_extended_disks_check_enable: false
  vm_network_check_enable: true
  vm_ping_check_enable: true
  vm_probation: 120
  vnic_collection_period: 0
instances: []

systemctl status monasca-agent.target
systemctl enable monasca-agent.target
systemctl restart monasca-agent.target

systemctl status monasca-collector
systemctl status monasca-forwarder
systemctl status monasca-statsd




CEPH cluster deploy in grafana:

First have to enable promethus module in ceph cluster. 
ceph mgr module enable prometheus



#########################################################
root@master:~/ceph-alert# cat prometheus-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
  labels:
    app: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus
        ports:
        - containerPort: 9090

        volumeMounts:
        - mountPath: /etc/prometheus/prometheus.yml
          name: config
          subPath: prometheus.yml
        - mountPath: /etc/prometheus/rules.yml
          name: rules
          subPath: rules.yml

      volumes:
      - configMap:
          defaultMode: 420
          name: prometheus-config
        name: config

      - configMap:
          defaultMode: 420
          name: alert-rule
        name: rules

###############################################

root@master:~/ceph-alert# cat prometheus-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus-svc
  namespace: monitoring
spec:
  ports:
  - name: service
    port: 9090
    protocol: TCP
    targetPort: 9090
  selector:
    app: prometheus

#################################################
# cat prometheus-config-map.yml
apiVersion: v1
data:
  prometheus.yml: |+
    global:
      scrape_interval: 10s

    rule_files:
      - /etc/prometheus/rules.yml


    alerting:
      alertmanagers:
        - static_configs:
          - targets:
            # Alertmanager's default port is 9093
            - alertmanager:9093

    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
      - job_name: 'ceph'
        static_configs:
          - targets: ['118.67.220.34:9283']
    #  - job_name: 'node-exporter'
    #    static_configs:
    #      - targets: ['localhost:9100']

kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring



# root@master:~/ceph-alert# cat rule
groups:
  - name: cluster health
    rules:
      - alert: health error
        expr: ceph_health_status == 2
        for: 5m
        labels:
          severity: critical
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.2.1
        annotations:
          description: >
            Ceph in HEALTH_ERROR state for more than 5 minutes.
            Please check "ceph health detail" for more information.

      - alert: health warn
        expr: ceph_health_status == 1
        for: 15m
        labels:
          severity: warning
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.2.2
        annotations:
          description: >
            Ceph has been in HEALTH_WARN for more than 15 minutes.
            Please check "ceph health detail" for more information.

  - name: mon
    rules:
      - alert: low monitor quorum count
        expr: sum(ceph_mon_quorum_status) < 3
        labels:
          severity: critical
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.3.1
        annotations:
          description: |
            Monitor count in quorum is below three.

            Only {{ $value }} of {{ with query "count(ceph_mon_quorum_status)" }}{{ . | first | value }}{{ end }} monitors are active.

            The following monitors are down:
            {{- range query "(ceph_mon_quorum_status == 0) + on(ceph_daemon) group_left(hostname) (ceph_mon_metadata * 0)" }}
              - {{ .Labels.ceph_daemon }} on {{ .Labels.hostname }}
            {{- end }}

  - name: osd
    rules:
      - alert: 10% OSDs down
        expr: count(ceph_osd_up == 0) / count(ceph_osd_up) * 100 >= 10
        labels:
          severity: critical
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.4.1
        annotations:
          description: |
            {{ $value | humanize }}% or {{ with query "count(ceph_osd_up == 0)" }}{{ . | first | value }}{{ end }} of {{ with query "count(ceph_osd_up)" }}{{ . | first | value }}{{ end }} OSDs are down (≥ 10%).

            The following OSDs are down:
            {{- range query "(ceph_osd_up * on(ceph_daemon) group_left(hostname) ceph_osd_metadata) == 0" }}
              - {{ .Labels.ceph_daemon }} on {{ .Labels.hostname }}
            {{- end }}

      - alert: OSD down
        expr: count(ceph_osd_up == 0) > 0
        for: 15m
        labels:
          severity: warning
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.4.2
        annotations:
          description: |
            {{ $s := "" }}{{ if gt $value 1.0 }}{{ $s = "s" }}{{ end }}
            {{ $value }} OSD{{ $s }} down for more than 15 minutes.

            {{ $value }} of {{ query "count(ceph_osd_up)" | first | value }} OSDs are down.

            The following OSD{{ $s }} {{ if eq $s "" }}is{{ else }}are{{ end }} down:
              {{- range query "(ceph_osd_up * on(ceph_daemon) group_left(hostname) ceph_osd_metadata) == 0"}}
                - {{ .Labels.ceph_daemon }} on {{ .Labels.hostname }}
              {{- end }}

      - alert: OSDs near full
        expr: |
          (
            ((ceph_osd_stat_bytes_used / ceph_osd_stat_bytes) and on(ceph_daemon) ceph_osd_up == 1)
            * on(ceph_daemon) group_left(hostname) ceph_osd_metadata
          ) * 100 > 90
        for: 5m
        labels:
          severity: critical
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.4.3
        annotations:
          description: >
            OSD {{ $labels.ceph_daemon }} on {{ $labels.hostname }} is
            dangerously full: {{ $value | humanize }}%

      - alert: flapping OSD
        expr: |
          (
            rate(ceph_osd_up[5m])
            * on(ceph_daemon) group_left(hostname) ceph_osd_metadata
          ) * 60 > 1
        labels:
          severity: warning
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.4.4
        annotations:
          description: >
            OSD {{ $labels.ceph_daemon }} on {{ $labels.hostname }} was
            marked down and back up at {{ $value | humanize }} times once a
            minute for 5 minutes.

      # alert on high deviation from average PG count
      - alert: high pg count deviation
        expr: |
          abs(
            (
              (ceph_osd_numpg > 0) - on (job) group_left avg(ceph_osd_numpg > 0) by (job)
            ) / on (job) group_left avg(ceph_osd_numpg > 0) by (job)
          ) * on(ceph_daemon) group_left(hostname) ceph_osd_metadata > 0.30
        for: 5m
        labels:
          severity: warning
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.4.5
        annotations:
          description: >
            OSD {{ $labels.ceph_daemon }} on {{ $labels.hostname }} deviates
            by more than 30% from average PG count.
      # alert on high commit latency...but how high is too high
  - name: mds
    rules:
    # no mds metrics are exported yet
  - name: mgr
    rules:
    # no mgr metrics are exported yet
  - name: pgs
    rules:
      - alert: pgs inactive
        expr: ceph_pool_metadata * on(pool_id,instance) group_left() (ceph_pg_total - ceph_pg_active) > 0
        for: 5m
        labels:
          severity: critical
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.7.1
        annotations:
          description: >
            {{ $value }} PGs have been inactive for more than 5 minutes in pool {{ $labels.name }}.
            Inactive placement groups aren't able to serve read/write
            requests.
      - alert: pgs unclean
        expr: ceph_pool_metadata * on(pool_id,instance) group_left() (ceph_pg_total - ceph_pg_clean) > 0
        for: 15m
        labels:
          severity: warning
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.7.2
        annotations:
          description: >
            {{ $value }} PGs haven't been clean for more than 15 minutes in pool {{ $labels.name }}.
            Unclean PGs haven't been able to completely recover from a
            previous failure.
  - name: nodes
    rules:
      - alert: root volume full
        expr: node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"} * 100 < 5
        for: 5m
        labels:
          severity: critical
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.8.1
        annotations:
          description: >
            Root volume (OSD and MON store) is dangerously full: {{ $value | humanize }}% free.

      # alert on nic packet errors and drops rates > 1% packets/s
      - alert: network packets dropped
        expr: |
          (
            increase(node_network_receive_drop_total{device!="lo"}[1m]) +
            increase(node_network_transmit_drop_total{device!="lo"}[1m])
          ) / (
            increase(node_network_receive_packets_total{device!="lo"}[1m]) +
            increase(node_network_transmit_packets_total{device!="lo"}[1m])
          ) >= 0.0001 or (
            increase(node_network_receive_drop_total{device!="lo"}[1m]) +
            increase(node_network_transmit_drop_total{device!="lo"}[1m])
          ) >= 10
        labels:
          severity: warning
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.8.2
        annotations:
          description: >
            Node {{ $labels.instance }} experiences packet drop > 0.01% or >
            10 packets/s on interface {{ $labels.device }}.

      - alert: network packet errors
        expr: |
          (
            increase(node_network_receive_errs_total{device!="lo"}[1m]) +
            increase(node_network_transmit_errs_total{device!="lo"}[1m])
          ) / (
            increase(node_network_receive_packets_total{device!="lo"}[1m]) +
            increase(node_network_transmit_packets_total{device!="lo"}[1m])
          ) >= 0.0001 or (
            increase(node_network_receive_errs_total{device!="lo"}[1m]) +
            increase(node_network_transmit_errs_total{device!="lo"}[1m])
          ) >= 10
        labels:
          severity: warning
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.8.3
        annotations:
          description: >
            Node {{ $labels.instance }} experiences packet errors > 0.01% or
            > 10 packets/s on interface {{ $labels.device }}.

      - alert: storage filling up
        expr: |
          predict_linear(node_filesystem_free_bytes[2d], 3600 * 24 * 5) *
          on(instance) group_left(nodename) node_uname_info < 0
        labels:
          severity: warning
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.8.4
        annotations:
          description: >
            Mountpoint {{ $labels.mountpoint }} on {{ $labels.nodename }}
            will be full in less than 5 days assuming the average fill-up
            rate of the past 48 hours.

      - alert: MTU Mismatch
        expr: node_network_mtu_bytes{device!="lo"} * (node_network_up{device!="lo"} > 0) != on() group_left() (quantile(0.5, node_network_mtu_bytes{device!="lo"}))
        labels:
          severity: warning
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.8.5
        annotations:
          description: >
            Node {{ $labels.instance }} has a different MTU size ({{ $value }})
            than the median value on device {{ $labels.device }}.

  - name: pools
    rules:
      - alert: pool full
        expr: |
          ceph_pool_stored / (ceph_pool_stored + ceph_pool_max_avail)
          * on(pool_id) group_right ceph_pool_metadata * 100 > 90
        labels:
          severity: critical
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.9.1
        annotations:
          description: Pool {{ $labels.name }} at {{ $value | humanize }}% capacity.

      - alert: pool filling up
        expr: |
          (
            predict_linear(ceph_pool_stored[2d], 3600 * 24 * 5)
            >= ceph_pool_stored + ceph_pool_max_avail
          ) * on(pool_id) group_left(name) ceph_pool_metadata
        labels:
          severity: warning
          type: ceph_default
          oid: 1.3.6.1.4.1.50495.15.1.2.9.2
        annotations:
          description: >
            Pool {{ $labels.name }} will be full in less than 5 days
            assuming the average fill-up rate of the past 48 hours.

  - name: healthchecks
    rules:
      - alert: Slow OSD Ops
        expr: ceph_healthcheck_slow_ops > 0
        for: 30s
        labels:
          severity: warning
          type: ceph_default
        annotations:
          description: >
            {{ $value }} OSD requests are taking too long to process (osd_op_complaint_time exceeded)

####################################################

root@master:~/ceph-alert# cat alertmanager-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alertmanager
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alertmanager
  template:
    metadata:
      name: alertmanager
      labels:
        app: alertmanager
    spec:
      containers:
      - name: alertmanager
        image: prom/alertmanager
        args:
          - "--config.file=/etc/alertmanager/alertmanager.yml"
          - "--storage.path=/alertmanager"
        ports:
        - name: alertmanager
          containerPort: 9093
        volumeMounts:
        - name: config-volume
          mountPath: /etc/alertmanager
          #        - name: templates-volume
          #          mountPath: /etc/alertmanager-templates
        - name: alertmanager
          mountPath: /alertmanager
      volumes:
      - name: config-volume
        configMap:
          name: alertmanager-config
          #      - name: templates-volume
          #        configMap:
          #          name: alertmanager-templates
      - name: alertmanager
        emptyDir: {}

##########################################################


root@master:~/ceph-alert# cat alertmanager-svc.yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/path: '/alertmanager/metrics'
  labels:
    name: alertmanager
  name: alertmanager
  namespace: monitoring
spec:
  selector:
    app: alertmanager
  #type: NodePort
  ports:
  - name: alertmanager
    protocol: TCP
    port: 9093
    targetPort: 9093

###########################################################


root@master:~/ceph-alert# cat alertmanager-config-cm.yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: alertmanager-config
  namespace: monitoring
data:
  alertmanager.yml: |-
    global: {}
    #   receivers:
    #- name: default-receiver
    route:
      group_interval: 5m
      group_wait: 10s
      #receiver: default-receiver
      repeat_interval: 3h
      receiver: 'email'
    receivers:
    - name: 'email'
      email_configs:
      - to: 'cloud.operation@brilliant.com.bd'
        from: 'monitor@brilliant.com.bd'
        smarthost: mta.brilliant.com.bd:25
        auth_username: 'monitor@brilliant.com.bd'
        auth_identity: 'monitor@brilliant.com.bd'
        auth_password: 'Cl0ud-M0n!t0r'
    inhibit_rules:
     - source_match:
        severity: 'critical'
       target_match:
        severity: 'warning'
       equal: ['alertname', 'dev', 'instance']


###########################################################

root@master:~/ceph-alert# cat  alert_rule-cm.yaml
apiVersion: v1
data:
  rules.yml: |
    groups:
      - name: cluster health
        rules:
          - alert: health error
            expr: ceph_health_status == 2
            for: 5m
            labels:
              severity: critical
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.2.1
            annotations:
              description: >
                Ceph in HEALTH_ERROR state for more than 5 minutes.
                Please check "ceph health detail" for more information.

          - alert: health warn
            expr: ceph_health_status == 1
            for: 15m
            labels:
              severity: warning
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.2.2
            annotations:
              description: >
                Ceph has been in HEALTH_WARN for more than 15 minutes.
                Please check "ceph health detail" for more information.

      - name: mon
        rules:
          - alert: low monitor quorum count
            expr: sum(ceph_mon_quorum_status) < 3
            labels:
              severity: critical
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.3.1
            annotations:
              description: |
                Monitor count in quorum is below three.

                Only {{ $value }} of {{ with query "count(ceph_mon_quorum_status)" }}{{ . | first | value }}{{ end }} monitors are active.

                The following monitors are down:
                {{- range query "(ceph_mon_quorum_status == 0) + on(ceph_daemon) group_left(hostname) (ceph_mon_metadata * 0)" }}
                  - {{ .Labels.ceph_daemon }} on {{ .Labels.hostname }}
                {{- end }}

      - name: osd
        rules:
          - alert: 10% OSDs down
            expr: count(ceph_osd_up == 0) / count(ceph_osd_up) * 100 >= 10
            labels:
              severity: critical
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.4.1
            annotations:
              description: |
                {{ $value | humanize }}% or {{ with query "count(ceph_osd_up == 0)" }}{{ . | first | value }}{{ end }} of {{ with query "count(ceph_osd_up)" }}{{ . | first | value }}{{ end }} OSDs are down (≥ 10%).

                The following OSDs are down:
                {{- range query "(ceph_osd_up * on(ceph_daemon) group_left(hostname) ceph_osd_metadata) == 0" }}
                  - {{ .Labels.ceph_daemon }} on {{ .Labels.hostname }}
                {{- end }}

          - alert: OSD down
            expr: count(ceph_osd_up == 0) > 0
            for: 15m
            labels:
              severity: warning
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.4.2
            annotations:
              description: |
                {{ $s := "" }}{{ if gt $value 1.0 }}{{ $s = "s" }}{{ end }}
                {{ $value }} OSD{{ $s }} down for more than 15 minutes.

                {{ $value }} of {{ query "count(ceph_osd_up)" | first | value }} OSDs are down.

                The following OSD{{ $s }} {{ if eq $s "" }}is{{ else }}are{{ end }} down:
                  {{- range query "(ceph_osd_up * on(ceph_daemon) group_left(hostname) ceph_osd_metadata) == 0"}}
                    - {{ .Labels.ceph_daemon }} on {{ .Labels.hostname }}
                  {{- end }}

          - alert: OSDs near full
            expr: |
              (
                ((ceph_osd_stat_bytes_used / ceph_osd_stat_bytes) and on(ceph_daemon) ceph_osd_up == 1)
                * on(ceph_daemon) group_left(hostname) ceph_osd_metadata
              ) * 100 > 90
            for: 5m
            labels:
              severity: critical
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.4.3
            annotations:
              description: >
                OSD {{ $labels.ceph_daemon }} on {{ $labels.hostname }} is
                dangerously full: {{ $value | humanize }}%

          - alert: flapping OSD
            expr: |
              (
                rate(ceph_osd_up[5m])
                * on(ceph_daemon) group_left(hostname) ceph_osd_metadata
              ) * 60 > 1
            labels:
              severity: warning
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.4.4
            annotations:
              description: >
                OSD {{ $labels.ceph_daemon }} on {{ $labels.hostname }} was
                marked down and back up at {{ $value | humanize }} times once a
                minute for 5 minutes.

          # alert on high deviation from average PG count
         # - alert: high pg count deviation
         #   expr: |
         #     abs(
         #       (
         #         (ceph_osd_numpg > 0) - on (job) group_left avg(ceph_osd_numpg > 0) by (job)
         #       ) / on (job) group_left avg(ceph_osd_numpg > 0) by (job)
         #     ) * on(ceph_daemon) group_left(hostname) ceph_osd_metadata > 0.30
         #   for: 5m
         #   labels:
         #     severity: warning
         #     type: ceph_default
         #     oid: 1.3.6.1.4.1.50495.15.1.2.4.5
         #   annotations:
         #     description: >
         #       OSD {{ $labels.ceph_daemon }} on {{ $labels.hostname }} deviates
         #       by more than 30% from average PG count.
          # alert on high commit latency...but how high is too high
      - name: mds
        rules:
        # no mds metrics are exported yet
      - name: mgr
        rules:
        # no mgr metrics are exported yet
      - name: pgs
        rules:
          - alert: pgs inactive
            expr: ceph_pool_metadata * on(pool_id,instance) group_left() (ceph_pg_total - ceph_pg_active) > 0
            for: 5m
            labels:
              severity: critical
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.7.1
            annotations:
              description: >
                {{ $value }} PGs have been inactive for more than 5 minutes in pool {{ $labels.name }}.
                Inactive placement groups aren't able to serve read/write
                requests.
          - alert: pgs unclean
            expr: ceph_pool_metadata * on(pool_id,instance) group_left() (ceph_pg_total - ceph_pg_clean) > 0
            for: 15m
            labels:
              severity: warning
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.7.2
            annotations:
              description: >
                {{ $value }} PGs haven't been clean for more than 15 minutes in pool {{ $labels.name }}.
                Unclean PGs haven't been able to completely recover from a
                previous failure.
      - name: nodes
        rules:
          - alert: root volume full
            expr: node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"} * 100 < 5
            for: 5m
            labels:
              severity: critical
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.8.1
            annotations:
              description: >
                Root volume (OSD and MON store) is dangerously full: {{ $value | humanize }}% free.

          # alert on nic packet errors and drops rates > 1% packets/s
          - alert: network packets dropped
            expr: |
              (
                increase(node_network_receive_drop_total{device!="lo"}[1m]) +
                increase(node_network_transmit_drop_total{device!="lo"}[1m])
              ) / (
                increase(node_network_receive_packets_total{device!="lo"}[1m]) +
                increase(node_network_transmit_packets_total{device!="lo"}[1m])
              ) >= 0.0001 or (
                increase(node_network_receive_drop_total{device!="lo"}[1m]) +
                increase(node_network_transmit_drop_total{device!="lo"}[1m])
              ) >= 10
            labels:
              severity: warning
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.8.2
            annotations:
              description: >
                Node {{ $labels.instance }} experiences packet drop > 0.01% or >
                10 packets/s on interface {{ $labels.device }}.

          - alert: network packet errors
            expr: |
              (
                increase(node_network_receive_errs_total{device!="lo"}[1m]) +
                increase(node_network_transmit_errs_total{device!="lo"}[1m])
              ) / (
                increase(node_network_receive_packets_total{device!="lo"}[1m]) +
                increase(node_network_transmit_packets_total{device!="lo"}[1m])
              ) >= 0.0001 or (
                increase(node_network_receive_errs_total{device!="lo"}[1m]) +
                increase(node_network_transmit_errs_total{device!="lo"}[1m])
              ) >= 10
            labels:
              severity: warning
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.8.3
            annotations:
              description: >
                Node {{ $labels.instance }} experiences packet errors > 0.01% or
                > 10 packets/s on interface {{ $labels.device }}.

          - alert: storage filling up
            expr: |
              predict_linear(node_filesystem_free_bytes[2d], 3600 * 24 * 5) *
              on(instance) group_left(nodename) node_uname_info < 0
            labels:
              severity: warning
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.8.4
            annotations:
              description: >
                Mountpoint {{ $labels.mountpoint }} on {{ $labels.nodename }}
                will be full in less than 5 days assuming the average fill-up
                rate of the past 48 hours.

          - alert: MTU Mismatch
            expr: node_network_mtu_bytes{device!="lo"} * (node_network_up{device!="lo"} > 0) != on() group_left() (quantile(0.5, node_network_mtu_bytes{device!="lo"}))
            labels:
              severity: warning
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.8.5
            annotations:
              description: >
                Node {{ $labels.instance }} has a different MTU size ({{ $value }})
                than the median value on device {{ $labels.device }}.

      - name: pools
        rules:
          - alert: pool full
            expr: |
              ceph_pool_stored / (ceph_pool_stored + ceph_pool_max_avail)
              * on(pool_id) group_right ceph_pool_metadata * 100 > 90
            labels:
              severity: critical
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.9.1
            annotations:
              description: Pool {{ $labels.name }} at {{ $value | humanize }}% capacity.

          - alert: pool filling up
            expr: |
              (
                predict_linear(ceph_pool_stored[2d], 3600 * 24 * 5)
                >= ceph_pool_stored + ceph_pool_max_avail
              ) * on(pool_id) group_left(name) ceph_pool_metadata
            labels:
              severity: warning
              type: ceph_default
              oid: 1.3.6.1.4.1.50495.15.1.2.9.2
            annotations:
              description: >
                Pool {{ $labels.name }} will be full in less than 5 days
                assuming the average fill-up rate of the past 48 hours.

      - name: healthchecks
        rules:
          - alert: Slow OSD Ops
            expr: ceph_healthcheck_slow_ops > 0
            for: 30s
            labels:
              severity: warning
              type: ceph_default
            annotations:
              description: >
                {{ $value }} OSD requests are taking too long to process (osd_op_complaint_time exceeded)
kind: ConfigMap
metadata:
  name: alert-rule
  namespace: monitoring

########################################################

















