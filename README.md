<p align="center">
  <a href="https://kubernetes.io/" title="Redirect to Kubernetes page">
    <img src="https://github.com/kubernetes/kubernetes/raw/master/logo/logo.png" width="250" />
  </a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/contributions-welcome-brightgreen.svg">
  <img src="https://img.shields.io/github/issues-pr/StenlyTU/K8s-training-official">
  <img src="https://img.shields.io/github/stars/StenlyTU/K8s-training-official?style=social">
  <img src="https://img.shields.io/github/forks/StenlyTU/K8s-training-official?style=social">
</p>

# K8s Practice Training

The goal of this tutorial is to give good understanding of Kubernetes and help preparing you for CKA, CKAD and CKS.

To achieve this you need running Kubernetes cluster.

During the tutorial every user is going to create personal namespace and execute all exercises there.

There are 50+ tasks with increasing difficulty. Tested with K8s version 1.19.2 and kubectl version 1.19.2.

## K8s learning materials:

1. Docker is a must. You can start with the book **_Docker in Action_**. The book can be [_downloaded_](https://www.pdfdrive.com/docker-in-action-e34422630.html) from Internet.
2. Check the free K8s courses in EDX: https://www.edx.org/course/introduction-to-kubernetes
3. The book **_Kubernetes in action_** gives good general overview. The book can be [_downloaded_](https://github.com/indrabasak/Books/blob/master/Kubernetes%20in%20Action.pdf) from Internet.
4. For security related topics have a look at **_Container Security by Liz Rice_**. The book can be [_downloaded_](https://cdn2.hubspot.net/hubfs/1665891/Assets/Container%20Security%20by%20Liz%20Rice%20-%20OReilly%20Apr%202020.pdf) from Internet.
5. And ofc https://kubernetes.io/docs/home/

## Hands-on experience:

Download the kubeconfig file from your cluster and configure kubectl to use it.

    export KUBECONFIG=/path/to/the/kubeconfig.yaml

### Core Concepts

1.  **Create a namespace called _my-practice_. All of the following commands should be executed into this namespace if not specified differently.**

    <details><summary>show</summary><p>

    ```
    kubectl create ns my-practise
    ```

    **Take-away**: always try to use shortnames. To find the shortname of resource run -> _kubectl api-resources | grep namespaces_

2.  **Create two pods with image named _dokuwiki1_ and _dokuwiki2_ into your namespace. All of them should have the label _app=dokuwiki_ .**
     <details><summary>show</summary><p>

        kubectl run -n my-practice dokuwiki1 --image=bitnami/dokuwiki --restart=Never --labels=app=dokuwiki
        kubectl run -n my-practice dokuwiki2 --image=bitnami/dokuwiki --restart=Never --labels=app=dokuwiki

    **Take-away**: Try to learn most important _kubectl run_ options which can save you a lot of time and manual work on yaml files.

3.  **Change pod _dokuwiki2_ label to _app=wiki_.**
    <details><summary>show</summary><p>

        kubectl -n practice label po dokuwiki2 app=wiki --overwrite

    **Take-away**: use _--overwrite_ when **changing** labels.

4.  **Get only pods with label _app=wiki_ from all namespaces.**
    <details><summary>show</summary><p>

        kubectl get pods --all-namespaces=true -l app=wiki

    **Take-away**: -l can be used to filter resources by labels.

    **Alternative**: `kubectl get pods -A -l app=wiki`

5.  **Remove the nginx pods to clean your namespace.**
    <details><summary>show</summary><p>

        kubectl -n practice delete pod dokuwiki{1,2}

    **Take-away**: Here we are using Brace Expansion to save time.

6.  **Create a messaging pod using _bitnami/redis:6.2.7_ image with label set to _tier=msg_. Check pod's labels.**

    <details><summary>show</summary><p>

        kubectl run -n my-practice messaging --image=bitnami/redis:6.2.7 -l tier=msg

    ```
    kubectl -n practice describe pod messaging| head
    Name:         messaging
    Namespace:    practice
    Priority:     0
    Node:         ip-10-250-13-141.eu-central-1.compute.internal/10.250.13.141
    Start Time:   Sun, 19 Apr 2020 16:25:19 +0300
    Labels:       tier=msg
    Annotations:  cni.projectcalico.org/podIP: 100.96.1.4/32
                  cni.projectcalico.org/podIPs: 100.96.1.4/32
                  kubernetes.io/psp: extensions.gardener.cloud.provider-aws.csi-driver-node
    Status:       Running
    ```

    **Take-away**: Use -l alongside _kubectl run_ to create pods with specific label.

7.  **Create a service called _messaging-service_ to expose the messaging application within the cluster on port 6379 and describe it.**
    <details><summary>show</summary><p>

    ```
    kubectl -n my-practice expose pod messaging --name messaging-service --port 6379
    ```

    ```
    $ kubectl -n practice describe svc messaging-service
    Name:              messaging-service
    Namespace:         practice
    Labels:            tier=msg
    Annotations:       <none>
    Selector:          tier=msg
    Type:              ClusterIP
    IP:                100.67.250.244
    Port:              <unset>  6379/TCP
    TargetPort:        6379/TCP
    Endpoints:         100.96.0.20:6379
    Session Affinity:  None
    Events:            <none>
    ```

    **Take-away**: _kubectl expose_ is easy way to create service automatically when applicable.

8.  **Create a _bitnami/busybox-echo_ pod that echoes 'Hello Kubernetes Learners' and exits. After that check the logs.**

    <details><summary>show</summary><p>

        kubectl -n my-practice run busybox-echo --image=busybox --command -- echo "Hello Kubernetes Learners!"
        kubectl -n my-practice logs busybox-echo

    **Take-away**: with _--command_ we can execute commands from within the container.

9.  **Create an _nginx-test_ pod using the Bitnami resoure and set an env value as _var1=val1_. Check the env value existence within the pod.**

    <details><summary>show</summary><p>

        kubectl -n my-practice run nginx-test --image=bitnami/nginx --env=var1=val1
        kubectl -n my-practice exec -it nginx-test -- env # should see var1=val1 in the output

### Deployments

10. **Create a deployment named _hr-app_ using the image _nginx:1.18_ with 2 replicas.**

    <details><summary>show</summary><p>

        kubectl -n practice create deployment hr-app --image=nginx:1.18 --replicas=2

    **Take-away**: _--replicas=2_ is the number of replicas to create, default is 1.

11. **Scale _hr-app_ deployment to 3 replicas.**
    <details><summary>show</summary><p>

        kubectl -n practice scale deploy/hr-app --replicas 3

    **Take-away**: resource_type/resource_name syntax can also be used.

12. **Update the _hr-app_ image to nginx:1.19.**
    <details><summary>show</summary><p>

        kubectl -n practice set image deploy/hr-app nginx=nginx:1.19

    **Take-away**: `nginx=nginx:1.19` syntax is confusing. `nginx` is the container name.

    **Alternative**: You can also edit the deployment manually with _kubectl -n practice edit deploy/hr-app_

13. **Check the rollout history of _hr-app_ and confirm that the replicas are OK.**
     <details><summary>show</summary><p>

        kubectl -n practice rollout history deploy hr-app
        kubectl -n practice get deploy hr-app
        kubectl -n practice get rs # check that a new replica set has been created
        kubectl -n practice get po -l app=hr-app

14. **Undo the latest rollout and verify that new pods have the old image (nginx:1.18)**
    <details><summary>show</summary><p>

        kubectl -n practice rollout undo deploy hr-app
        kubectl -n practice get po # select one of the 'Running' pods
        kubectl -n practice describe po hr-app-695f79495-6gfsw | grep -i Image: # should be nginx:1.18

15. **Do an update of the deployment with a wrong image _nginx:1.91_ and check the status.**
    <details><summary>show</summary><p>

        kubectl -n practice set image deploy/hr-app nginx=nginx:1.91
        kubectl -n practice rollout status deploy hr-app
        kubectl -n practice get po # you'll see 'ImagePullBackOff'

16. **Return the deployment to working state and verify the image is _nginx:1.19_.**
    <details><summary>show</summary><p>

        kubectl -n practice rollout undo deploy hr-app
        kubectl -n practice describe deploy hr-app | grep Image:
        kubectl -n practice get pods -l app=hr-app

### Scheduling

17. **Shedule a nginx pod on specific node using _NodeName_.**

     <details><summary>show</summary><p>

    [Assigning Pods to Nodes documentation](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodename)

    Generate yaml file:

        kubectl -n practice run nginx-nodename --image nginx --dry-run=client -o yaml > nodename.yaml

    Choose one of the nodes(kubectl get nodes) and edit the file:

    ```YAML
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
         run: nginx-nodename
      name: nginx-nodename
    spec:
      nodeName: <node_name> # add
      containers:
      - image: nginx
        name: nginx-nodename
        resources: {}
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    status: {}
    ```

    Create the pod and check where the pod was scheduled.

    _Hint: Use '-o wide' to check on which node the pod landed._

    **Take-away**: _--dry-run=client_ is used to check if the resource can be created. Adding _-o yaml > filename.yaml_ redirects the raw output to file.

18. **Schedule a nginx pod based on node label using _nodeSelector_.**

    <details><summary>show</summary><p>

    [Assigning Pods to Nodes documentation](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)

    Pick one of the nodes and check for hostname label.

        kubectl describe node <node-name> | grep hostname

    Generate yaml file and add nodeSelector field with above label as described into the documentation.

    Check if the pod has landed at the correct node.

    **Take-away**: use nodeSelector when you want to schedule pods only on nodes with specific labels.

19. **Taint a node with key=spray, value=mortein and effect=NoSchedule. Check that new pods are not scheduled on it.**

    <details><summary>show</summary><p>

    [Taint and Toleration documentation](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)

        kubectl taint nodes <node-name> spray=mortein:NoSchedule

    Create nginx pod and check that it's not scheduled onto the tainted node.

    **Take-away**: A taint allows a node to refuse pod to be scheduled unless that pod has a matching toleration.

20. **Create another pod called _nginx-toleration_ with _nginx_ image, which tolerates the above taint.**

    <details><summary>show</summary><p>

    [Taint and Toleration documentation](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)

    Use the documentation to figure out the yaml and check that the pod has landed onto the tainted node.

    Delete the pod and remove the taint from the node. Use the following to remove the taint:

        kubectl taint nodes <node-name> spray=mortein:NoSchedule-

    **Take-away**: Pods can be scheduled on taint nodes if they tolerate the taint.

21. **Create a _DaemonSet_ using image _fluentd-elasticsearch:1.20_.**

    <details><summary>show</summary><p>

    [DaemonSet documentation](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

    Use this yaml or try to make it alone from the documentation.

    ```YAML
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      creationTimestamp: null
      labels:
        app: elastic-search
      name: elastic-search
      namespace: practice
    spec:
      selector:
        matchLabels:
          app: elastic-search
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: elastic-search
        spec:
          containers:
          - image: k8s.gcr.io/fluentd-elasticsearch:1.20
            name: fluentd-elasticsearch
            resources: {}
    ```

    **Take-away**: A DaemonSet ensures that all (or some) Nodes run a copy of a Pod.

22. **Add label _color=blue_ to one node and create nginx deployment called _blue_ with 5 replicas and node Affinity rule to place the pods onto the labeled node.**

    <details><summary>show</summary><p>

    [Affinity and anti-affinity documentation](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)

        kubectl label node <node-name> color=blue

    Generate your own yaml or use this one. There is something wrong with it(indentation or something is missing).

    ```YAML
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: blue
    spec:
      replicas: 5
      selector:
        matchLabels:
          run: nginx
    template:
        metadata:
          labels:
            run: nginx
        spec:
          containers:
          - image: nginx
            imagePullPolicy: Always
            name: nginx
          affinity:
            requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                - matchExpressions:
                  - key: color
                    operator: In
                    values:
                    - blue
    ```

    Check that all pods are scheduled onto the labeled node.

    **Take-away**: Node affinity is a set of rules used by the scheduler to determine where a pod can be placed.

### Configurations

23. **Create a configmap named _my-config_ with values _key1=val1_ and _key2=val2_. Check it's values.**
    <details><summary>show</summary><p>

    [ConfigMap documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

        kubectl -n practice create configmap my-config --from-literal=key1=val1 --from-literal=key2=val2
        kubectl -n practice get cm my-config -o yaml

    **Take-away**: ConfigMap gives you a way to inject configurational data into your application.

24. **Create a configMap called _opt_ with value _key5=val5_. Create a new _nginx-opt_ pod that loads the value from key _key5_ in an env variable called _OPTIONS_.**
    <details><summary>show</summary><p>

    [Configmap documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap)

    Use the documentation to figure out the yaml file.

        kubectl -n practice exec -it nginx-opt -- env | grep OPTIONS # should return val5

    **Take-away**: ConfigMap is namespaced resource.

25. **Create a configmap _anotherone_ with values _var6=val6_ and _var7=val7_. Load this configmap as an env variables into a _nginx-sec_ pod.**
    <details><summary>show</summary><p>

    [Configmap documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap)

        kubectl -n practice exec -it nginx-sec -- env | grep var # should return var6=val6\nvar7=val7

26. **Create a configMap _cmvolume_ with values _var8=val8_ and _var9=val9_. Load this as a volume inside an _nginx-cm_ pod on path _/etc/spartaa_. Create the pod and 'ls' into the _/etc/spartaa_ directory.**
    <details><summary>show</summary><p>

    [Configmap documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap)

    _Hints: create the CM and use --dry-run=client to generate the yaml. After that add the corresponding fieds to the yaml._

        kubectl -n practice exec -it nginx-cm -- ls /etc/spartaa # should return var8 var9

27. **Create an _nginx-requests_ pod with requests _cpu=100m, memory=256Mi_ and limits _cpu=200m, memory=512Mi_.**

    <details><summary>show</summary><p>

    [Assign Resources documentation](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

    _Hint: check kubectl run options._

28. **Create a secret called _mysecret_ with values _password=mypass_ and check its yaml.**

    <details><summary>show</summary><p>

    [Secrets documentation](https://kubernetes.io/docs/concepts/configuration/secret/)

        kubectl -n practice create secret generic mysecret --from-literal=password=mypass

    **Take-away**: Secrets are base64 encoded not encrypted -> bXlwYXNz.

29. **Create an nginx pod that mounts the secret _mysecret_ in a volume on path _/etc/foo_.**

    <details><summary>show</summary><p>

    [How to use Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

    _Hint: The approach is similar to configMaps._

    **Take-away**: Secret is namespaced resource.

### Observability

30. **Get the list of nodes in JSON format and store it in a file.**

    <details><summary>show</summary><p>

        kubectl get nodes -o json > brahmaputra.json

    **Take-away**: Check what other output formats are available.

31. **Get CPU/memory utilization for nodes.**

     <details><summary>show</summary><p>

        kubectl top nodes

    **Take-away**: _kubectl top pods --all-namespaces=true_ can be used for pods.

32. **Create an nginx pod with a liveness probe that just runs the command _ls_. Check probe status.**

    <details><summary>show</summary><p>

    [Configure Liveness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-liveness-command)

        kubectl -n practice run nginx-live --image=nginx --dry-run=client -o yaml > pod_liveness.yaml
        vi pod_liveness.yaml

    ```YAML
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
        run: nginx-live
      name: nginx-live
    spec:
      containers:
      - image: nginx
        name: nginx-live
        resources: {}
        livenessProbe: # add
          exec:        # add
            command:   # add
            - ls       # add
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    status: {}
    ```

        kubectl -n practice apply -f pod.yaml
        kubectl -n practice describe pod nginx-live

    **Take-away**: The kubelet uses liveness probes to know when to restart a container.

33. **Create an nginx pod (that includes port 80) with an HTTP readinessProbe on path '/' on port 80.**

    <details><summary>show</summary><p>

    [Configure Liveness, Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

        kubectl -n practice run nginx-ready --image=nginx --dry-run=client -o yaml --port=80 > pod_readiness.yaml

    Find what needs to be added to the file from the above documentation.

    **Take-away**: K8s uses readiness probes to decide when the container is available for accepting traffic.

34. **Use JSON PATH query to retrieve the _osImages_ of all the nodes.**

    <details><summary>show</summary><p>

        kubectl get nodes -o jsonpath="{.items[*].status.nodeInfo.osImage}"

    You should see something like that: Container Linux by CoreOS 2303.3.0 (Rhyolite)

    **Take-away**: Try to understand the construct of the query.

### Storage

35. **Create a PersistentVolume of 1Gi, called 'myvolume-practice'. Make it have accessMode of 'ReadWriteOnce' and 'ReadWriteMany', storageClassName 'normal', mounted on hostPath '/etc/foo'. List all PersistentVolume**

    <details><summary>show</summary><p>

    [PersistentVolume documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume)

    Use the following output:

    ```YAML
    kind: PersistentVolume
    apiVersion: v1
    metadata:
      name: myvolume-practice
    spec:
      storageClassName: normal
      capacity:
        storage: 1Gi
      accessModes:
        - ReadWriteOnce
        - ReadWriteMany
      hostPath:
        path: /etc/foo
    ```

        kubectl get pv # status should be Available

    **Take-away**: PersistentVolume is not namespaced resource.

36. **Create a PersistentVolumeClaim called 'mypvc-practice' requesting 400Mi with accessMode of 'ReadWriteOnce' and storageClassName of normal. Check the status of the PersistenVolume.**

    <details><summary>show</summary><p>

    Use [PersistentVolumeClaim documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim) to figure out the correct yaml.

    The status of the PersistentVolume myvolume-practice should be bound.

    **Take-away**: PersistentVolumeClaim is namespaced resource.

37. **Create a busybox pod with command 'sleep 3600'. Mount the PersistentVolumeClaim mypvc-practice to '/etc/foo'. Connect to the 'busybox' pod, and copy the '/etc/passwd' file to '/etc/foo/passwd'.**

    <details><summary>show</summary><p>

    Use [PersistentVolume documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-pod)

38. **Create a second pod which is identical with the one you just created (use different name). Connect to it and verify that '/etc/foo' contains the 'passwd' file. Delete the pods.**

    <details><summary>show</summary><p>

    Nope

### Security

39. **Create _busybox-user_ pod that runs sleep for 1 hour and has user ID set to 101. Check the UID from within the container.**

    <details><summary>show</summary><p>

    [Security Context documentation](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-container)

        kubectl -n practice run busybox-user --image=busybox --command sleep 3600 --dry-run=client -o yaml > pod.yaml
        vi pod.yaml

    ```YAML
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
        run: busybox-user
      name: busybox-user
    spec:
      securityContext:   # Add
        runAsUser: 101   # Add
      containers:
      - command:
        - sleep
        - "3600"
        image: busybox
        name: busybox-user
        resources: {}
      dnsPolicy: ClusterFirst
      restartPolicy: Always
    status: {}
    ```

        kubectl -n practice exec -it busybox-user -- id -u # should return 101

40. **Create the YAML for an nginx pod that has capabilities "NET_ADMIN" and "SYS_TIME".**

    <details><summary>show</summary><p>

    [Security Context documentation](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

41. **Create a new service account with the name _pvviewer-practice_. Grant this Service account access to list all PersistentVolumes in the cluster by creating an appropriate cluster role called _pvviewer-role-practice_ and ClusterRoleBinding called _pvviewer-role-binding-practice_**

    <details><summary>show</summary><p>

    [RBAC documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

        kubectl -n practice create serviceaccount pvviewer-practice
        kubectl create clusterrole pvviewer-role-practice --resource=pv --verb=list
        kubectl create clusterrolebinding pvviewer-role-binding-practice --clusterrole=pvviewer-role-practice --serviceaccount=practice:pvviewer-practice

    **Take-away**: Read the documentation and try to understand more for Role, ClusterRole, RoleBinding and ClusterRoleBinding.

### Networking

42. **Create a pod with image nginx called _nginx-1_ and expose its port 80.**

    <details><summary>show</summary><p>

        kubectl -n practice run nginx-1 --image=nginx --port=80 --expose

    Check that both pod and service are created.

    **Take-away**: --expose can be really handy for basic services.

43. **Get service's ClusterIP, create a temp _busybox-1_ pod and 'hit' that IP with wget.**

    <details><summary>show</summary><p>

        kubectl -n practice get svc nginx-1
        kubectl -n practice run busybox-1 --rm --image=busybox -it -- sh
        / # wget -O- $CLUSTER_IP:80

    **Take-away**: ClusterIP is only reachable from within the cluster.

44. **Convert the ClusterIP to NodePort for the same service and find the NodePort. Hit the service(create temp busybox pod) using Node's IP and Port.**

    <details><summary>show</summary><p>

        kubectl -n practice edit svc nginx-1 # ClusterIP -> NodePort
        kubectl -n practice describe svc nginx-1 # find NodePort

    Create temp busybox pod and execute the following:

        / # wget -O- $NODE_IP:$NODE_PORT

45. **Create an _nginx-last_ deployment of 2 replicas, expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: granted' can access the deployment.**

    <details><summary>show</summary><p>

    Create the deployment and expose it.

    [Network Policy documentation](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

    ```YAML
    kind: NetworkPolicy
    apiVersion: networking.k8s.io/v1
    metadata:
      name: access-nginx
    spec:
      podSelector:
        matchLabels:
          app: nginx-last # selector for the pods
      ingress:            # allow ingress traffic
      - from:
        - podSelector:    # from pods
            matchLabels:  # with this label
              access: granted
    ```

    Apply the above yaml and test with temporary busybox pods.

        kubectl -n practice run busybox --image=busybox --rm -it -- wget -O- http://nginx-last:80 --timeout 2 # This should fail
        kubectl -n practice run busybox --image=busybox --rm -it --labels=access=granted -- wget -O- http://nginx-last:80 --timeout 2 # This should work

    **Take-away**: With NetworkPolicy you can configure how groups of pods are allowed to communicate with each other.

### Challenging

46. **Create an nginx pod called _nginx-resolver_ using image _nginx_, expose it internally with a service called _nginx-resolver-service_. Test that you are able to look up the service and pod names from within the cluster. Use the image: busybox:1.28 for dns lookup.**
    <details><summary>show</summary><p>

    You need to figure it out alone :)

47. **List the InternalIP of all nodes of the cluster.**

    <details><summary>show</summary><p>

    _Hint: use jsonpath_

48. **Taint one worker node to be Unschedulable. Once done, create a pod called _dev-redis_ with image _redis:alpine_ to ensure workloads are not scheduled to the tainted node. Finally, create a new pod called _prod-redis_ with image _redis:alpine_ with toleration to be scheduled on the tainted node.**

49. **Create a Pod called _redis-storage_ with image _redis:alpine_ with a Volume of type emptyDir that lasts for the life of the Pod. Use volumeMount with mountPath = /data/redis.**

50. **Create a new deployment called _nginx-deploy_, with image _nginx:1.16_ and 1 replica. Record the version. Next upgrade the deployment to version 1.17 using rolling update. Make sure that the version upgrade is recorded in the resource annotation.**

## Cleanup

    kubectl delete ns practice

## Other repos

1. If you are interested in **_Linux_** have a look here: https://github.com/StenlyTU/LFCS-official

2. Linux Foundation LFCE knowledge base: coming soon

:checkered_flag: If you liked the repo please star it :star:
