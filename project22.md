# DEPLOYING APPLICATIONS INTO KUBERNETES CLUSTER
## INTRODUCTION

## STEP 1: Creating A Pod For The Nginx Application

- Creating nginx pod by applying the manifest file:`kubectl apply -f nginx-pod.yaml`

**nginx-pod.yaml manifest file**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project22/13-creating%20nginx-pod%20file.png)

- Running the following commands to inspect the the setup:
```
$ kubectl get pod nginx-pod --show-labels

$ kubectl get pod nginx-pod -o wide
```
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project22/14-applying%20the%20config.png)

## STEP 2: Accessing The Nginx Application Through A Browser

- First of all, Let's try accessing the Nginx Pod through its IP address from within the Kubernetes cluster. To do this an image that already has curl software installed is needed.
- Running the **kubectl** command to connect inside the container:`$ kubectl run curl --image=dareyregistry/curl -i --tty`
- Running **curl** and pointing it to the IP address of the Nginx Pod:`**$ curl -v 172.50.202.214:80**`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project22/15-connecting%20to%20nginx%20pod%20from%20another%20pod.png)

- Now Let's try and access the application through the browser, but first we need to create a service for the Nginx pod.
- Creating service for the nginx pod by applying the manifest file:`$ kubectl apply -f nginx-service.yaml`

**nginx-service.yaml manifest file**
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx-pod  
spec:
  containers:
  - image: nginx:latest
    name: nginx-pod
    ports:
    - containerPort: 80
      protocol: TCP
```
- Running the following commands to inspect the the setup:
```
$ kubectl get service nginx-service -o wide

$ kubectl get svc nginx-service --show-labels
```

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project22/16-creating%20nginx%20service.png)

- Since the type of service created for the Nginx pod is a ClusterIP which cannot be accessed externally, we can do port-forwarding in order to bind the machine's port to the ClusterIP service port, i.e, tunnelling traffic through the machine's port number to the port number of the nginx-service: `$ kubectl port-forward svc/nginx-service 8089:80`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project22/17-port-forwading%20to%20nginx%20service.png)

- Accessing the Nginx application from the browser:`http://localhost:8089`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project22/18-accessing%20nginx%20from%20browser.png)

- Another way of accessing the Nginx app through browser is the use of **NodePort** which is a type of service that exposes the service on a static port on the node’s IP address and they range from **30000-32767** by default.
- Exposing the Nginx service to be accessible to the browser by adding NodePort as a type of service in the nginx-service.yml manifest file:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx-pod
  ports:
    - protocol: TCP
      port: 80
      nodePort: 30080
```

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project22/19-creating%20nodeport%20service.png)

- Accessing the nginx application from the browser with the the value of the nodeport **30080** which is a port on the node in which the Pod is scheduled to run:`http://localhost:30080`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project22/20-accessing%20on%20browser.png) 

## STEP 4: Deploying Tooling Application With Kubernetes

## STEP 5: Creating A Replica Set
- The replicaSet object helps to maintain a stable set of Pod replicas running at any given time to achieve availability in case one or two pods dies.
- Deleting the nginx-pod:`kubectl delete pod nginx-pod`
- Creating the replicaSet manifest file and applying it:`$ kubectl apply -f rs.yaml`

**rs.yaml manifest file**
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    app: nginx-pod
  template:
    metadata:
      name: nginx-pod
      labels:
         app: nginx-pod
    spec:
      containers:
      - image: nginx:latest
        name: nginx-pod
        ports:
        - containerPort: 80
          protocol: TCP
```
- Inspecting the setup:
```
$ kubectl get pods

$ kubectl get rs -o wide
```
- Deleting one of the pods will cause another one to be scheduled and set to run:`$ kubectl delete pod nginx-rs-7tdmr`

**Another pod scheduled**

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project22/21-creating%20rs.png)

- Two ways pods can be scaled: **Imperative** and **Declarative**
- Imperative method is by running a command on the CLI: `$ kubectl scale --replicas 5 replicaset nginx-rs`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project22/22-imperative%20command.png)

- Declarative method is done by editing the **rs.yaml** manifest and changing to the desired number of replicas and applying the update

## STEP 6: Using AWS Load Balancer To Access The Nginx Application

- Another type of service used in Kubernetes is the **loadbalancer** which does not only create a Service object but also provisions a real external Load Balancer- Elastic Load Balancer. But this can be possible using EKS.
- Editing the nginx-service.yaml file and adding **loadbalancer** in the appropriate section and applying it:`$ kubectl apply -f nginx-service.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    tier: frontend
  ports:
    - protocol: TCP
      port: 80 # This is the port the Loadbalancer is listening at
      targetPort: 80 # This is the port the container is listening at
```

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project22/26-lb%20created.png)

- Inspecting the setup:`$ kubectl get service nginx-service -o yaml`

![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project22/25-creating%20lb%20service.png)
![](https://github.com/somex6/Darey.io-Projects/blob/main/img/project22/25-creating%20lb%20service-2.png)

- Accessing the Nginx Application with the load balancer's address on the browser:`http://.......`
