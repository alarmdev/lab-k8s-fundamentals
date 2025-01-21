For the exercises you will be doing, you will be using the `kubectl` command line program to interact with Kubernetes. This is provided for you via the interactive terminal session accessible through the **Terminal** tab, here in the workshop environment. You do not need to install anything on your own computer. You will be doing everything here through your web browser. There is no need to login as you are already connected to the Kubernetes cluster you will be using.

The workshop environment also provides you with a web based view into the Kubernetes cluster. This is available through the **Console** tab of the workshop environment. This is included so you can visually see the results of what you do in the exercises, but the exercises do not depend on it.

Before continuing, verify that the `kubectl` command runs and the workshop environment is also functioning. To do this run:

```execute
kubectl version
```

Did you type the command in yourself? If you did, click on the command here instead and you will find that it is executed for you. You can click on any command block here in the workshop notes which has the <span class="fas fa-running"></span> icon shown to the right of it, and it will be copied to the interactive terminal and run for you. Other action blocks may also be used in this workshop, showing different icons, you can also click on these to trigger the action described.

When run, you should see output similar to:

```
Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.10", GitCommit:"a7a32748b5c60445c4c7ee904caf01b91f2dbb71", GitTreeState:"clean", BuildDate:"2022-02-16T11:24:04Z", GoVersion:"go1.16.14", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.1", GitCommit:"5e58841cce77d4bc13713ad2b91fa0d961e69192", GitTreeState:"clean", BuildDate:"2021-05-21T23:01:33Z", GoVersion:"go1.16.4", Compiler:"gc", Platform:"linux/amd64"}
```




Task
Take a backup of the etcd cluster and save it to /opt/etcd-backup.db.

Q. 2

Task
Create a Pod called redis-storage with image: redis:alpine with a Volume of type emptyDir that lasts for the life of the Pod.

Specs on the below.

Solution
Use the command kubectl run and create a pod definition file for redis-storage pod and add volume.

Alternatively, run the command:

kubectl run redis-storage --image=redis:alpine --dry-run=client -oyaml > redis-storage.yaml
and add volume emptyDir in it.

Solution manifest file to create a pod redis-storage as follows:

---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis-storage
  name: redis-storage
spec:
  containers:
  - image: redis:alpine
    name: redis-storage
    volumeMounts:
    - mountPath: /data/redis
      name: temp-volume
  volumes:
  - name: temp-volume
    emptyDir: {}

Details

Pod named 'redis-storage' created


Pod 'redis-storage' uses Volume type of emptyDir


Pod 'redis-storage' uses volumeMount with mountPath = /data/redis

Q. 3

Task
Create a new pod called super-user-pod with image busybox:1.28. Allow the pod to be able to set system_time.

The container should sleep for 4800 seconds.

Solution
Solution manifest file to create a pod super-user-pod as follows:

---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: super-user-pod
  name: super-user-pod
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: busybox:1.28
    name: super-user-pod
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
  dnsPolicy: ClusterFirst
  restartPolicy: Always

Details

Pod: super-user-pod


Container Image: busybox:1.28


Is SYS_TIME capability set for the container?

Q. 4

Task
A pod definition file is created at /root/CKA/use-pv.yaml. Make use of this manifest file and mount the persistent volume called pv-1. Ensure the pod is running and the PV is bound.

mountPath: /data

persistentVolumeClaim Name: my-pvc

Solution
Add a persistentVolumeClaim definition to pod definition file.

Solution manifest file to create a pvc my-pvc as follows:

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
       storage: 10Mi

And then, update the pod definition file as follows:

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: use-pv
  name: use-pv
spec:
  containers:
  - image: nginx
    name: use-pv
    volumeMounts:
    - mountPath: "/data"
      name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: my-pvc

Finally, create the pod by running:
kubectl create -f /root/CKA/use-pv.yaml

Details

persistentVolume Claim configured correctly


pod using the correct mountPath


pod using the persistent volume claim?

Q. 5

Task
Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Next upgrade the deployment to version 1.17 using rolling update.

Solution
Explore the --record option while creating the deployment while working with the deployment definition file. Then make use of the kubectl apply command to create or update the deployment.

To create a deployment definition file nginx-deploy:

$ kubectl create deployment nginx-deploy --image=nginx:1.16 --dry-run=client -o yaml > deploy.yaml

To create a resource from definition file and to record:

$ kubectl apply -f deploy.yaml --record

To view the history of deployment nginx-deploy:

$ kubectl rollout history deployment nginx-deploy

To upgrade the image to next given version:

$ kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record

To view the history of deployment nginx-deploy:

$ kubectl rollout history deployment nginx-deploy

Details

Deployment : nginx-deploy. Image: nginx:1.16


Image: nginx:1.16


Task: Upgrade the version of the deployment to 1:17


Task: Record the changes for the image upgrade

Q. 6

Task
Create a new user called john. Grant him access to the cluster. John should have permission to create, list, get, update and delete pods in the development namespace . The private key exists in the location: /root/CKA/john.key and csr at /root/CKA/john.csr.

Important Note: As of kubernetes 1.19, the CertificateSigningRequest object expects a signerName.

Please refer the documentation to see an example. The documentation tab is available at the top right of terminal.

Solution
Solution manifest file to create a CSR as follows:

---
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  signerName: kubernetes.io/kube-apiserver-client
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUt2Um1tQ0h2ZjBrTHNldlF3aWVKSzcrVVdRck04ZGtkdzkyYUJTdG1uUVNhMGFPCjV3c3cwbVZyNkNjcEJFRmVreHk5NUVydkgyTHhqQTNiSHVsTVVub2ZkUU9rbjYra1NNY2o3TzdWYlBld2k2OEIKa3JoM2prRFNuZGFvV1NPWXBKOFg1WUZ5c2ZvNUpxby82YU92czFGcEc3bm5SMG1JYWpySTlNVVFEdTVncGw4bgpjakY0TG4vQ3NEb3o3QXNadEgwcVpwc0dXYVpURTBKOWNrQmswZWhiV2tMeDJUK3pEYzlmaDVIMjZsSE4zbHM4CktiSlRuSnY3WDFsNndCeTN5WUFUSXRNclpUR28wZ2c1QS9uREZ4SXdHcXNlMTdLZDRaa1k3RDJIZ3R4UytkMEMKMTNBeHNVdzQyWVZ6ZzhkYXJzVGRMZzcxQ2NaanRxdS9YSmlyQmxVQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQ1VKTnNMelBKczB2czlGTTVpUzJ0akMyaVYvdXptcmwxTGNUTStsbXpSODNsS09uL0NoMTZlClNLNHplRlFtbGF0c0hCOGZBU2ZhQnRaOUJ2UnVlMUZnbHk1b2VuTk5LaW9FMnc3TUx1a0oyODBWRWFxUjN2SSsKNzRiNnduNkhYclJsYVhaM25VMTFQVTlsT3RBSGxQeDNYVWpCVk5QaGhlUlBmR3p3TTRselZuQW5mNm96bEtxSgpvT3RORStlZ2FYWDdvc3BvZmdWZWVqc25Yd0RjZ05pSFFTbDgzSkljUCtjOVBHMDJtNyt0NmpJU3VoRllTVjZtCmlqblNucHBKZWhFUGxPMkFNcmJzU0VpaFB1N294Wm9iZDFtdWF4bWtVa0NoSzZLeGV0RjVEdWhRMi80NEMvSDIKOWk1bnpMMlRST3RndGRJZjAveUF5N05COHlOY3FPR0QKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  usages:
  - digital signature
  - key encipherment
  - client auth

To approve this certificate, run: kubectl certificate approve john-developer

Next, create a role developer and rolebinding developer-role-binding, run the command:

$ kubectl create role developer --resource=pods --verb=create,list,get,update,delete --namespace=development

$ kubectl create rolebinding developer-role-binding --role=developer --user=john --namespace=development

To verify the permission from kubectl utility tool:

$ kubectl auth can-i update pods --as=john --namespace=development

Details

CSR: john-developer Status:Approved


Role Name: developer, namespace: development, Resource: Pods


Access: User 'john' has appropriate permissions

Q. 7

Task
Create a nginx pod called nginx-resolver using image nginx, expose it internally with a service called nginx-resolver-service. Test that you are able to look up the service and pod names from within the cluster. Use the image: busybox:1.28 for dns lookup. Record results in /root/CKA/nginx.svc and /root/CKA/nginx.pod

Solution
Use the command kubectl run and create a nginx pod and busybox pod. Resolve it, nginx service and its pod name from busybox pod.

To create a pod nginx-resolver and expose it internally:

kubectl run nginx-resolver --image=nginx
kubectl expose pod nginx-resolver --name=nginx-resolver-service --port=80 --target-port=80 --type=ClusterIP

To create a pod test-nslookup. Test that you are able to look up the service and pod names from within the cluster:

kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service > /root/CKA/nginx.svc

Get the IP of the nginx-resolver pod and replace the dots(.) with hyphon(-) which will be used below.

kubectl get pod nginx-resolver -o wide
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup <P-O-D-I-P.default.pod> > /root/CKA/nginx.pod

Details

Pod: nginx-resolver created


Service DNS Resolution recorded correctly


Pod DNS resolution recorded correctly

Q. 8

Task
Create a static pod on node01 called nginx-critical with image nginx and make sure that it is recreated/restarted automatically in case of a failure.

Use /etc/kubernetes/manifests as the Static Pod path for example.

Solution
To create a static pod called nginx-critical by using below command:

kubectl run nginx-critical --image=nginx --dry-run=client -o yaml > static.yaml

Copy the contents of this file or use scp command to transfer this file from controlplane to node01 node.

root@controlplane:~# scp static.yaml node01:/root/

To know the IP Address of the node01 node:

root@controlplane:~# kubectl get nodes -o wide

# Perform SSH
root@controlplane:~# ssh node01
OR
root@controlplane:~# ssh <IP of node01>

On node01 node:

Check if static pod directory is present which is /etc/kubernetes/manifests, if it's not present then create it.

root@node01:~# mkdir -p /etc/kubernetes/manifests

Add that complete path to the staticPodPath field in the kubelet config.yaml file.

root@node01:~# vi /var/lib/kubelet/config.yaml

now, move/copy the static.yaml to path /etc/kubernetes/manifests/.

root@node01:~# cp /root/static.yaml /etc/kubernetes/manifests/

Go back to the controlplane node and check the status of static pod:

root@node01:~# exit
logout
root@controlplane:~# kubectl get pods 

Details

static pod configured under /etc/kubernetes/manifests ?


Pod nginx-critical-node01 is up and running




The version of Kubernetes being used may be different to the version shown here.
