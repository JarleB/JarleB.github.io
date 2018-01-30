---
# HA setup with ansible playbook

## Inventory

Contents of /etc/ansible/hosts

```
[OSEv3:vars]

###########################################################################
### Ansible Vars
###########################################################################
timeout=60
ansible_become=yes
ansible_ssh_user=ec2-user


# disable memory check, as we are not a production environment
openshift_disable_check="memory_availability"




openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/master/htpasswd'}]
#osm_default_subdomain=cloudapps-165f.oslab.opentlc.com
osm_default_subdomain=apps.165f.example.opentlc.com

#os_sdn_network_plugin_name='redhat/openshift-ovs-multitenant'
os_sdn_network_plugin_name='redhat/openshift-ovs-networkpolicy'

openshift_hosted_router_selector='env=infra'
openshift_hosted_router_replicas=2
osm_default_node_selector='env=app'
openshift_hosted_registry_selector='env=infra'

#openshift_master_default_subdomain=cloudapps-165f.oslab.opentlc.com
openshift_master_default_subdomain=apps.165f.example.opentlc.com
deployment_type=openshift-enterprise

openshift_master_cluster_hostname=loadbalancer.165f.example.opentlc.com
openshift_master_cluster_public_hostname=loadbalancer.165f.example.opentlc.com

#*.apps.${GUID}.example.opentlc.com

#openshift_master_ca_certificate={'certfile': '/root/intermediate_ca.crt', 'keyfile': '/root/intermediate_ca.key'}

openshift_logging_install_logging=True
openshift_logging_storage_kind=nfs
openshift_logging_storage_access_modes=['ReadWriteOnce']
openshift_logging_storage_nfs_directory=/exports
openshift_logging_storage_nfs_options='*(rw,root_squash)'
openshift_logging_storage_volume_name=logging-es
openshift_logging_storage_volume_size=10Gi
openshift_logging_storage_labels={'storage': 'logging'}
openshift_logging_es_cluster_size=1
openshift_logging_es_memory_limit=2Gi
openshift_logging_es_nodeselector={"env":"infra"}
openshift_logging_kibana_nodeselector={"env":"infra"}
openshift_logging_curator_nodeselector={"env":"infra"}


openshift_metrics_install_metrics=True
openshift_metrics_storage_kind=nfs
openshift_metrics_storage_access_modes=['ReadWriteOnce']
openshift_metrics_storage_nfs_directory=/exports
openshift_metrics_storage_nfs_options='*(rw,root_squash)'
openshift_metrics_storage_volume_name=metrics
openshift_metrics_storage_volume_size=10Gi
openshift_metrics_storage_labels={'storage': 'metrics'}
openshift_metrics_cassandra_nodeselector={"env":"infra"}
openshift_metrics_hawkular_nodeselector={"env":"infra"}
openshift_metrics_heapster_nodeselector={"env":"infra"}

[OSEv3:children]
lb
masters
etcd
nodes
nfs

[lb]
loadbalancer1.165f.internal host_zone=eu-central-1b

[masters]
master3.165f.internal host_zone=eu-central-1b
master1.165f.internal host_zone=eu-central-1b
master2.165f.internal host_zone=eu-central-1b

[etcd]
master3.165f.internal host_zone=eu-central-1b
master1.165f.internal host_zone=eu-central-1b
master2.165f.internal host_zone=eu-central-1b

[nodes]
## These are the masters
master3.165f.internal openshift_hostname=master3.165f.internal  openshift_node_labels="{'logging':'true','openshift_schedulable':'False','cluster': '165f', 'zone': 'eu-central-1b'}"
master1.165f.internal openshift_hostname=master1.165f.internal  openshift_node_labels="{'logging':'true','openshift_schedulable':'False','cluster': '165f', 'zone': 'eu-central-1b'}"
master2.165f.internal openshift_hostname=master2.165f.internal  openshift_node_labels="{'logging':'true','openshift_schedulable':'False','cluster': '165f', 'zone': 'eu-central-1b'}"

## These are infranodes
infranode2.165f.internal openshift_hostname=infranode2.165f.internal  openshift_node_labels="{'logging':'true','cluster': '165f', 'env':'infra', 'zone': 'eu-central-1b'}"
infranode1.165f.internal openshift_hostname=infranode1.165f.internal  openshift_node_labels="{'logging':'true','cluster': '165f', 'env':'infra', 'zone': 'eu-central-1b'}"

## These are regular nodes
node3.165f.internal openshift_hostname=node3.165f.internal  openshift_node_labels="{'logging':'true','cluster': '165f', 'env':'app', 'zone': 'eu-central-1b'}"
node2.165f.internal openshift_hostname=node2.165f.internal  openshift_node_labels="{'logging':'true','cluster': '165f', 'env':'app', 'zone': 'eu-central-1b'}"
node1.165f.internal openshift_hostname=node1.165f.internal  openshift_node_labels="{'logging':'true','cluster': '165f', 'env':'app', 'zone': 'eu-central-1b'}"

[nfs]
support1.165f.internal openshift_hostname=support1.165f.internal
```

## Run config playbook

```
[root@bastion ~]# ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/byo/config.yml

(....)
PLAY RECAP *******************************************************************************************************************************************************************************************************************************
infranode1.165f.internal   : ok=191  changed=58   unreachable=0    failed=0   
infranode2.165f.internal   : ok=191  changed=58   unreachable=0    failed=0   
loadbalancer1.165f.internal : ok=72   changed=15   unreachable=0    failed=0   
localhost                  : ok=14   changed=0    unreachable=0    failed=0   
master1.165f.internal      : ok=421  changed=150  unreachable=0    failed=0   
master2.165f.internal      : ok=421  changed=150  unreachable=0    failed=0   
master3.165f.internal      : ok=998  changed=393  unreachable=0    failed=0   
node1.165f.internal        : ok=191  changed=58   unreachable=0    failed=0   
node2.165f.internal        : ok=191  changed=58   unreachable=0    failed=0   
node3.165f.internal        : ok=191  changed=58   unreachable=0    failed=0   
support1.165f.internal     : ok=70   changed=14   unreachable=0    failed=0   


INSTALLER STATUS *************************************************************************************************************************************************************************************************************************
Initialization             : Complete
Health Check               : Complete
etcd Install               : Complete
NFS Install                : Complete
Load balancer Install      : Complete
Master Install             : Complete
Master Additional Install  : Complete
Node Install               : Complete
Hosted Install             : Complete
Metrics Install            : Complete
Logging Install            : Complete
Service Catalog Install    : Complete
```

## Enable remote admin user and assign cluster-admin privilige to it

```
root@bastion ~]# ansible masters -m command -a 'htpasswd -b /etc/origin/master/htpasswd admin XXXXX'
master3.165f.internal | SUCCESS | rc=0 >>
Adding password for user admin

master1.165f.internal | SUCCESS | rc=0 >>
Adding password for user admin

master2.165f.internal | SUCCESS | rc=0 >>
Adding password for user admin

[root@bastion ~]#

root@bastion ~]# ssh master1.165f.internal
Last login: Tue Dec 19 10:19:42 2017 from ip-192-199-0-248.eu-central-1.compute.internal
[ec2-user@master1 ~]$ sudo -i
[root@master1 ~]# oc adm policy add-cluster-role-to-user cluster-admin admin
cluster role "cluster-admin" added: "admin"
[root@master1 ~]# logout
[ec2-user@master1 ~]$ logout
Connection to master1.165f.internal closed.
[root@bastion ~]# oc get nodes
NAME                       STATUS                     AGE       VERSION
infranode1.165f.internal   Ready                      22m       v1.7.6+a08f5eeb62
infranode2.165f.internal   Ready                      22m       v1.7.6+a08f5eeb62
master1.165f.internal      Ready,SchedulingDisabled   22m       v1.7.6+a08f5eeb62
master2.165f.internal      Ready,SchedulingDisabled   22m       v1.7.6+a08f5eeb62
master3.165f.internal      Ready,SchedulingDisabled   22m       v1.7.6+a08f5eeb62
node1.165f.internal        Ready                      22m       v1.7.6+a08f5eeb62
node2.165f.internal        Ready                      22m       v1.7.6+a08f5eeb62
node3.165f.internal        Ready                      22m       v1.7.6+a08f5eeb62
[root@bastion ~]#
```

## Check master services on master

### Check that master api services is running on all masters.

```
[root@bastion ~]# ansible masters -m command -a 'systemctl status atomic-openshift-master-api'
master2.165f.internal | SUCCESS | rc=0 >>
● atomic-openshift-master-api.service - Atomic OpenShift Master API
   Loaded: loaded (/usr/lib/systemd/system/atomic-openshift-master-api.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2017-12-20 05:15:35 EST; 5h 9min ago
     Docs: https://github.com/openshift/origin
 Main PID: 11121 (openshift)
   Memory: 242.2M
   CGroup: /system.slice/atomic-openshift-master-api.service
           └─11121 /usr/bin/openshift start master api --config=/etc/origin/master/master-config.yaml --loglevel=2 --listen=https://0.0.0.0:8443 --master=https://master2.165f.internal:8443

Dec 20 10:24:13 master2.165f.internal atomic-openshift-master-api[11121]: I1220 10:24:13.097201   11121 rest.go:362] Starting watch for /apis/security.openshift.io/v1/securitycontextconstraints, rv=59270 labels= fields= timeout=8m26s
Dec 20 10:24:13 master2.165f.internal atomic-openshift-master-api[11121]: E1220 10:24:13.180940   11121 watcher.go:210] watch chan error: etcdserver: mvcc: required revision has been compacted
Dec 20 10:24:13 master2.165f.internal atomic-openshift-master-api[11121]: W1220 10:24:13.181138   11121 reflector.go:343] github.com/openshift/origin/pkg/security/generated/informers/internalversion/factory.go:45: watch of *security.SecurityContextConstraints ended with: etcdserver: mvcc: required revision has been compacted
Dec 20 10:24:14 master2.165f.internal atomic-openshift-master-api[11121]: I1220 10:24:14.184414   11121 rest.go:362] Starting watch for /apis/security.openshift.io/v1/securitycontextconstraints, rv=60821 labels= fields= timeout=5m29s
Dec 20 10:24:22 master2.165f.internal atomic-openshift-master-api[11121]: I1220 10:24:22.643882   11121 rest.go:362] Starting watch for /apis/network.openshift.io/v1/netnamespaces, rv=59466 labels= fields= timeout=7m19s
Dec 20 10:24:37 master2.165f.internal atomic-openshift-master-api[11121]: I1220 10:24:37.639915   11121 rest.go:362] Starting watch for /apis/network.openshift.io/v1/hostsubnets, rv=18198 labels= fields= timeout=7m59s
Dec 20 10:24:47 master2.165f.internal atomic-openshift-master-api[11121]: I1220 10:24:47.516811   11121 rest.go:362] Starting watch for /api/v1/namespaces, rv=59464 labels= fields= timeout=6m49s
Dec 20 10:24:47 master2.165f.internal atomic-openshift-master-api[11121]: E1220 10:24:47.536508   11121 watcher.go:210] watch chan error: etcdserver: mvcc: required revision has been compacted
Dec 20 10:24:47 master2.165f.internal atomic-openshift-master-api[11121]: W1220 10:24:47.536753   11121 reflector.go:343] github.com/openshift/origin/vendor/k8s.io/kubernetes/pkg/client/informers/informers_generated/internalversion/factory.go:72: watch of *api.Namespace ended with: etcdserver: mvcc: required revision has been compacted
Dec 20 10:24:48 master2.165f.internal atomic-openshift-master-api[11121]: I1220 10:24:48.540230   11121 rest.go:362] Starting watch for /api/v1/namespaces, rv=60901 labels= fields= timeout=7m4s

master3.165f.internal | SUCCESS | rc=0 >>
● atomic-openshift-master-api.service - Atomic OpenShift Master API
   Loaded: loaded (/usr/lib/systemd/system/atomic-openshift-master-api.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2017-12-20 05:15:21 EST; 5h 9min ago
     Docs: https://github.com/openshift/origin
 Main PID: 13402 (openshift)
   Memory: 232.1M
   CGroup: /system.slice/atomic-openshift-master-api.service
           └─13402 /usr/bin/openshift start master api --config=/etc/origin/master/master-config.yaml --loglevel=2 --listen=https://0.0.0.0:8443 --master=https://master3.165f.internal:8443

Dec 20 10:22:30 master3.165f.internal atomic-openshift-master-api[13402]: I1220 10:22:30.512048   13402 rest.go:362] Starting watch for /apis/apps/v1beta1/namespaces/helloworld/statefulsets, rv=59877 labels= fields= timeout=1h32m50.395030352s
Dec 20 10:22:31 master3.165f.internal atomic-openshift-master-api[13402]: I1220 10:22:31.520217   13402 rest.go:362] Starting watch for /api/v1/pods, rv=59678 labels= fields=spec.nodeName=node3.165f.internal timeout=9m6s
Dec 20 10:22:53 master3.165f.internal atomic-openshift-master-api[13402]: I1220 10:22:53.479990   13402 rest.go:362] Starting watch for /api/v1/services, rv=59556 labels= fields= timeout=7m27s
Dec 20 10:23:32 master3.165f.internal atomic-openshift-master-api[13402]: I1220 10:23:32.808040   13402 rest.go:362] Starting watch for /apis/apps.openshift.io/v1/deploymentconfigs, rv=59677 labels= fields= timeout=9m28s
Dec 20 10:24:03 master3.165f.internal atomic-openshift-master-api[13402]: I1220 10:24:03.672756   13402 rest.go:362] Starting watch for /apis/rbac.authorization.k8s.io/v1beta1/clusterrolebindings, rv=59651 labels= fields= timeout=7m13s
Dec 20 10:24:12 master3.165f.internal atomic-openshift-master-api[13402]: I1220 10:24:12.934267   13402 rest.go:362] Starting watch for /api/v1/resourcequotas, rv=58497 labels= fields= timeout=9m43s
Dec 20 10:24:13 master3.165f.internal atomic-openshift-master-api[13402]: E1220 10:24:12.960412   13402 watcher.go:210] watch chan error: etcdserver: mvcc: required revision has been compacted
Dec 20 10:24:13 master3.165f.internal atomic-openshift-master-api[13402]: W1220 10:24:12.960737   13402 reflector.go:343] github.com/openshift/origin/vendor/k8s.io/kubernetes/pkg/client/informers/informers_generated/internalversion/factory.go:72: watch of *api.ResourceQuota ended with: etcdserver: mvcc: required revision has been compacted
Dec 20 10:24:13 master3.165f.internal atomic-openshift-master-api[13402]: I1220 10:24:13.963919   13402 rest.go:362] Starting watch for /api/v1/resourcequotas, rv=60820 labels= fields= timeout=9m31s
Dec 20 10:24:49 master3.165f.internal atomic-openshift-master-api[13402]: I1220 10:24:49.788237   13402 rest.go:362] Starting watch for /apis/storage.k8s.io/v1/storageclasses, rv=60081 labels= fields= timeout=5m27s
Dec 20 10:24:51 master3.165f.internal atomic-openshift-master-api[13402]: I1220 10:24:51.953974   13402 rest.go:362] Starting watch for /apis/rbac.authorization.k8s.io/v1beta1/rolebindings, rv=59479 labels= fields= timeout=5m16s
Dec 20 10:24:52 master3.165f.internal atomic-openshift-master-api[13402]: E1220 10:24:52.031986   13402 watcher.go:210] watch chan error: etcdserver: mvcc: required revision has been compacted
Dec 20 10:24:52 master3.165f.internal atomic-openshift-master-api[13402]: W1220 10:24:52.032240   13402 reflector.go:343] github.com/openshift/origin/vendor/k8s.io/kubernetes/pkg/client/informers/informers_generated/internalversion/factory.go:72: watch of *rbac.RoleBinding ended with: etcdserver: mvcc: required revision has been compacted

master1.165f.internal | SUCCESS | rc=0 >>
● atomic-openshift-master-api.service - Atomic OpenShift Master API
   Loaded: loaded (/usr/lib/systemd/system/atomic-openshift-master-api.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2017-12-20 05:15:35 EST; 5h 9min ago
     Docs: https://github.com/openshift/origin
 Main PID: 11567 (openshift)
   Memory: 311.1M
   CGroup: /system.slice/atomic-openshift-master-api.service
           └─11567 /usr/bin/openshift start master api --config=/etc/origin/master/master-config.yaml --loglevel=2 --listen=https://0.0.0.0:8443 --master=https://master1.165f.internal:8443

Dec 20 10:24:25 master1.165f.internal atomic-openshift-master-api[11567]: I1220 10:24:25.000381   11567 rest.go:362] Starting watch for /apis/apps/v1beta1/statefulsets, rv=59699 labels= fields= timeout=7m47s
Dec 20 10:24:26 master1.165f.internal atomic-openshift-master-api[11567]: I1220 10:24:26.009452   11567 rest.go:362] Starting watch for /api/v1/persistentvolumes, rv=60057 labels= fields= timeout=6m16s
Dec 20 10:24:26 master1.165f.internal atomic-openshift-master-api[11567]: I1220 10:24:26.690429   11567 rest.go:362] Starting watch for /apis/extensions/v1beta1/networkpolicies, rv=60035 labels= fields= timeout=9m9s
Dec 20 10:24:30 master1.165f.internal atomic-openshift-master-api[11567]: I1220 10:24:30.593718   11567 rest.go:362] Starting watch for /api/v1/nodes, rv=60855 labels= fields= timeout=6m13s
Dec 20 10:24:31 master1.165f.internal atomic-openshift-master-api[11567]: I1220 10:24:31.571760   11567 rest.go:362] Starting watch for /api/v1/endpoints, rv=59673 labels= fields= timeout=8m22s
Dec 20 10:24:32 master1.165f.internal atomic-openshift-master-api[11567]: I1220 10:24:32.660162   11567 rest.go:362] Starting watch for /apis/batch/v2alpha1/cronjobs, rv=59999 labels= fields= timeout=6m54s
Dec 20 10:24:39 master1.165f.internal atomic-openshift-master-api[11567]: I1220 10:24:39.267831   11567 rest.go:362] Starting watch for /apis/rbac.authorization.k8s.io/v1beta1/roles, rv=59624 labels= fields= timeout=8m44s
Dec 20 10:24:40 master1.165f.internal atomic-openshift-master-api[11567]: I1220 10:24:40.725538   11567 rest.go:362] Starting watch for /api/v1/services, rv=59556 labels= fields= timeout=7m52s
Dec 20 10:24:42 master1.165f.internal atomic-openshift-master-api[11567]: I1220 10:24:42.632631   11567 rest.go:362] Starting watch for /apis/network.openshift.io/v1/netnamespaces, rv=59466 labels= fields= timeout=8m7s
Dec 20 10:24:42 master1.165f.internal atomic-openshift-master-api[11567]: I1220 10:24:42.658470   11567 rest.go:362] Starting watch for /apis/network.openshift.io/v1/netnamespaces, rv=59466 labels= fields= timeout=8m25s
```

### Check that master controller services is running on all masters

```
root@bastion ~]# ansible masters -m command -a 'systemctl status atomic-openshift-master-controllers'
master2.165f.internal | SUCCESS | rc=0 >>
● atomic-openshift-master-controllers.service - Atomic OpenShift Master Controllers
   Loaded: loaded (/usr/lib/systemd/system/atomic-openshift-master-controllers.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2017-12-20 05:15:36 EST; 5h 11min ago
     Docs: https://github.com/openshift/origin
 Main PID: 11227 (openshift)
   Memory: 24.5M
   CGroup: /system.slice/atomic-openshift-master-controllers.service
           └─11227 /usr/bin/openshift start master controllers --config=/etc/origin/master/master-config.yaml --loglevel=2 --listen=https://0.0.0.0:8444

Dec 20 05:15:36 master2.165f.internal systemd[1]: Starting Atomic OpenShift Master Controllers...
Dec 20 05:15:36 master2.165f.internal atomic-openshift-master-controllers[11227]: I1220 05:15:36.424356   11227 plugins.go:77] Registered admission plugin "NamespaceLifecycle"
Dec 20 05:15:36 master2.165f.internal atomic-openshift-master-controllers[11227]: I1220 05:15:36.431088   11227 start_master.go:412] Starting controllers on 0.0.0.0:8444 (v3.7.9)
Dec 20 05:15:36 master2.165f.internal atomic-openshift-master-controllers[11227]: I1220 05:15:36.431111   11227 start_master.go:416] Using images from "openshift3/ose-<component>:v3.7.9"
Dec 20 05:15:36 master2.165f.internal atomic-openshift-master-controllers[11227]: I1220 05:15:36.432667   11227 standalone_apiserver.go:106] Started health checks at 0.0.0.0:8444
Dec 20 05:15:36 master2.165f.internal atomic-openshift-master-controllers[11227]: I1220 05:15:36.434302   11227 plugins.go:77] Registered admission plugin "NamespaceLifecycle"
Dec 20 05:15:36 master2.165f.internal atomic-openshift-master-controllers[11227]: I1220 05:15:36.434772   11227 configgetter.go:53] Initializing cache sizes based on 0MB limit
Dec 20 05:15:36 master2.165f.internal atomic-openshift-master-controllers[11227]: I1220 05:15:36.434888   11227 leaderelection.go:105] Attempting to acquire openshift-master-controllers lease as master-master2.165f.internal-192.199.0.15-wmjkf4rc, renewing every 3s, holding for 15s, and giving up after 10s
Dec 20 05:15:36 master2.165f.internal atomic-openshift-master-controllers[11227]: I1220 05:15:36.434950   11227 leaderelection.go:179] attempting to acquire leader lease...
Dec 20 05:15:36 master2.165f.internal systemd[1]: Started Atomic OpenShift Master Controllers.

master3.165f.internal | SUCCESS | rc=0 >>
● atomic-openshift-master-controllers.service - Atomic OpenShift Master Controllers
   Loaded: loaded (/usr/lib/systemd/system/atomic-openshift-master-controllers.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2017-12-20 05:15:22 EST; 5h 11min ago
     Docs: https://github.com/openshift/origin
 Main PID: 13507 (openshift)
   Memory: 22.3M
   CGroup: /system.slice/atomic-openshift-master-controllers.service
           └─13507 /usr/bin/openshift start master controllers --config=/etc/origin/master/master-config.yaml --loglevel=2 --listen=https://0.0.0.0:8444

Dec 20 05:15:22 master3.165f.internal systemd[1]: Starting Atomic OpenShift Master Controllers...
Dec 20 05:15:22 master3.165f.internal atomic-openshift-master-controllers[13507]: I1220 05:15:22.937406   13507 plugins.go:77] Registered admission plugin "NamespaceLifecycle"
Dec 20 05:15:22 master3.165f.internal atomic-openshift-master-controllers[13507]: I1220 05:15:22.949135   13507 start_master.go:412] Starting controllers on 0.0.0.0:8444 (v3.7.9)
Dec 20 05:15:22 master3.165f.internal atomic-openshift-master-controllers[13507]: I1220 05:15:22.949160   13507 start_master.go:416] Using images from "openshift3/ose-<component>:v3.7.9"
Dec 20 05:15:22 master3.165f.internal atomic-openshift-master-controllers[13507]: I1220 05:15:22.951230   13507 standalone_apiserver.go:106] Started health checks at 0.0.0.0:8444
Dec 20 05:15:22 master3.165f.internal atomic-openshift-master-controllers[13507]: I1220 05:15:22.957708   13507 plugins.go:77] Registered admission plugin "NamespaceLifecycle"
Dec 20 05:15:22 master3.165f.internal atomic-openshift-master-controllers[13507]: I1220 05:15:22.958352   13507 configgetter.go:53] Initializing cache sizes based on 0MB limit
Dec 20 05:15:22 master3.165f.internal atomic-openshift-master-controllers[13507]: I1220 05:15:22.958571   13507 leaderelection.go:105] Attempting to acquire openshift-master-controllers lease as master-master3.165f.internal-192.199.0.63-7p975cjh, renewing every 3s, holding for 15s, and giving up after 10s
Dec 20 05:15:22 master3.165f.internal atomic-openshift-master-controllers[13507]: I1220 05:15:22.958606   13507 leaderelection.go:179] attempting to acquire leader lease...
Dec 20 05:15:22 master3.165f.internal systemd[1]: Started Atomic OpenShift Master Controllers.

master1.165f.internal | SUCCESS | rc=0 >>
● atomic-openshift-master-controllers.service - Atomic OpenShift Master Controllers
   Loaded: loaded (/usr/lib/systemd/system/atomic-openshift-master-controllers.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2017-12-20 05:15:36 EST; 5h 11min ago
     Docs: https://github.com/openshift/origin
 Main PID: 11674 (openshift)
   Memory: 111.2M
   CGroup: /system.slice/atomic-openshift-master-controllers.service
           └─11674 /usr/bin/openshift start master controllers --config=/etc/origin/master/master-config.yaml --loglevel=2 --listen=https://0.0.0.0:8444

Dec 20 10:23:56 master1.165f.internal atomic-openshift-master-controllers[11674]: W1220 10:23:56.230313   11674 reflector.go:343] github.com/openshift/origin/pkg/image/generated/informers/internalversion/factory.go:45: watch of *image.ImageStream ended with: etcdserver: mvcc: required revision has been compacted
Dec 20 10:24:07 master1.165f.internal atomic-openshift-master-controllers[11674]: W1220 10:24:07.769303   11674 reflector.go:343] github.com/openshift/origin/vendor/k8s.io/kubernetes/pkg/client/informers/informers_generated/externalversions/factory.go:72: watch of *v1beta1.ReplicaSet ended with: etcdserver: mvcc: required revision has been compacted
Dec 20 10:25:22 master1.165f.internal atomic-openshift-master-controllers[11674]: W1220 10:25:22.115760   11674 reflector.go:343] github.com/openshift/origin/vendor/k8s.io/kubernetes/pkg/client/informers/informers_generated/externalversions/factory.go:72: watch of *v1.ServiceAccount ended with: etcdserver: mvcc: required revision has been compacted
Dec 20 10:25:35 master1.165f.internal atomic-openshift-master-controllers[11674]: W1220 10:25:35.803976   11674 reflector.go:343] github.com/openshift/origin/pkg/apps/generated/informers/internalversion/factory.go:45: watch of *apps.DeploymentConfig ended with: etcdserver: mvcc: required revision has been compacted
Dec 20 10:25:40 master1.165f.internal atomic-openshift-master-controllers[11674]: W1220 10:25:40.037827   11674 reflector.go:343] github.com/openshift/origin/vendor/k8s.io/kubernetes/pkg/client/informers/informers_generated/externalversions/factory.go:72: watch of *v1.ResourceQuota ended with: etcdserver: mvcc: required revision has been compacted
Dec 20 10:25:48 master1.165f.internal atomic-openshift-master-controllers[11674]: W1220 10:25:48.864544   11674 reflector.go:343] github.com/openshift/origin/vendor/k8s.io/kubernetes/pkg/client/informers/informers_generated/internalversion/factory.go:72: watch of *rbac.ClusterRoleBinding ended with: etcdserver: mvcc: required revision has been compacted
Dec 20 10:25:52 master1.165f.internal atomic-openshift-master-controllers[11674]: W1220 10:25:52.672888   11674 reflector.go:343] github.com/openshift/origin/vendor/k8s.io/kubernetes/pkg/controller/garbagecollector/graph_builder.go:231: watch of <nil> ended with: etcdserver: mvcc: required revision has been compacted
Dec 20 10:25:53 master1.165f.internal atomic-openshift-master-controllers[11674]: W1220 10:25:53.073922   11674 reflector.go:343] github.com/openshift/origin/vendor/k8s.io/kubernetes/pkg/client/informers/informers_generated/externalversions/factory.go:72: watch of *v1.PodTemplate ended with: etcdserver: mvcc: required revision has been compacted
Dec 20 10:25:55 master1.165f.internal atomic-openshift-master-controllers[11674]: W1220 10:25:55.267260   11674 reflector.go:343] github.com/openshift/origin/vendor/k8s.io/kubernetes/pkg/client/informers/informers_generated/externalversions/factory.go:72: watch of *v1beta1.CertificateSigningRequest ended with: etcdserver: mvcc: required revision has been compacted
Dec 20 10:26:33 master1.165f.internal atomic-openshift-master-controllers[11674]: W1220 10:26:33.750881   11674 reflector.go:343] github.com/openshift/origin/pkg/build/generated/informers/internalversion/factory.go:45: watch of *build.BuildConfig ended with: etcdserver: mvcc: required revision has been compacted
```

## Check etcd on masters

```
[root@bastion ~]# ansible masters -m command -a 'systemctl status etcd'
master2.165f.internal | SUCCESS | rc=0 >>
● etcd.service - Etcd Server
   Loaded: loaded (/usr/lib/systemd/system/etcd.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2017-12-20 04:33:06 EST; 5h 49min ago
 Main PID: 13820 (etcd)
   Memory: 289.8M
   CGroup: /system.slice/etcd.service
           └─13820 /usr/bin/etcd --name=master2.165f.internal --data-dir=/var/lib/etcd/ --listen-client-urls=https://192.199.0.15:2379

Dec 20 10:00:19 master2.165f.internal etcd[13820]: apply entries took too long [509.616755ms for 1 entries]
Dec 20 10:00:19 master2.165f.internal etcd[13820]: avoid queries with large range/delete range!
Dec 20 10:05:18 master2.165f.internal etcd[13820]: store.index: compact 57480
Dec 20 10:05:18 master2.165f.internal etcd[13820]: finished scheduled compaction at 57480 (took 1.351108ms)
Dec 20 10:10:18 master2.165f.internal etcd[13820]: store.index: compact 58167
Dec 20 10:10:18 master2.165f.internal etcd[13820]: finished scheduled compaction at 58167 (took 1.242125ms)
Dec 20 10:15:18 master2.165f.internal etcd[13820]: store.index: compact 58847
Dec 20 10:15:18 master2.165f.internal etcd[13820]: finished scheduled compaction at 58847 (took 1.19336ms)
Dec 20 10:20:18 master2.165f.internal etcd[13820]: store.index: compact 59580
Dec 20 10:20:18 master2.165f.internal etcd[13820]: finished scheduled compaction at 59580 (took 1.285919ms)

master1.165f.internal | SUCCESS | rc=0 >>
● etcd.service - Etcd Server
   Loaded: loaded (/usr/lib/systemd/system/etcd.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2017-12-20 04:33:06 EST; 5h 49min ago
 Main PID: 14343 (etcd)
   Memory: 315.8M
   CGroup: /system.slice/etcd.service
           └─14343 /usr/bin/etcd --name=master1.165f.internal --data-dir=/var/lib/etcd/ --listen-client-urls=https://192.199.0.81:2379

Dec 20 10:05:18 master1.165f.internal etcd[14343]: store.index: compact 57480
Dec 20 10:05:18 master1.165f.internal etcd[14343]: finished scheduled compaction at 57480 (took 1.159163ms)
Dec 20 10:10:18 master1.165f.internal etcd[14343]: store.index: compact 58167
Dec 20 10:10:18 master1.165f.internal etcd[14343]: finished scheduled compaction at 58167 (took 1.262211ms)
Dec 20 10:13:44 master1.165f.internal etcd[14343]: apply entries took too long [200.552388ms for 1 entries]
Dec 20 10:13:44 master1.165f.internal etcd[14343]: avoid queries with large range/delete range!
Dec 20 10:15:18 master1.165f.internal etcd[14343]: store.index: compact 58847
Dec 20 10:15:18 master1.165f.internal etcd[14343]: finished scheduled compaction at 58847 (took 1.284945ms)
Dec 20 10:20:18 master1.165f.internal etcd[14343]: store.index: compact 59580
Dec 20 10:20:18 master1.165f.internal etcd[14343]: finished scheduled compaction at 59580 (took 1.263974ms)

master3.165f.internal | SUCCESS | rc=0 >>
● etcd.service - Etcd Server
   Loaded: loaded (/usr/lib/systemd/system/etcd.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2017-12-20 04:33:06 EST; 5h 49min ago
 Main PID: 21017 (etcd)
   Memory: 272.5M
   CGroup: /system.slice/etcd.service
           └─21017 /usr/bin/etcd --name=master3.165f.internal --data-dir=/var/lib/etcd/ --listen-client-urls=https://192.199.0.63:2379

Dec 20 10:16:34 master3.165f.internal etcd[21017]: sync duration of 1.139394789s, expected less than 1s
Dec 20 10:16:45 master3.165f.internal etcd[21017]: apply entries took too long [414.609505ms for 4 entries]
Dec 20 10:16:45 master3.165f.internal etcd[21017]: avoid queries with large range/delete range!
Dec 20 10:16:56 master3.165f.internal etcd[21017]: apply entries took too long [312.377341ms for 1 entries]
Dec 20 10:16:56 master3.165f.internal etcd[21017]: avoid queries with large range/delete range!
Dec 20 10:17:45 master3.165f.internal etcd[21017]: apply entries took too long [491.422488ms for 3 entries]
Dec 20 10:17:45 master3.165f.internal etcd[21017]: avoid queries with large range/delete range!
Dec 20 10:20:18 master3.165f.internal etcd[21017]: store.index: compact 59580
Dec 20 10:20:18 master3.165f.internal etcd[21017]: finished scheduled compaction at 59580 (took 1.315112ms)
Dec 20 10:21:35 master3.165f.internal etcd[21017]: sync duration of 1.372885601s, expected less than 1s
```

## Check loadbalancer

```
[root@loadbalancer1 ~]# echo "show stat" | nc -U /var/lib/haproxy/stats | cut -d "," -f 1,2,50,37 | column -s, -t
# pxname              svname    check_status  cli_abrt
stats                 FRONTEND                
stats                 BACKEND                 0
atomic-openshift-api  FRONTEND                
atomic-openshift-api  master0   L4OK          4
atomic-openshift-api  master1   L4OK          0
atomic-openshift-api  master2   L4OK          0
atomic-openshift-api  BACKEND                 4
[root@loadbalancer1 ~]#

kmbpro|~curl -v   https://loadbalancer.165f.example.opentlc.com:8443/healthz --insecure
*   Trying 18.195.69.155...
* TCP_NODELAY set
* Connected to loadbalancer.165f.example.opentlc.com (18.195.69.155) port 8443 (#0)
* WARNING: disabling hostname validation also disables SNI.
* TLS 1.2 connection using TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
* Server certificate: 172.30.0.1
* Server certificate: openshift-signer@1513762505
> GET /healthz HTTP/1.1
> Host: loadbalancer.165f.example.opentlc.com:8443
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Cache-Control: no-store
< Date: Thu, 21 Dec 2017 09:28:10 GMT
< Content-Length: 2
< Content-Type: text/plain; charset=utf-8
<
* Connection #0 to host loadbalancer.165f.example.opentlc.com left intact
okmbpro|~$
```

## Check DNS wildcard for infranodes

```
mbpro|~$ host foobar.apps.165f.example.opentlc.com
foobar.apps.165f.example.opentlc.com has address 18.194.51.78
foobar.apps.165f.example.opentlc.com has address 18.195.234.0

mbpro|~$ host barfoo.apps.165f.example.opentlc.com
barfoo.apps.165f.example.opentlc.com has address 18.195.234.0
barfoo.apps.165f.example.opentlc.com has address 18.194.51.78
mbpro|~$
```

## Check infranodes

```
[root@bastion ~]# oc get nodes -l env=infra
NAME                       STATUS    AGE       VERSION
infranode1.165f.internal   Ready     23h       v1.7.6+a08f5eeb62
infranode2.165f.internal   Ready     23h       v1.7.6+a08f5eeb62
[root@bastion ~]#

[root@bastion ~]# oc adm manage-node infranode1.165f.internal --list-pods

Listing matched pods on node: infranode1.165f.internal

NAME                         READY     STATUS    RESTARTS   AGE
docker-registry-1-qsjgc      1/1       Running   1          22h
router-1-bpzph               1/1       Running   1          22h
logging-curator-1-xjmr7      1/1       Running   1          22h
logging-fluentd-fg6mj        1/1       Running   1          22h
logging-kibana-1-26bg8       2/2       Running   2          22h
hawkular-cassandra-1-tktjb   1/1       Running   1          22h
hawkular-metrics-lkgjn       1/1       Running   6          22h
[root@bastion ~]# oc adm manage-node infranode2.165f.internal --list-pods

Listing matched pods on node: infranode2.165f.internal

NAME                                      READY     STATUS    RESTARTS   AGE
docker-registry-1-z7vkc                   1/1       Running   1          22h
router-1-mt7sl                            1/1       Running   1          22h
logging-es-data-master-3t7qza5i-1-8tcx4   2/2       Running   2          22h
logging-fluentd-vvcsc                     1/1       Running   1          22h
heapster-p6gjn                            1/1       Running   5          22h
[root@bastion ~]#
```

## Troubleshooting metrics

Metrics-cassandra pod was hanging on volume mount, stuck in pending.

Found that the allocated pv was mapped to the wrong path (default /exports/metrics) rather than what the variable "openshift_metrics_storage_nfs_directory/openshift_metrics_storage_volume_name" says in the inventory.

Worked around by:

```
[root@bastion ~]# oc describe pv metrics-volume -n openshift-infra
Name:        metrics-volume
Labels:        storage=metrics
Annotations:    pv.kubernetes.io/bound-by-controller=yes
StorageClass:    
Status:        Bound
Claim:        openshift-infra/metrics-1
Reclaim Policy:    Retain
Access Modes:    RWO
Capacity:    10Gi
Message:    
Source:
    Type:    NFS (an NFS mount that lasts the lifetime of a pod)
    Server:    support1.165f.internal
    Path:    /exports/metrics
    ReadOnly:    false
Events:        <none>

[root@bastion ~]# oc edit pv metrics-volume -n openshift-infra
persistentvolume "metrics-volume" edited

[root@bastion ~]# oc describe pv metrics-volume -n openshift-infra
Name:        metrics-volume
Labels:        storage=metrics
Annotations:    pv.kubernetes.io/bound-by-controller=yes
StorageClass:    
Status:        Bound
Claim:        openshift-infra/metrics-1
Reclaim Policy:    Retain
Access Modes:    RWO
Capacity:    10Gi
Message:    
Source:
    Type:    NFS (an NFS mount that lasts the lifetime of a pod)
    Server:    support1.165f.internal
    Path:    /srv/nfs/metrics
    ReadOnly:    false
Events:        <none>
[root@bastion ~]#
```

After deleting the stuck cassandra pod, the new cassandra pod started in a second, volume was mounted and metrics work.

```
Pod status
[root@bastion ~]# oc get pods --all-namespaces
NAMESPACE         NAME                                      READY     STATUS    RESTARTS   AGE
default           docker-registry-1-qsjgc                   1/1       Running   0          3h
default           docker-registry-1-z7vkc                   1/1       Running   0          3h
default           registry-console-1-458g4                  1/1       Running   0          3h
default           router-1-bpzph                            1/1       Running   0          3h
default           router-1-mt7sl                            1/1       Running   0          3h
logging           logging-curator-1-xjmr7                   1/1       Running   0          3h
logging           logging-es-data-master-3t7qza5i-1-8tcx4   2/2       Running   0          3h
logging           logging-fluentd-bbtb2                     1/1       Running   0          3h
logging           logging-fluentd-ff8rp                     1/1       Running   0          3h
logging           logging-fluentd-fg6mj                     1/1       Running   0          3h
logging           logging-fluentd-hswfl                     1/1       Running   0          3h
logging           logging-fluentd-nlndc                     1/1       Running   0          3h
logging           logging-fluentd-nwxfk                     1/1       Running   0          3h
logging           logging-fluentd-vvcsc                     1/1       Running   0          3h
logging           logging-fluentd-z6j5j                     1/1       Running   0          3h
logging           logging-kibana-1-26bg8                    2/2       Running   0          3h
openshift-infra   hawkular-cassandra-1-tktjb                1/1       Running   0          2h
openshift-infra   hawkular-metrics-lkgjn                    1/1       Running   5          3h
openshift-infra   heapster-p6gjn                            1/1       Running   4          3h
[root@bastion ~]#
```

# Basic requirements

## Authenticate admin with htpasswd and add privileges

```
[root@bastion ~]# ansible masters -m command -a 'htpasswd -b /etc/origin/master/htpasswd admin #usualsuspect#'
(...)

[root@master1 ~]# oc adm policy add-cluster-role-to-user cluster-admin admin
cluster role "cluster-admin" added: "admin"
[root@master1 ~]#
[root@bastion ~]# oc login https://loadbalancer1.165f.internal:8443 --username=admin --password=#usualsuspect#
Login successful.

You have access to the following projects and can switch between them with 'oc project <projectname>':
(...)

[root@bastion ~]# oc get nodes
NAME                       STATUS                     AGE       VERSION
infranode1.165f.internal   Ready                      23h       v1.7.6+a08f5eeb62
infranode2.165f.internal   Ready                      23h       v1.7.6+a08f5eeb62
master1.165f.internal      Ready,SchedulingDisabled   23h       v1.7.6+a08f5eeb62
master2.165f.internal      Ready,SchedulingDisabled   23h       v1.7.6+a08f5eeb62
master3.165f.internal      Ready,SchedulingDisabled   23h       v1.7.6+a08f5eeb62
node1.165f.internal        Ready                      23h       v1.7.6+a08f5eeb62
node2.165f.internal        Ready                      23h       v1.7.6+a08f5eeb62
node3.165f.internal        Ready                      23h       v1.7.6+a08f5eeb62
[root@bastion ~]#
```

## Registry storage

```
[root@bastion ~]# oc get pv -n default |grep registry
registry-volume     10Gi       RWX           Retain          Bound       default/registry-claim                               23h
[root@bastion ~]# oc get pvc -n default
NAME             STATUS    VOLUME            CAPACITY   ACCESSMODES   STORAGECLASS   AGE
registry-claim   Bound     registry-volume   10Gi       RWX                          23h
[root@bastion ~]#

[root@bastion ~]# oc describe dc/docker-registry -n default
Name:        docker-registry
Namespace:    default
Created:    23 hours ago
Labels:        docker-registry=default
Annotations:    <none>
Latest Version:    1
Selector:    docker-registry=default
Replicas:    2
Triggers:    Config
Strategy:    Rolling
Template:
Pod Template:
  Labels:        docker-registry=default
  Service Account:    registry
  Containers:
   registry:
    Image:    openshift3/ose-docker-registry:v3.7.9
    Port:    5000/TCP
    Requests:
      cpu:    100m
      memory:    256Mi
    Liveness:    http-get https://:5000/healthz delay=10s timeout=5s period=10s #success=1 #failure=3
    Readiness:    http-get https://:5000/healthz delay=0s timeout=5s period=10s #success=1 #failure=3
    Environment:
      REGISTRY_HTTP_ADDR:                    :5000
      REGISTRY_HTTP_NET:                    tcp
      REGISTRY_HTTP_SECRET:                    JkIUIqW7AtcWGF6gnynolZc/zw+jPVHESZeOIbpXPyA=
      REGISTRY_MIDDLEWARE_REPOSITORY_OPENSHIFT_ENFORCEQUOTA:    false
      OPENSHIFT_DEFAULT_REGISTRY:                docker-registry.default.svc:5000
      REGISTRY_HTTP_TLS_KEY:                    /etc/secrets/registry.key
      REGISTRY_HTTP_TLS_CERTIFICATE:                /etc/secrets/registry.crt
    Mounts:
      /etc/secrets from registry-certificates (rw)
      /registry from registry-storage (rw)
  Volumes:
   registry-storage:
    Type:    PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:    registry-claim
    ReadOnly:    false
   registry-certificates:
    Type:    Secret (a volume populated by a Secret)
    SecretName:    registry-certificates
    Optional:    false

Deployment #1 (latest):
    Name:        docker-registry-1
    Created:    23 hours ago
    Status:        Complete
    Replicas:    2 current / 2 desired
    Selector:    deployment=docker-registry-1,deploymentconfig=docker-registry,docker-registry=default
    Labels:        docker-registry=default,openshift.io/deployment-config.name=docker-registry
    Pods Status:    2 Running / 0 Waiting / 0 Succeeded / 0 Failed

Events:    <none>
[root@bastion ~]#

[root@bastion ~]# oc get po -n default|grep registry
docker-registry-1-qsjgc    1/1       Running   1          23h
docker-registry-1-z7vkc    1/1       Running   1          23h
registry-console-1-458g4   1/1       Running   1          23h
[root@bastion ~]#
```

## Router on infranodes

```
[root@bastion ~]# oc get nodes -l env=infra
NAME                       STATUS    AGE       VERSION
infranode1.165f.internal   Ready     23h       v1.7.6+a08f5eeb62
infranode2.165f.internal   Ready     23h       v1.7.6+a08f5eeb62
[root@bastion ~]#

[root@bastion ~]# oc adm manage-node infranode1.165f.internal --list-pods

Listing matched pods on node: infranode1.165f.internal

NAME                         READY     STATUS    RESTARTS   AGE
docker-registry-1-qsjgc      1/1       Running   1          22h
router-1-bpzph               1/1       Running   1          22h
logging-curator-1-xjmr7      1/1       Running   1          22h
logging-fluentd-fg6mj        1/1       Running   1          22h
logging-kibana-1-26bg8       2/2       Running   2          22h
hawkular-cassandra-1-tktjb   1/1       Running   1          22h
hawkular-metrics-lkgjn       1/1       Running   6          22h
[root@bastion ~]# oc adm manage-node infranode2.165f.internal --list-pods

Listing matched pods on node: infranode2.165f.internal

NAME                                      READY     STATUS    RESTARTS   AGE
docker-registry-1-z7vkc                   1/1       Running   1          22h
router-1-mt7sl                            1/1       Running   1          22h
logging-es-data-master-3t7qza5i-1-8tcx4   2/2       Running   2          22h
logging-fluentd-vvcsc                     1/1       Running   1          22h
heapster-p6gjn                            1/1       Running   5          22h
[root@bastion ~]#
```

## Add PVs for apps to consume

```
root@bastion ~]# ansible-playbook nfs.yml

[root@bastion ~]# oc get pv
NAME                CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS      CLAIM                       STORAGECLASS   REASON    AGE
logging-es-volume   10Gi       RWO           Retain          Bound       logging/logging-es-0                                 5h
metrics-volume      10Gi       RWO           Retain          Bound       openshift-infra/metrics-1                            5h
pv01-nfs            1Gi        RWO,RWX       Recycle         Available                                                        32s
pv02-nfs            1Gi        RWO,RWX       Recycle         Available                                                        32s
pv03-nfs            1Gi        RWO,RWX       Recycle         Available                                                        32s
pv04-nfs            1Gi        RWO,RWX       Recycle         Available                                                        32s
pv05-nfs            5Gi        RWO,RWX       Recycle         Available                                                        32s
pv06-nfs            15Gi       RWO,RWX       Recycle         Available                                                        32s
pv07-nfs            15Gi       RWO,RWX       Recycle         Available                                                        32s
pv08-nfs            15Gi       RWO,RWX       Recycle         Available                                                        31s
pv09-nfs            15Gi       RWO,RWX       Recycle         Available                                                        31s
pv10-nfs            15Gi       RWO,RWX       Recycle         Available                                                        31s
pv11-nfs            15Gi       RWO,RWX       Recycle         Available                                                        31s
pv12-nfs            15Gi       RWO,RWX       Recycle         Available                                                        31s
pv13-nfs            15Gi       RWO,RWX       Recycle         Available                                                        31s
pv14-nfs            15Gi       RWO,RWX       Recycle         Available                                                        31s
pv15-nfs            15Gi       RWO,RWX       Recycle         Available                                                        31s
registry-volume     10Gi       RWX           Retain          Bound       default/registry-claim                               5h
```

# Environment config

## Multitenancy and multiple devops users in separate projects

Creating two devops users: [root@bastion ~]# ansible masters -m command -a 'htpasswd -b /etc/origin/master/htpasswd devops1 #usualsuspect#' master3.165f.internal | SUCCESS | rc=0 >> Adding password for user devops1

```
master2.165f.internal | SUCCESS | rc=0 >>
Adding password for user devops1

master1.165f.internal | SUCCESS | rc=0 >>
Adding password for user devops1

[root@bastion ~]#
[root@bastion ~]# ansible masters -m command -a 'htpasswd -b /etc/origin/master/htpasswd devops2 #usualsuspect#'
master1.165f.internal | SUCCESS | rc=0 >>
Updating password for user devops2

master2.165f.internal | SUCCESS | rc=0 >>
Updating password for user devops2

master3.165f.internal | SUCCESS | rc=0 >>
Updating password for user devops2

[root@bastion ~]#
```

## Using oc as devops users from laptop

### Troubleshooting

```
mbpro|~$ oc login https://loadbalancer.165f.example.opentlc.com:8443 --insecure-skip-tls-verify=true
Server [https://localhost:8443]: https://loadbalancer.165f.example.opentlc.com:8443
error: net/http: TLS handshake timeout
```

Seems to be related to <https://github.com/openshift/origin/issues/12295>

Tried to config with token:

```
oc config set-credentials cluster-admin  --token='0BFU5C473D0BFU5C473D0BFU5C473D0BFU5C473D0BFU5C473D'
oc config set-cluster lb --server=https://loadbalancer.165f.example.opentlc.com:8443
oc config set-context admin --cluster=lb --user=cluster-admin
oc config use-context admin
edit ~/.kube/config and add     
  insecure-skip-tls-verify: true
for cluster
```

Voila!

```
mbpro|~$ oc get node
NAME                       STATUS                     AGE       VERSION
infranode1.165f.internal   Ready                      4h        v1.7.6+a08f5eeb62
infranode2.165f.internal   Ready                      4h        v1.7.6+a08f5eeb62
master1.165f.internal      Ready,SchedulingDisabled   4h        v1.7.6+a08f5eeb62
master2.165f.internal      Ready,SchedulingDisabled   4h        v1.7.6+a08f5eeb62
master3.165f.internal      Ready,SchedulingDisabled   4h        v1.7.6+a08f5eeb62
node1.165f.internal        Ready                      4h        v1.7.6+a08f5eeb62
node2.165f.internal        Ready                      4h        v1.7.6+a08f5eeb62
node3.165f.internal        Ready                      4h        v1.7.6+a08f5eeb62
mbpro|~$
```

Set context for user devops1

```
mbpro|~$ oc config set-credentials devops1  --token='0BFU5C473D0BFU5C473D0BFU5C473D0BFU5C473D0BFU5C473D'
User "devops1" set.
mbpro|~$ oc config set-context developer --cluster=lb --user=devops1
Context "developer" created.
mbpro|~$ oc config use-context developer
Switched to context "developer".
mbpro|~$ oc get project
No resources found.
mbpro|~$
```

### Deploy an app as devops1

```
mbpro|~$ oc new-project helloworld
Now using project "helloworld" on server "https://loadbalancer.165f.example.opentlc.com:8443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app centos/ruby-22-centos7~https://github.com/openshift/ruby-ex.git

to build a new example application in Ruby.

mbpro|~$ oc new-app openshift/hello-openshift:v1.1.1.1
--> Found Docker image fa30db8 (23 months old) from Docker Hub for "openshift/hello-openshift:v1.1.1.1"

    * An image stream will be created as "hello-openshift:v1.1.1.1" that will track this image
    * This image will be deployed in deployment config "hello-openshift"
    * Ports 8080/tcp, 8888/tcp will be load balanced by service "hello-openshift"
      * Other containers can access this service through the hostname "hello-openshift"
    * WARNING: Image "openshift/hello-openshift:v1.1.1.1" runs as the 'root' user which may not be permitted by your cluster administrator

--> Creating resources ...
    imagestream "hello-openshift" created
    deploymentconfig "hello-openshift" created
    service "hello-openshift" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/hello-openshift'
    Run 'oc status' to view your app.
mbpro|~$ oc get pod
NAME                       READY     STATUS              RESTARTS   AGE
hello-openshift-1-deploy   0/1       ContainerCreating   0          6s

mbpro|~$ oc get dc
NAME              REVISION   DESIRED   CURRENT   TRIGGERED BY
hello-openshift   1          1         0         config,image(hello-openshift:v1.1.1.1)

mbpro|~$ oc get pod
NAME                       READY     STATUS              RESTARTS   AGE
hello-openshift-1-deploy   1/1       Running             0          26s
hello-openshift-1-qpzq4    0/1       ContainerCreating   0          1s

mbpro|~$ oc get svc
NAME              CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
hello-openshift   172.30.38.172   <none>        8080/TCP,8888/TCP   36s
mbpro|~$ oc get pod
NAME                      READY     STATUS    RESTARTS   AGE
hello-openshift-1-qpzq4   1/1       Running   0          4m
mbpro|~$
```

No route was created

```
mbpro|~$ oc get route
No resources found.
mbpro|~$
```

So expose it

```
mbpro|~$ oc expose service hello-openshift
route "hello-openshift" exposed
```

Test it

```
mbpro|~$ curl http://hello-openshift-helloworld.apps.165f.example.opentlc.com/
Hello OpenShift!
mbpro|~$
```

### Deploy an app as devops2

```
mbpro|~$ oc whoami
devops2
mbpro|~$ oc get project
No resources found.
mbpro|~$ oc  new-project helloworld2
Now using project "helloworld2" on server "https://loadbalancer.165f.example.opentlc.com:8443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app centos/ruby-22-centos7~https://github.com/openshift/ruby-ex.git

to build a new example application in Ruby.

mbpro|~$
mbpro|~$ oc get pod
NAME                      READY     STATUS    RESTARTS   AGE
hello-openshift-1-7rkwl   1/1       Running   0          3m

mbpro|~$ oc get svc
NAME              CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
hello-openshift   172.30.236.85   <none>        8080/TCP,8888/TCP   3m
mbpro|~$ oc logs hello-openshift-1-7rkwl
serving on 8080
serving on 8888

mbpro|~$ oc expose hello-openshift
error: resource(s) were provided, but no name, label selector, or --all flag specified
See 'oc expose -h' for help and examples.

mbpro|~$ oc expose svc hello-openshift
route "hello-openshift" exposed
mbpro|~$
mbpro|~$ oc get route  
NAME              HOST/PORT                                                   PATH      SERVICES          PORT       TERMINATION   WILDCARD
hello-openshift   hello-openshift-helloworld2.apps.165f.example.opentlc.com             hello-openshift   8080-tcp                 None

mbpro|~$ curl hello-openshift-helloworld2.apps.165f.example.opentlc.com
Hello OpenShift!
mbpro|~$
```

## Verify network multitenancy

Separate network namespaces for the two projects created.

```
mbpro|~$ oc whoami
admin
mbpro|~$
mbpro|~$ oc get netnamespace
NAME               NETID      EGRESS IPS
default            0          []
helloworld         4803043    []
helloworld2        16235517   []
kube-public        10530874   []
kube-system        15546326   []
logging            15722671   []
management-infra   14541433   []
openshift          8436126    []
openshift-infra    15843186   []
openshift-node     5742431    []
mbpro|~$
```

Deploy a container/dc for testing access from project helloworld2.

```
mbpro|~$ oc whoami
devops2
mbpro|~$ oc get project
NAME          DISPLAY NAME   STATUS
helloworld2                  Active
mbpro|~$
mbpro|~$ oc run ngx2 --image=registry.access.redhat.com/rhscl/nginx-110-rhel7 -- sleep 999999
deploymentconfig "ngx2" created

mbpro|~$ oc get pod
NAME                      READY     STATUS    RESTARTS   AGE
hello-openshift-1-7rkwl   1/1       Running   0          29m
ngx2-1-mnkqb              1/1       Running   0          42s

mbpro|~$ oc rsh ngx2-1-mnkqb
sh-4.2$

sh-4.2$ host hello-openshift
hello-openshift.helloworld2.svc.cluster.local has address 172.30.236.85
```

Access to service in same project

```
sh-4.2$ curl hello-openshift.helloworld2.svc.cluster.local:8080
Hello OpenShift!
```

Access to service in project helloworld; no access as expected.

```
sh-4.2$ curl hello-openshift.helloworld.svc.cluster.local:8080
^C
sh-4.2$
```

And vice versa: mbpro|~$ oc whoami devops1 mbpro|~$ oc get project NAME DISPLAY NAME STATUS helloworld Active mbpro|~$

```
mbpro|~$ oc run ngx1 --image=registry.access.redhat.com/rhscl/nginx-110-rhel7 -- sleep 999999
deploymentconfig "ngx1" created

mbpro|~$ oc get pod
NAME                      READY     STATUS    RESTARTS   AGE
hello-openshift-1-qpzq4   1/1       Running   1          23h
ngx1-1-deploy             1/1       Running   0          13s
ngx1-1-mnn9p              1/1       Running   0          10s

mbpro|~$ oc rsh ngx1-1-mnn9p
sh-4.2$ host hello-openshift
hello-openshift.helloworld.svc.cluster.local has address 172.30.38.172

sh-4.2$ curl hello-openshift.helloworld.svc.cluster.local:8080
Hello OpenShift!

sh-4.2$ host hello-openshift.helloworld2
hello-openshift.helloworld2.svc.cluster.local has address 172.30.236.85

sh-4.2$ curl hello-openshift.helloworld2.svc.cluster.local
^C
sh-4.2$
```

## Verify metrics

```
mbpro|~$ curl -s --header 'Hawkular-Tenant: helloworld' --header 'Authorization: Bearer 0BFU5C473D0BFU5C473D0BFU5C473D0BFU5C473D0BFU5C473D'   --insecure https://hawkular-metrics.apps.165f.example.opentlc.com/hawkular/metrics/counters|jq '.|length'
27
mbpro|~$
mbpro|~$ curl -s --header 'Hawkular-Tenant: helloworld' --header 'Authorization: Bearer 0BFU5C473D0BFU5C473D0BFU5C473D0BFU5C473D0BFU5C473D'   --insecure https://hawkular-metrics.apps.165f.example.opentlc.com/hawkular/metrics/metrics|jq '.|length'
106
mbpro|~$
```

Metrics also work inside OCP as admin and each of the tenants.

Also, there is data stored on the nfs server serving those volumes.

```
[root@support1 ~]# find /srv/nfs/metrics/|wc -l
840
[root@support1 ~]#
```

## Logging

Logging works for admin and tenants. Redirect to kibana with transparant oauth access, and logs are available in Kibana.

Also, there is data stored on the nfs server serving those volumes.

```
[root@support1 ~]# find /srv/nfs/logging-es/|wc -l
829
[root@support1 ~]#
```

# CICD workflow

## Create persistent Jenkins instance for user devops1

```
[jbjorgee-redhat.com@bastion src]$ oc whoami
devops1
[jbjorgee-redhat.com@bastion src]$ oc new-project cicd
Now using project "cicd" on server "https://loadbalancer1.165f.internal:8443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app centos/ruby-22-centos7~https://github.com/openshift/ruby-ex.git

to build a new example application in Ruby.

[jbjorgee-redhat.com@bastion src]$
[jbjorgee-redhat.com@bastion src]$ oc new-app jenkins-persistent
--> Deploying template "openshift/jenkins-persistent" to project cicd

     Jenkins (Persistent)
     ---------
     Jenkins service, with persistent storage.

     NOTE: You must have persistent volumes available in your cluster to use this template.

     A Jenkins service has been created in your project.  Log into Jenkins with your OpenShift account.  The tutorial at https://github.com/openshift/origin/blob/master/examples/jenkins/README.md contains more information about using this template.

     * With parameters:
        * Jenkins Service Name=jenkins
        * Jenkins JNLP Service Name=jenkins-jnlp
        * Enable OAuth in Jenkins=true
        * Memory Limit=512Mi
        * Volume Capacity=1Gi
        * Jenkins ImageStream Namespace=openshift
        * Jenkins ImageStreamTag=jenkins:latest

--> Creating resources ...
    route "jenkins" created
    persistentvolumeclaim "jenkins" created.
    deploymentconfig "jenkins" created
    serviceaccount "jenkins" created
    rolebinding "jenkins_edit" created
    service "jenkins-jnlp" created
    service "jenkins" created
--> Success
    Access your application via route 'jenkins-cicd.apps.165f.example.opentlc.com'
    Run 'oc status' to view your app.
[jbjorgee-redhat.com@bastion src]$ oc get pvc
NAME      STATUS    VOLUME     CAPACITY   ACCESSMODES   STORAGECLASS   AGE
jenkins   Bound     pv03-nfs   1Gi        RWO,RWX                      16s
[jbjorgee-redhat.com@bastion src]$
```

Pods not coming up due to erronous triggers in jenkins template. Is trigger refers to jenkins:latest which does not exist.

```
[jbjorgee-redhat.com@bastion src]$ oc get po
No resources found.
[jbjorgee-redhat.com@bastion src]$ oc get all
NAME                        REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfigs/jenkins   0          1         0         config,image(jenkins:latest)

NAME             HOST/PORT                                    PATH      SERVICES   PORT      TERMINATION     WILDCARD
routes/jenkins   jenkins-cicd.apps.165f.example.opentlc.com             jenkins    <all>     edge/Redirect   None

NAME               CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
svc/jenkins        172.30.226.224   <none>        80/TCP      1m
svc/jenkins-jnlp   172.30.4.101     <none>        50000/TCP   1m

[jbjorgee-redhat.com@bastion src]$ oc rollout latest dc/jenkins -n cicd
Error from server (BadRequest): cannot trigger a deployment for "jenkins" because it contains unresolved images
[jbjorgee-redhat.com@bastion src]$

oc edit dc/jenkins
```

Changed

```
  triggers:
  - imageChangeParams:
      automatic: true
      containerNames:
      - jenkins
      from:
        kind: ImageStreamTag
        name: jenkins:latest
        namespace: openshift
    type: ImageChange
```

to

```
  triggers:
  - imageChangeParams:
      automatic: true
      containerNames:
      - jenkins
      from:
        kind: ImageStreamTag
        name: jenkins:2
        namespace: openshift
    type: ImageChange
```

Better now

```
[jbjorgee-redhat.com@bastion src]$ oc get pod
NAME              READY     STATUS    RESTARTS   AGE
jenkins-1-94b4f   1/1       Running   0          42s
[jbjorgee-redhat.com@bastion src]$


[jbjorgee-redhat.com@bastion src]$ oc get route
NAME      HOST/PORT                                    PATH      SERVICES   PORT      TERMINATION     WILDCARD
jenkins   jenkins-cicd.apps.165f.example.opentlc.com             jenkins    <all>     edge/Redirect   None
[jbjorgee-redhat.com@bastion src]$
```

Confirmed access to <https://jenkins-cicd.apps.165f.example.opentlc.com/> using the devops1 user.

## Deploy openshift-tasks using Jenkins

Found (by experimenting in another temporary project ) that the openshift-tasks template refer to a non-existing tag (latest) for jboss-eap as base image. Forked the repo and changed to tag 1.3 that does exist in my installations' image stream for jboss-eap64-openshift [^1].

[^1] <https://github.com/JarleB/openshift-tasks/commit/083936414c229e7ab82fd27d3993409ff3061c7c>

Create template from updated definition

```
[jbjorgee-redhat.com@bastion src]$ oc create -f https://raw.githubusercontent.com/JarleB/openshift-tasks/master/app-template.yaml
template "openshift-tasks" created
[jbjorgee-redhat.com@bastion src]$ oc get template
NAME              DESCRIPTION   PARAMETERS    OBJECTS
openshift-tasks                 3 (all set)   5
[jbjorgee-redhat.com@bastion src]$

[jbjorgee-redhat.com@bastion src]$  oc process openshift-tasks  --parameters
NAME                DESCRIPTION                      GENERATOR           VALUE
APPLICATION_NAME    The name for the application.                        tasks
SOURCE_URL          Git source URI for application                       https://github.com/openshiftdemos/openshift-tasks
SOURCE_REF          Git branch/tag reference                             master
[jbjorgee-redhat.com@bastion src]$
```

Create deployement and build configs

[jbjorgee-redhat.com@bastion src]$ oc new-app --template=openshift-tasks -p SOURCE_URL=<https://github.com/JarleB/openshift-tasks> --> Deploying template "cicd/openshift-tasks" to project cicd

```
     * With parameters:
        * APPLICATION_NAME=tasks
        * SOURCE_URL=https://github.com/JarleB/openshift-tasks
        * SOURCE_REF=master

--> Creating resources ...
    imagestream "tasks" created
    service "tasks" created
    route "tasks" created
    buildconfig "tasks" created
    deploymentconfig "tasks" created
--> Success
    Access your application via route 'tasks-cicd.apps.165f.example.opentlc.com'
    Build scheduled, use 'oc logs -f bc/tasks' to track its progress.
    Run 'oc status' to view your app.
[jbjorgee-redhat.com@bastion src]$

[jbjorgee-redhat.com@bastion src]$ oc get builds
NAME      TYPE      FROM          STATUS    STARTED         DURATION
tasks-1   Source    Git@0839364   Running   4 minutes ago   
[jbjorgee-redhat.com@bastion src]$  

[jbjorgee-redhat.com@bastion src]$ oc logs po/tasks-1-build

(...)

[INFO] Packaging webapp
[INFO] Assembling webapp [jboss-tasks-rs] in [/home/jboss/source/target/openshift-tasks]
[INFO] Processing war project
[INFO] Copying webapp resources [/home/jboss/source/src/main/webapp]
[INFO] Webapp assembled in [67 msecs]
[INFO] Building war: /home/jboss/source/deployments/ROOT.war
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1:49.622s
[INFO] Finished at: Wed Dec 27 08:46:37 EST 2017
[INFO] Final Memory: 25M/104M
[INFO] ------------------------------------------------------------------------
Copying all war artifacts from /home/jboss/source/target directory into /opt/eap/standalone/deployments for later deployment...
Copying all ear artifacts from /home/jboss/source/target directory into /opt/eap/standalone/deployments for later deployment...
Copying all rar artifacts from /home/jboss/source/target directory into /opt/eap/standalone/deployments for later deployment...
Copying all jar artifacts from /home/jboss/source/target directory into /opt/eap/standalone/deployments for later deployment...
Copying all war artifacts from /home/jboss/source/deployments directory into /opt/eap/standalone/deployments for later deployment...
'/home/jboss/source/deployments/ROOT.war' -> '/opt/eap/standalone/deployments/ROOT.war'
Copying all ear artifacts from /home/jboss/source/deployments directory into /opt/eap/standalone/deployments for later deployment...
Copying all rar artifacts from /home/jboss/source/deployments directory into /opt/eap/standalone/deployments for later deployment...
Copying all jar artifacts from /home/jboss/source/deployments directory into /opt/eap/standalone/deployments for later deployment...
Pushing image docker-registry.default.svc:5000/cicd/tasks:latest ...
Pushed 0/6 layers, 2% complete
Pushed 1/6 layers, 22% complete
Pushed 2/6 layers, 59% complete
Pushed 3/6 layers, 78% complete
Pushed 4/6 layers, 93% complete
Pushed 5/6 layers, 97% complete
Pushed 6/6 layers, 100% complete
Push successful
```

Confirmed that tasks app is available on <http://tasks-cicd.apps.165f.example.opentlc.com/>

Now do the build and deploy throught Jenkins

```
[jbjorgee-redhat.com@bastion src]$ oc create -f https://raw.githubusercontent.com/JarleB/openshift-tasks/master/pipeline-bc.yaml
buildconfig "tasks-pipeline" created
[jbjorgee-redhat.com@bastion src]$
[jbjorgee-redhat.com@bastion src]$ oc describe bc tasks
Name:        tasks
Namespace:    cicd
Created:    10 minutes ago
Labels:        app=openshift-tasks
        application=tasks
        template=openshift-tasks
Annotations:    openshift.io/generated-by=OpenShiftNewApp
Latest Version:    1

Strategy:    Source
URL:        https://github.com/JarleB/openshift-tasks
Ref:        master
From Image:    ImageStreamTag openshift/jboss-eap64-openshift:1.3
Force Pull:    yes
Output to:    ImageStreamTag tasks:latest

Build Run Policy:    Serial
Triggered by:        ImageChange, Config
Webhook GitHub:
    URL:    https://loadbalancer1.165f.internal:8443/apis/build.openshift.io/v1/namespaces/cicd/buildconfigs/tasks/webhooks/kJZLvfQr3hZg/github
Webhook Generic:
    URL:        https://loadbalancer1.165f.internal:8443/apis/build.openshift.io/v1/namespaces/cicd/buildconfigs/tasks/webhooks/kJZLvfQr3hZg/generic
    AllowEnv:    false

Build        Status        Duration    Creation Time
tasks-1     complete     6m17s         2017-12-27 08:41:17 -0500 EST

Events:
  FirstSeen    LastSeen    Count    From            SubObjectPath    Type        Reason                Message
  ---------    --------    -----    ----            -------------    --------    ------                -------
  10m        10m        1    buildconfig-controller            Warning        BuildConfigInstantiateFailed    gave up on Build for BuildConfig cicd/tasks (0) due to fatal error: the LastVersion(1) on build config cicd/tasks does not match the build request LastVersion(0)
[jbjorgee-redhat.com@bastion src]$
```

Confirmed that cicd/tasks-pipeline is visible in jenkins, but with no builds yet.

Check pods, builds and buildconfigs

```
[jbjorgee-redhat.com@bastion src]$ oc get pod
NAME              READY     STATUS      RESTARTS   AGE
jenkins-1-94b4f   1/1       Running     0          39m
tasks-1-build     0/1       Completed   0          14m
tasks-1-tljxx     1/1       Running     0          8m
[jbjorgee-redhat.com@bastion src]$ oc get build
NAME      TYPE      FROM          STATUS     STARTED          DURATION
tasks-1   Source    Git@0839364   Complete   14 minutes ago   6m17s
[jbjorgee-redhat.com@bastion src]$ oc get bc
NAME             TYPE              FROM         LATEST
tasks            Source            Git@master   1
tasks-pipeline   JenkinsPipeline                0
```

Start new build with jenkins-pipeline

```
[jbjorgee-redhat.com@bastion src]$ oc start-build tasks-pipeline
build "tasks-pipeline-1" started
[jbjorgee-redhat.com@bastion src]$ oc get build
NAME               TYPE              FROM          STATUS     STARTED              DURATION
tasks-1            Source            Git@0839364   Complete   16 minutes ago       6m17s
tasks-2            Source            Git@0839364   Running    29 seconds ago       
tasks-pipeline-1   JenkinsPipeline                 Running    About a minute ago   
[jbjorgee-redhat.com@bastion src]$
```

Confirmed that build and deployement was done in Jenkins, and that the pipeline was executed and green in the ocp web gui.

And we have a new pod serving the openshift tasks applications.

```
[jbjorgee-redhat.com@bastion src]$ oc get pod
NAME              READY     STATUS      RESTARTS   AGE
jenkins-1-94b4f   1/1       Running     0          44m
tasks-1-build     0/1       Completed   0          19m
tasks-2-build     0/1       Completed   0          3m
tasks-3-bsmxj     1/1       Running     0          56s
[jbjorgee-redhat.com@bastion src]$
```

Confirmed that the tasks application is still available on <http://tasks-cicd.apps.165f.example.opentlc.com/>

## Is persistent storage being used?

Jenkins:

```
[root@bastion ~]# oc get pv |grep cicd
pv03-nfs            1Gi        RWO,RWX       Recycle         Bound       cicd/jenkins                                         5h
pv13-nfs            15Gi       RWO,RWX       Recycle         Bound       cicd-bar5/jenkins                                    7d
[root@bastion ~]# oc whoami
admin
[root@bastion ~]#
[root@support1 ~]# ls /srv/nfs/pv03/
configured                     hudson.plugins.git.GitTool.xml                                  nodeMonitors.xml                                            plugins                   userContent
config.xml                     identity.key.enc                                                nodes                                                       queue.xml.bak             users
config.xml.tpl                 io.fabric8.jenkins.openshiftsync.GlobalPluginConfiguration.xml  org.jenkinsci.main.modules.sshd.SSHD.xml                    secret.key                war
credentials.xml                jenkins.model.JenkinsLocationConfiguration.xml                  org.jenkinsci.plugins.workflow.flow.FlowExecutionList.xml   secret.key.not-so-secret  workflow-libs
credentials.xml.tpl            jobs                                                            org.jenkinsci.plugins.workflow.support.steps.StageStep.xml  secrets
hudson.model.UpdateCenter.xml  logs                                                            password                                                    updates
[root@support1 ~]#
```

Registry:

```
[root@support1 ~]# ls /srv/nfs/registry/docker/registry/v2/repositories/cicd/tasks/
_layers  _manifests  _uploads
[root@support1 ~]#
```

## Trigger build with github webhook

Configured webhook and secret in forked repository to trigger pipeline start build upon push to repo.

Verified that pipeline built and deployed new app with code change. (modified title)

## Autoscale pods

```
[jbjorgee-redhat.com@bastion ~]$ oc autoscale dc/tasks --max=5 --cpu-percent=80
deploymentconfig "tasks" autoscaled
[jbjorgee-redhat.com@bastion ~]$ oc get hpa/tasks
NAME      REFERENCE                TARGETS           MINPODS   MAXPODS   REPLICAS   AGE
tasks     DeploymentConfig/tasks   <unknown> / 80%   1         5         0          27s
[jbjorgee-redhat.com@bastion ~]$
```

Create limitrange in order to start metric collection for pods/containers.

```
[root@bastion ~]# oc create -f /home/jbjorgee-redhat.com/limits.json -n cicd
limitrange "limits" created
[root@bastion ~]# cat  /home/jbjorgee-redhat.com/limits.json
{
    "kind": "LimitRange",
    "apiVersion": "v1",
    "metadata": {
        "name": "limits",
        "creationTimestamp": null
    },
    "spec": {
        "limits": [
            {
                "type": "Pod",
                "max": {
                    "cpu": "300m",
                    "memory": "750Mi"
                },
                "min": {
                    "cpu": "10m",
                    "memory": "5Mi"
                }
            },
            {
                "type": "Container",
                "max": {
                    "cpu": "300m",
                    "memory": "750Mi"
                },
                "min": {
                    "cpu": "10m",
                    "memory": "5Mi"
                },
                "default": {
                    "cpu": "200m",
                    "memory": "100Mi"
                }
            }
        ]
    }
}
[root@bastion ~]#
```

Redeploy.

```
[jbjorgee-redhat.com@bastion ~]$ oc rollout latest dc/tasks -n cicd
```

Limits were to narrow for pods to start when redeploying.

```
[root@bastion ~]# oc edit limitrange  -n cicd
limitrange "limits" edited
[root@bastion ~]#
[root@bastion ~]# oc describe limitrange  -n cicd
Name:        limits
Namespace:    cicd
Type        Resource    Min    Max    Default Request    Default Limit    Max Limit/Request Ratio
----        --------    ---    ---    ---------------    -------------    -----------------------
Pod            cpu            10m    700m    -        -        -
Pod            memory        5Mi    750Mi    -        -        -
Container    cpu            10m    700m    600m        600m        -
Container    memory        5Mi    750Mi    500Mi        500Mi        -
[root@bastion ~]#
```

Now redeployment works, and hpa is getting metrics

```
[jbjorgee-redhat.com@bastion ~]$ oc describe hpa/tasks
Name:                            tasks
Namespace:                        cicd
Labels:                            <none>
Annotations:                        <none>
CreationTimestamp:                    Fri, 29 Dec 2017 03:22:21 -0500
Reference:                        DeploymentConfig/tasks
Metrics:                        ( current / target )
  resource cpu on pods  (as a percentage of request):    28% (113m) / 80%
Min replicas:                        1
Max replicas:                        5
Conditions:
  Type            Status    Reason            Message
  ----            ------    ------            -------
  AbleToScale        False    BackoffDownscale    the time since the previous scale is still within the downscale forbidden window
  ScalingActive        True    ValidMetricFound    the HPA was able to succesfully calculate a replica count from cpu resource utilization (percentage of request)
  ScalingLimited    False    DesiredWithinRange    the desired replica count is within the acceptible range
Events:
  FirstSeen    LastSeen    Count    From                SubObjectPath    Type        Reason                Message
  ---------    --------    -----    ----                -------------    --------    ------                -------
  16m        10m        13    horizontal-pod-autoscaler            Warning        FailedComputeMetricsReplicas    failed to get cpu utilization: missing request for cpu on container tasks in pod cicd/tasks-20-xfngt
  16m        6m        21    horizontal-pod-autoscaler            Warning        FailedGetResourceMetric        missing request for cpu on container tasks in pod cicd/tasks-20-xfngt
[jbjorgee-redhat.com@bastion ~]$ oc get hpa/tasks
NAME      REFERENCE                TARGETS     MINPODS   MAXPODS   REPLICAS   AGE
tasks     DeploymentConfig/tasks   28% / 80%   1         5         3          17m
[jbjorgee-redhat.com@bastion ~]$ oc logs tasks-22-deploy ^C
[jbjorgee-redhat.com@bastion ~]$ oc get hpa/tasks
NAME      REFERENCE                TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
tasks     DeploymentConfig/tasks   1% / 80%   1         5         1          25m
[jbjorgee-redhat.com@bastion ~]$
```

Trying to generate load for 1500s

```
[jbjorgee-redhat.com@bastion ~]$ oc get hpa/tasks
NAME      REFERENCE                TARGETS     MINPODS   MAXPODS   REPLICAS   AGE
tasks     DeploymentConfig/tasks   17% / 80%   1         5         1          28m
[jbjorgee-redhat.com@bastion ~]$ oc get hpa/tasks
NAME      REFERENCE                TARGETS     MINPODS   MAXPODS   REPLICAS   AGE
tasks     DeploymentConfig/tasks   99% / 80%   1         5         1          28m
[jbjorgee-redhat.com@bastion ~]$ oc get hpa/tasks
NAME      REFERENCE                TARGETS     MINPODS   MAXPODS   REPLICAS   AGE
tasks     DeploymentConfig/tasks   99% / 80%   1         5         2          28m
[jbjorgee-redhat.com@bastion ~]$ oc get hpa/tasks
NAME      REFERENCE                TARGETS     MINPODS   MAXPODS   REPLICAS   AGE
tasks     DeploymentConfig/tasks   85% / 80%   1         5         2          30m
[jbjorgee-redhat.com@bastion ~]$ oc get hpa/tasks
NAME      REFERENCE                TARGETS     MINPODS   MAXPODS   REPLICAS   AGE
tasks     DeploymentConfig/tasks   51% / 80%   1         5         2          30m
[jbjorgee-redhat.com@bastion ~]$
[jbjorgee-redhat.com@bastion ~]$ oc get po|grep 23
tasks-23-bx45l    1/1       Running     0          17m
tasks-23-m2qqr    1/1       Running     0          4m
[jbjorgee-redhat.com@bastion ~]$
```

# Multi tenancy

## Project limitation with admissionConfig and custom template

Configure master to limit number of projects based on user label. Default to 1 project.

```
admissionConfig:
.......
.......
    ProjectRequestLimit:
      configuration:
        apiVersion: v1
        kind: ProjectRequestLimitConfig
        limits:
        - selector:
            tenant: bob
          maxProjects: 3
        - selector:
            tenant: alice
          maxProjects: 2
        - maxProjects: 1
......
......
```

Script for creating and labeling new users with tenant

[root@bastion ~]# cat ocp-user.sh

```
#!/bin/bash

set -eo pipefail

if [[ -z $2 ]]
then
echo "usage: $0 <username> <delete/create>"
exit 1
fi

OCP_USER=`oc whoami`

if [[ ${OCP_USER} -ne "admin" ]]
then
  echo "Please log in to openshift (with oc), as admin user"
  exit 2
fi

PW=`pwmake 128`
USERNAME=$1
ACTION=$2

if [[ ${ACTION} == 'create' ]]
then
  ansible masters -m command -a "htpasswd -b /etc/origin/master/htpasswd ${USERNAME} ${PW}"
  oc ${ACTION} user ${USERNAME}
  oc ${ACTION} identity htpasswd_auth:${USERNAME}
  oc ${ACTION} useridentitymapping htpasswd_auth:${USERNAME} ${USERNAME}
  oc label user ${USERNAME} tenant=${USERNAME}
  echo "User ${USERNAME} created with tenant=${USERNAME}  and password ${PW}"
elif [[ ${ACTION} -eq 'delete' ]]
then
  oc ${ACTION} useridentitymapping htpasswd_auth:${USERNAME}
  oc ${ACTION} identity htpasswd_auth:${USERNAME}
  oc ${ACTION} user ${USERNAME}
  ansible masters -m command -a "htpasswd -D /etc/origin/master/htpasswd ${USERNAME}"
  echo "User ${USERNAME} deleted"
fi
```

Run it to create user bob

```
[root@bastion ~]# ./ocp-user.sh bob create
master3.795d.internal | SUCCESS | rc=0 >>
Adding password for user bob

master2.795d.internal | SUCCESS | rc=0 >>
Adding password for user bob

master1.795d.internal | SUCCESS | rc=0 >>
Adding password for user bob

user "bob" created
identity "htpasswd_auth:bob" created
useridentitymapping "htpasswd_auth:bob" created
user "bob" labeled
User bob created with tenant=bob  and password XXXXXXXXXXXXX
[root@bastion ~]#
```

Verify that user bob was created with correct label

```
[root@bastion ~]# oc get user bob --show-labels
NAME      UID                                    FULL NAME   IDENTITIES          LABELS
bob       8d3de5a7-025b-11e8-9449-06c3baf01e9e               htpasswd_auth:bob   tenant=bob
[root@bastion ~]#
```

Create skeleton project template

```
oc adm create-bootstrap-project-template > limit-range-multitenant.yaml
```

Changed name to

```
name: multitenant-template
```

Added to annotations

```
openshift.io/node-selector: tenant=${PROJECT_REQUESTING_USER}
```

Added the following limitrange

```
- apiVersion: v1
  kind: LimitRange
  metadata:
    name: payment
  spec:
    limits:
    - default:
        cpu: 500m
        memory: 512Mi
      defaultRequest:
        cpu: 250m
        memory: 256Mi
      max:
        cpu: 1000m
        memory: 1Gi
      min:
        cpu: 100m
        memory: 100Mi
      type: Container
```

Create template

```
oc create -f limit-range-multitenant.yaml
```

Refer to custom template in /etc/origin/master/master-config.yaml

```
projectRequestTemplate: "default/multitenant-template"
```

Test creation of more than three projects for user bob

```
$ oc whoami
bob

$ oc new-project one
(...)
$ oc new-project two
(...)
$ oc new-project three
(...)
$ oc new-project four
Error from server (Forbidden): projectrequests.project.openshift.io "four" is forbidden: user bob cannot create more than 3 project(s).
```

Add som apps and verify that they land on the correct node

```
$ oc whoami
bob
$ oc new-app --name=one openshift/hello-openshift:v1.1.1.1
(...)
$ oc new-app --name=two openshift/hello-openshift:v1.1.1.1
(...)
$ oc new-app --name=three openshift/hello-openshift:v1.1.1.1
(...)
$ oc get pods  -o wide
NAME            READY     STATUS    RESTARTS   AGE       IP            NODE
one-1-sttl5     1/1       Running   0          38s       10.131.2.16   node1.795d.internal
three-1-x2wxb   1/1       Running   0          16s       10.131.2.20   node1.795d.internal
two-1-wjt2m     1/1       Running   0          26s       10.131.2.18   node1.795d.internal
$
```

Pod's containers get limits assigned from project template's limitrange.

```
$ oc describe pod one-1-sttl5
Name:        one-1-sttl5
Namespace:    three
Node:        node1.795d.internal/192.199.0.195
Start Time:    Fri, 26 Jan 2018 06:47:15 +0100
Labels:        app=one
        deployment=one-1
        deploymentconfig=one
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicationController","namespace":"three","name":"one-1","uid":"5b6f3da6-025c-11e8-9449-06c3baf01e9e","ap...
        kubernetes.io/limit-ranger=LimitRanger plugin set: cpu, memory request for container one; cpu, memory limit for container one
        openshift.io/deployment-config.latest-version=1
        openshift.io/deployment-config.name=one
        openshift.io/deployment.name=one-1
        openshift.io/generated-by=OpenShiftNewApp
        openshift.io/scc=restricted
Status:        Running
IP:        10.131.2.16
Created By:    ReplicationController/one-1
Controlled By:    ReplicationController/one-1
Containers:
  one:
    Container ID:    docker://6bdbb694ee18a7256ba4b47eb9bc8e7ea254cdd6ee2bb846cb9a10b22fb1242b
    Image:        openshift/hello-openshift@sha256:ff52e1c571e7c840f8876d6385437d162be9f30935876f4df6c43f67202a70c9
    Image ID:        docker-pullable://docker.io/openshift/hello-openshift@sha256:ff52e1c571e7c840f8876d6385437d162be9f30935876f4df6c43f67202a70c9
    Ports:        8080/TCP, 8888/TCP
    State:        Running
      Started:        Fri, 26 Jan 2018 06:47:17 +0100
    Ready:        True
    Restart Count:    0
    Limits:
      cpu:    500m
      memory:    512Mi
    Requests:
      cpu:        250m
      memory:        256Mi
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-pgq5z (ro)
Conditions:
  Type        Status
  Initialized     True
  Ready     True
  PodScheduled     True
Volumes:
  default-token-pgq5z:
    Type:    Secret (a volume populated by a Secret)
    SecretName:    default-token-pgq5z
    Optional:    false
QoS Class:    Burstable
Node-Selectors:    tenant=bob
Tolerations:    <none>
Events:
  FirstSeen    LastSeen    Count    From                SubObjectPath        Type        Reason            Message
  ---------    --------    -----    ----                -------------        --------    ------            -------
  1m        1m        1    default-scheduler                    Normal        Scheduled        Successfully assigned one-1-sttl5 to node1.795d.internal
  1m        1m        1    kubelet, node1.795d.internal                Normal        SuccessfulMountVolume    MountVolume.SetUp succeeded for volume "default-token-pgq5z"
  1m        1m        1    kubelet, node1.795d.internal    spec.containers{one}    Normal        Pulled            Container image "openshift/hello-openshift@sha256:ff52e1c571e7c840f8876d6385437d162be9f30935876f4df6c43f67202a70c9" already present on machine
  1m        1m        1    kubelet, node1.795d.internal    spec.containers{one}    Normal        Created            Created container
  1m        1m        1    kubelet, node1.795d.internal    spec.containers{one}    Normal        Started            Started container
```
