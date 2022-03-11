# Openshif - Administración

***********************************************************************************
### MANAGE Openshift Container Platform
***********************************************************************************

### Understand and Use the Command Line and Web Console

```
$ oc login
$ oc new-project my-project
$ oc status
$ oc new-app https://github.com/sclorg/cakephp-ex
$ oc get pods -o wide
$ oc logs cakephp-ex-1-deploy
$ oc api-resources   // muy util para averiguar el apiVersion de un kind.
  EJ: $ oc api-resources | grep OAuth
      config.openshift.io/v1
$ oc help
$ oc create --help
$ oc adm --help

$ tail -10 ocp_cluster/.openshift_install.log
```


### Create and Delete Projects

```
$ oc new-project <project-name> --description="<descritpion>" --display-name="<display-name>"
$ oc get projects
$ oc project <project_name>
$ oc delete project <project_name>
```

LAB:
```
$ oc new-project demotmp --description="Throw away porject for demo" --display-name="Garbage"
$ oc new-project demo --description="Demo project for our lesson" --display-name="Demo Project"
$ oc projects  // muestra con * el actual
$ oc project demotmp
$ oc new-app https://github.com/openshift/ruby-hello-world.git
$ oc get pods
$ oc get all
$ oc delete project demotmp
$ oc get pods
$ oc get all
$ oc projects
```

### Import, Export, and Configure Kubernetes Resources

```
$ oc get -o yaml <resource> <resource>.yaml
$ oc get -o json <resource> <resource>.json
$ oc create -f <resource>.yaml
$ oc create -f <resource>.json
$ oc replace
$ oc extract
$ oc apply
$ oc rsync
```

LAB:
```
$ mkdir resources
$ oc project demo
$ oc get all
$ oc get -o yaml --export pods > resources/pod.yaml
$ ls resources
$ vim resources/pod.yaml
$ oc replace -f resources/pod.yaml
$ oc get secrets
$ oc extract secret/builder-token-6hn9c --to=resources
```

### Examine Resources and Cluster Status

Web console.
Dashboards
```
$ oc adm top node
$ oc adm top node <node>
$ oc adm top pod 
$ oc adm top pod <pod>
```
### View Logs

```
$ oc logs -f bc/<buildconfig_name> 
$ oc logs -f dc/<deploymentconfig_name>
$ oc logs --version=1  dc/<deploymentconfig_name>
$ oc logs -f pod/<pod-name> --tail=5
$ oc logs backend -c <container_name>
$ oc logs -f pod/backend -c <container_name>
```
LAB:
```
$ oc get pods
$ oc logs -f pod/<pod_name> --tail=5
$ oc get all
$ oc logs -f bc/<deploymentconfig_name>
$ oc logs --version=1 bc/<deploymentconfig_name>
```

### Monitor Cluster Events and Alerts

```
$ oc get events -n <project_name>
$ oc get events -n openshift-config
```
LAB:
```
$ oc get events -n openshift-controller-manager
```
web console -> Alerts -> Silence
 

***********************************************************************************
### MANAGE USER AND POLICIES
***********************************************************************************
```
$ sudo yum install httpd-tools
```

### Configure the HTPasswd Identity Provider for Authentication

creo el archivo con la opcion -c

```  
$ hspasswd -c -B -b </path_to/htpasswd_file> <user_name> <passord>
$ hspasswd -B -b </path_to/htpasswd_file> <user_name> <passord>
```

Crea el Htpasswd secret
```
$ oc create secret generic htpass-secret --from-file=htpasswd=</path_to/htpasswd_file> -n openshift-config

apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders
    - name: my_htpasswd_provider
      mappingMethod: claim
      type: HTPasswd
      htpasswd:
        fileData:
          name: htpass-secret

$ oc aplly -f <path_to/CR>
$ oc login -u <username>
$ oc whoami
```
LAB:
```  
$ hspasswd -c -B users.htpasswd admin   // como le puse la opcion -b me va a pedir el password
$ for USER in tbonino lbonino
$ hspasswd -B users.htpasswd tbonino
$ cat users.htpasswd
$ oc create secret generic htpass-secret --from-file=htpasswd=users.htpasswd -n openshift-config
$ oc get secrets
$ vim htpasswd.yaml

apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: clusteroc ñlpo
spec:
  identityProviders
    - name: users.htpasswd
      mappingMethod: claim
      type: HTPasswd
      htpasswd:
        fileData:
          name: htpass-secret
$ oc apply -f tepasswd.yaml
$ oc login -u abonino
$ oc whoami
```
  
web console 
Cluster Settings->Global Configuration -> OAuth

### Create and delete Users

```  
$ ocget secret -n openshift-config
$ oc extract secret/HTPASSWD_SECRET --to - -n openshift-config > htpasswd
Agrego usuarios
$ hspasswd -B -b htpasswd <user_name> <password>
$ oc create secret generic HTPASSWD_SECRET --from-file=htpasswd --dry-run -o yaml -n openshift-config | oc replace -f -
$ oc fet identity
$ ode get users
```
  
LAB:
```  
$ oc get secret -n openshift-config
htpass-secret
$ oc extract secret/htpass-secret --to - -n openshift-config > users.htpasswd
$ cat users.htpasswd
$ for USER in tbonino lbonino; do  hspasswd -B -b users.htpasswd $USER passwd;done
$ cat users.htpasswd
$ oc create secret generic htpass-secret --from-file=htpasswd=users.htpasswd --dry-run -o yaml -n openshift-config | oc replace -f -
$ oc get identity
$ oc login -u abonino
$ oc get identity
$ oc get users
$ oc extract secret/htpass-secret --to - -n openshift-config > users.htpasswd
$ vim users.htpasswd
borro la linea con el user a sacar
$ oc create secret generic htpass-secret --from-file=htpasswd=users.htpasswd --dry-run -o yaml -n openshift-config | oc replace -f -
$ oc get indentity
$ oc delete identity/users.htpasswd:<user_borrado>
$ oc get indentity
$ oc get users
$ oc delete user/<user_borrado>
```


### Modify user Passwords

```  
$ oc get secret -n openshift-config
$ oc extract secret/HTPASSWD_SECRET --to - -n openshift-config > htpasswd
Moidifico el password
$ hspasswd -B -b htpasswd <user_name> <password>
$ oc create secret generic HTPASSWD_SECRET --from-file=htpasswd --dry-run -o yaml -n openshift-config | oc replace -f -
```

### Modify User and Group Permissions

Defaults Cluster Roles

admin:puede ver y modificar cualquier recurso excepto la cuota.
basic-user: puede get info basica de proyectos y users.
cluster-admin: Super-user. Puede realizar cualquier accion en cualquier proyecto.
cluster-status: puede ver cluster status information
edit: puede modificar muchos objetos pero no puede ver o modificar roles o bindings.
self-provisioner: puede crear sus propios objetos
view:No puede hacer ninguna modificacion, pero púede ver objetos en un proyecto.No puede modificar roles o bindings.

```  
$ oc adm policy add-role-to-user <role> <user> -n <project>
$ oc adm policy add-role-to-group <role> <group> -n <project>
$ oc describe rolebinding.rbac -n <project>
$ oc adm policy add-cluster-role-to-user cluster-admin <user>
$ oc delete secrets kubeadmin -n kube-system
```
LAB:
```  
$ oc adm policy add-role-to-user admin abonino -n desa1
$ oc adm policy add-role-to-user edit tbonino -n desa1
$ oc adm policy add-role-to-user view lbonino -n desa1
$ oc describe rolebinding.rbac -n desa1
$ oc adm policy add-cluster-role-to-user cluster-admin admin
$ oc delete secrets kubeadmin -n kube-system
```
### Create and Manage Groups

```
$ oc adm groups new <group>
$ oc adm groups new <group> <user>
$ oc adm groups add-users <group> <user1> <user2>
$ oc adm groups remove-users <group> <user1> <user2>
```
  
Defult Virtul Group
system:autheticated
system:authenticated:oauth
system:unauthenticated

LAB:
```
$ oc new-project demo2
$ oc adm groups new dev
$ oc adm groups add-user dev abonino tbonino
$ oc adm policy add-role-to-group edit dev -n demo2
$ oc adm groups remove-users dev tbonino
```
 
  
***********************************************************************************
### CONTROL ACCESS TO RESOURCES 
***********************************************************************************

### Define Role-Based Access Controls

Creacion de un role local
```
$ oc create role <name> --verb=<verb> --resource=<resource> -n <project>
```
Creacion de un role de cluster
```
$ oc create clusterrole <name> --verb=<verb> --resource=<resource>
```
LAB:
```
$ oc create role podview --verb=get --resource=pod -n desa1
$ oc adm policy add-role-to-user podview abonino --role-namespace=desa1 -n desa1
$ oc create clusterrole podviewonly --verb=get --resource=pod
$ oc adm policy add-cluster-role-to-user podviewonly tbonino
$ oc describe rolebinding.rbac -n desa1
$ od describe clusterrolebinding.rbac podviewonly
```

### Apply Permissions to Users

```
$ oc adm policy who-can <verb> <resource>
$ oc adm policy add-role-to-user <role> <user>
$ oc adm policy remove-role-to-user <role> <user>
$ oc adm policy remove-user <user>
$ oc adm policy add-role-to-group <role> <group>
$ oc adm policy remove-role-from-group <role> <group>
$ oc adm policy remove-group <group>

$ oc adm policy add-cluster-role-to-user <role> <user>
$ oc adm policy remove-cluster-role-from-user <role> <user>
$ oc adm policy add-cluster-role-to-group <role> <group>
$ oc adm policy remove-cluster-role-from-group <role> <group>
```
LAB:
```
$ oc project demo
$ oc adm policy who-can get pod
$ oc adm policy who-can create pod
```

### Create and Apply Secrests to Manage Sensitive Information

Tipos de secrests:

- kubernetes.io/service-account-token

- kubernetes.io/basic-auth

- kubernetes.io/ssh-auth

- kubernetes.io/tls

- Opaque

Ejemplo de secret.yaml
```
apiVersion: v1
kind: secret
metadata:
  name: mysecret
type: Opaque
data:
  username: dXnsdNasaskdj
  password: aasiDdsyHSDJI=

$ oc create -f secret.yaml
```

Ejemplo de uso de secret

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-example-pod
spec:
  containers:
    - name: secret-test-container
      image: busybox
      command: [" /bin/sh", "-c", "export" ]	
      env: 
        - name: TEST_SECRET_USERNAME_ENV_VAR		
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: username
      restartpolicy: Never		   		
```
```
apiVersion: v1
kind: BuilConfig
metadata:
  name: secret-example-bc
spec:
  strategy:
    sourceStrategy:
      env: 
        - name: TEST_SECRET_USERNAME_ENV_VAR		
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: username
```

LAB:
```
$ oc project demousy
$ echo -n 'abonino' | base64
$ echo -n 'password' | base64
usar estos datos para poner en el secrets
$ oc create -f secret.yaml
$ oc get secrets -n demo
$ oc describe secret/mysecret -n demo
```

### Create Service Accountas and Apply Permissions Using Security Context Constraints

Una cuenta de servivio permite que un componente acceda directamente a la API. (Sin compartir las credenciales de un usuario normal).

El nombre se deriva de su proyecto:

system:serviceaccount:<project>:<name_sa>
  
Cada cuenta de servicio pertenece a dos grupos:
  
- system:serviceaccounts (incluye todas las cuentas de servicio del sistema)   
- system:serviceaccounts:<proyecto>  (Incluye todas las cuentas de servcio del proyecto)
  
Cada cuenta de servicio tiene dos secrets asociados. Un token de API y credenciales para Openshift Container Registry.



```
$ oc get sa
$ oc create sa <service-account-name>  // la crea en el proyecto actual
$ oc describe sa/<service-account-name>
Asigna un role a la SA en el projecto actual
$ oc adm policy add-role-to-user <role> system:serviceaccount:<project>:<service-account-name>
  // con la opcion -z le otorgo a la sa acceso a un proyecto.
$ oc adm policy add-role-to-user <role> -z <service-account-name>  
```

ejemplo sa.yaml
```
apiVersion: v1
kind: SecurityContextXConstraints
metadata:
  name: scc-admin
allowPrivilegedContainer: true
runAduser:
  type: RunAsUser
seLinuxContext:
  type: RunAdUser
fsGroup:
  type: RunAsUser
supplementlGroups:
  type: RunAsUser
user:
- <service-account-name>

$ oc create -f sa.yaml
```

SCC: controlan los permisos de los pods. Incluyen acciones que puede realizar un pod y a que recurso puede acceder.   
  
LAB:
 
```
$ oc creta sa demosa
$ oc get sa
$ oc describe sa demosa  
$ vim scc-admin.yaml
apiVersion: v1
kind: SecurityContextXConstraints
metadata:
  name: scc-admin
allowPrivilegedContainer: true
runAduser:
  type: RunAsUser
seLinuxContext:
  type: RunAdUser
fsGroup:
  type: RunAsUser
supplementlGroups:
  type: RunAsUser
user:
- demosa
requiredDropCapabilities:
- KILL
- MKNOD
- SYS_CHROOT

$ oc create -f scc-admin.yaml
$ oc get scc
$ oc describe scc scc-admin
```

***********************************************************************************
### Configure Networking Components 
***********************************************************************************

### Troubleshoot Software Defined Networking

Mustra el estado del CNO deployment (Cluster network operator)
```
$ oc get -n openshift-network-operator deployment/network-operator
```
ver el estado del CNO
```
$ oc get clusteroperator/network

$ oc describe network.config/cluster

$ oc logs --namespace=openshift-network-operator deployment/network-operator

$ oc get -n openshift-dns-operator deployment/dns-operator
$ oc get clusteroperator/dns
$ oc describe clusteroperator/dns
$ oc logs --namespace=openshift-dns-operator deployment/dns-operator
```

Debuggins Routes
```
$ oc get endpoints -n <porject>
$ oc get route -n <project>
$ oc get pods -n <project> --template='{{range.items}}HostIP: {{status.hostIP}} PodID: {{.status.podIP}}{{"\n"}}{{end}}'
$ oc get services -n <project>
$ oc get endpoints -n <project>
```
LAB:
```
$ oc get -n openshift-network-operator deployment/network-operator
$ oc get clusteroperator/network
$ oc describe network.config/cluster
$ oc oc logs -n openshift-network-operator deployment/network-operator --tail 10
$ oc get -n openshift-dns-operator deployment/dns-operator
$ oc get clusteroperator/dns
$ oc describe clusteroperator/dns
$ oc get endpoint -n demo3
$ oc get pods -n demo3 --template='{{range.items}}HostIP: {{status.hostIP}} PodID: {{.status.podIP}}{{"\n"}}{{end}}'
$ oc get pods -n demo3 -o wide
$ oc get route -n demo3
```

### Create and Edit External Routes

```
$ oc expose service <service_name>
```

Crear un route con una etiqeuta y un nombre
```
$ oc expose service <service_name> -l name=<label> --name=<route_name>
$ oc expose service <service_name> --port=<port> --protocol="<protocol>"
$ oc expose service <service_name> --path=<path>

$ oc annotate route <route> --overwrite haproxy.router.openshift.io/timeout=<timeout_unit>
$ oc annotate route <route> router.openshift.io/<cookie_name>="-<cookie_annotation>"
$ oc annotate route <route> haproxy.router.openshift.io/ip_whitelist="<ip1 ip2 ip3>"
$ oc annotate route <route> haproxy.router.openshift.io/rate-limit-connections=true
```
LAB:
```
$ oc project demo
$ oc get services
$ oc expose service ruby-hello-world -l name=hello --name=helloworld
$ oc annotate route helloworld --overwrite haproxy.router.openshift.io/timeout=5s
$ oc annotate route helloworld router.openshift.io/helloworld="-helloworld_annotation"
$ oc annotate route helloworld haproxy.route.openshift.io/rate-limit-connections=true
$ oc describe route helloworld
```

### Control Cluster Network Ingress

```
$ vim example-ingress.yaml

apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: example
  namespace: <project>
spec:
  rules:
  - host: example.com
    http:
      paths:
        - path: /testpath
          backend:  
            serviceName: test
            servicePort: 80

$ oc apply -f example-ingress.yaml
```

LAB:
```
$ oc get service -n demo3
$ oc expose service django-psql-example -n demo3
$ oc get route -n demo3
django-psql-example-demo3.apps.ocplab.openshift.labdemodomain.com
$ vim demo3-ingress.yaml 
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: demo3-ingress
  namespace: demo3
spec:
  rules:
  - host: django-psql-example-demo3.apps.ocplab.openshift.labdemodomain.com
    http:
      paths:
        - path: /demo
          backend:  
            serviceName: postgresql
            servicePort: 5432
$ oc apply -f demo-ingress.yaml
$ oc get ingress -n demo3
$ oc get routes -n demo3
$ oc describe ingress/demo3-ingress -n demo3
```

### Create a Self Signed Certificate


Crear self-signed certificate
```
$ openssl req -x509 -newkey rsa:4096 -nodes -keyout cert.key -out cert.crt
```

Remove a paddphrase from a key file
```
$ openssl rsa -in <password_protected_key_file>.key -out <key_file>.key
```
LAB:
```
$ openssl req -x509 -newkey rsa:4096 -nodes -keyout helloworld.key -out helloworld.crt
:US
:Ohio
:Columbis
:DoubleTap
:IT
:helloworld-demo.apps.ocplab.openshift.labdemodomain.com
:aaa@gmail.com
$ ls
$ cat helloworld.crt
$ cat helloworld.key



$ openssl rsa -in <password_protected_key_file>.key -out <key_file>.key
```

### Secure Routes Using TLS Certificates

```
$ oc create route edge --service=frontend --cert=tls.crt --key=tls.key --hostaname=www.example.com
```

Examine Route

```
apiVersion: v1
kind: Route
metadata:
  name: frontend
spec: 
  host: www.example.com
  to: 
    kind: Service
    name: frontend
  tls:
    termination: edge
    key: |-
       -----BEGIN PRIVATE KEY-----
       [...]
       -----END PRIVATE KEY-----
    crtificate: |-
       -----BEGIN CERTIFICATE-----
       [...]
       -----END CERTIFICATE-----
```

LAB:

```
$ oc project demo
$ oc delete route hellowprld
$ oc get services
ruby-hello-world
$ oc create route edge helloworld --service=ruby-hello-world --cert=helloworld.crt --key=helloworld.key --hostname=helloworld-demo.apps.ocplab.openshift.laddemodomain.com
$ oc get routes
$ oc describe route helloworld
$ oc edit route helloworld
```

***********************************************************************************
### Configure Pod Scheduling
***********************************************************************************

### Limite Resource Usage


core-object-counts.yaml
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: core-object-counts
spec:
  hard:
    configmap: "10"	
    persistentvolumeclaims: "4"
    replicationcontrollers: "20"
    secrets: "10"
    services: "10"
    services.loadbalance: "2"
```
compute-resources-long-running.yaml
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources-long-running
spec:
  hard:
    pods: "4"	
    limits.cpu: "4"
    limits.memory: "2Gi"
    limits.ephemeral-storage: "4Gi"
  scope:
  - NotTerminating  
```
```
$ oc create -f <file> -n <project_name>
$ oc create quota <name> --hard=count/<resource>=<quota>,<resource>/<resource>=<quota>
$ oc get quota -n <project_name>
$ oc describe quota <quota_name> -n <project_name>
```
LAB:
```
$ vim compute-resources-yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources-long-running
spec:
  hard:
    pods: "4"	
    limits.cpu: "4"
    limits.memory: "2Gi"
    limits.ephemeral-storage: "4Gi"
    configmaps: "7"
    secrets: "10"
    services: "8"

$ oc create -f compute-resources-yaml -n demo
$ oc get quota -n demo
$ oc describe quota compute-resources -n demo
```

### Scale Applications to Meet Increased Demand

```
$ oc scale --replicas=<numbre> dc/<deployment_name>
$ oc scale --replicas=2 dc/ruby-hello-wolrd
$ oc autoscale dc/<deployment_name> --min <number> --max <number> --cpu-percent=<percentage>
$ oc autoscale dc/<deployment_name> --min 1 --max 5 --cpu-percent=75
```

LAB:

```
$ oc project demo
$ oc get pods
$ oc get deploymentconfig
$ oc scale --replicas=3 dc/ruby-hello-world
$ oc get deploymentconfig
$ oc scale --replicas=2 dc/ruby-hello-world
$ oc autoscale dc/ruby-hello-world --min 1 --max 4 --cpu-percent=90
$ oc get hpa
$ oc edit hpa ruby-hello-world
$ vim cpu-autoscale.yaml
apiVersion: autoscaling/v1
kind: horizontalPodAutoscaler
metadata:
  name: cpu-autoscale
spec:
  maxReplicas: 4
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps.openshift.io/v1
    kind: DeploymentConfig
    name: ruby-hello-world
  targetCPUUtilizationPrecentage: 90
```


### Control Pod Placement Across Cluster Nodes

```
$ oc create configmap -n openshift-config --from-file=policy.cfg <configmap>
```

Agregar un ConfigMar al Scheduler Operator CR
```
$ oc patch Scheduler cluster --type='merge' -p '{"spec":{"policy":{"name":<configmap>}}}

$ oc logs <scheduler_pod> | grep predicates
```

LAB:

```
$ vim policy.cfg
{
"kind" : "Policy",
"apiVersion" : "v1",
"predicates" : [
        {"name" : "PodFitsHostPorts"}, 
        {"name" : "PodFitsResources"}, 
        {"name" : "NoDiskConflict"}, 
        {"name" : "NoVolumeZoneConflict"}, 
        {"name" : "MatchNodeSelector"}, 
        {"name" : "MaxEBSVolumeCount"}, 
        {"name" : "checkServiceAffinity"},
        {"name" : "MaxAzureDiskAffinity"}, 
        {"name" : "PodToleratesNodeNoExecuteTaints"}, 
        {"name" : "MaxGCEPDVolumeCount"}, 
        {"name" : "MatchInterPodAffinity"}, 
        {"name" : "PodToleratesNodeTaints"}, 
        {"name" : "HostName"} 
        ],
"priorities" : [
        {"name" : "LeastRequetedPriority", "weight",1}, 
        {"name" : "BalacedResourceAllocation", "weight",1}, 
        {"name" : "ServiceSpreadingPriority", "weight",1}, 
        {"name" : "EqualPriority", "weight",1}
        ]
}
$ oc create configmap -n openshift-config --from-file=policy.cfg pod-scheduler-policy
$ oc patch Scheduler cluster --type='merge' -p '{"spec":{"policy":{"name":"pod-scheduler-policy"}}}}
$ oc get pods -n openshift-kube-scheduler
$ oc logs openshift-kube-scheduler-ip-10-0-142-79.ec2.interna -n openshift-kube-scheduler | grep predicates
$ oc get configmap -n openshift-config
$ oc edit configmap pod-scheduler-policy -n openshift-config 
```


***********************************************************************************
### Configure Cluster Scaling
***********************************************************************************

### Manually Control the Number of Cluster Workers

```
$ oc get machinesets -n openshift-machine-api
```
Dos maneras de escalamiento manual.

```
$ oc scale --replicas=2 machineset <machineset> -n openshift-machine-api
$ oc edit machineset <machineset> -n openshift-machine-api
```

LAB: 
```
$ oc get machinesets -n openshift-machine-api
$ oc scale --replicas=2 machineset <el-que-mostro-el-comnado-anterior> -n openshift-machine-api
$ oc get nodes
veo dos worker
$ oc edit machineset <el-que-mostro-el-comnado-anterior> -n openshift-machine-api
Voy hasta replicas en la seccion de spec y edito el numero por ejemplo por 3
$ oc get machinesets -n openshift-machine-api
$ oc get nodes
Vemos tres worker nuevamente
$ oc get machinesets -n openshift-machine-api
```

### Automatically Control the Number of Cluster Workers

Machine Autoscaler

```
apiVersion: "autoscaling.openshift.io/v1beta1"
kind: "MachineAutoscaler"
metadata:
  name: "worker-us-east-la"
  namespace: "openshift-machine-api"
spec:
  minReplicas: 1
  maxReplicas: 4
  scaleTargetRef:
    apiVersion: machine.openshift.io/v1beta1
    kind: MachineSet
    name: "worker-us-east-la"

$ oc create -f <filename.yaml>
```


LAB:


```
$ oc get machinesets -n openshift-machine-api
3 master 3 worker
$ vim MachineAutoscaler.yaml
apiVersion: "autoscaling.openshift.io/v1beta1"
kind: "MachineAutoscaler"
metadata:
  name: "worker-us-east-la"
  namespace: "openshift-machine-api"
spec:
  minReplicas: 1
  maxReplicas: 4
  scaleTargetRef:
    apiVersion: machine.openshift.io/v1beta1
    kind: MachineSet
    name: apclab-l96n4-worker-us-east-1a
$ oc create -f MachineAutoscaler.yaml
$ oc get machinesets -n openshift-machine-api
$ oc edit machineautoscaler worker-autoscaler -n openshift-machine-api
```
lo edito y o salvo
```
$oc describe machineautoscaler weorker-autoscaler -n openshift-machine-api
```

