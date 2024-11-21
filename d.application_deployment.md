# Application Deployment  (20 %)

|[x] | Use Kubernetes primitives to implement common deployment strategies (e.g. blue/green or canary)|
|[x] | Understand Deployments and how to perform rolling updates|
|[x] | Use the Helm package manager to deploy existing packages|
|[x] | Kustomize|

## Understand Deployments and how to perform rolling updates

kubernetes.io > Documentation > Concepts > Workloads > Workload Resources > [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment)<sup>Kubernetes Documentation</sup>

- [Practical Examples for 3 Advanced Kubernetes deployment strategies](https://teckbootcamps.com/practical-examples-for-3-advanced-kubernetes-deployment-strategies/)<sup>Blog</sup>


### Create a deployment with image nginx:1.18.0, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)

<details><summary>show</summary>
<p>

```bash
kubectl create deployment nginx  --image=nginx:1.18.0  --dry-run=client -o yaml > deploy.yaml
vi deploy.yaml
# change the replicas field from 1 to 2
# add this section to the container spec and save the deploy.yaml file
# ports:
#   - containerPort: 80
kubectl apply -f deploy.yaml
```

or, do something like:

```bash
kubectl create deployment nginx  --image=nginx:1.18.0  --dry-run=client -o yaml | sed 's/replicas: 1/replicas: 2/g'  | sed 's/image: nginx:1.18.0/image: nginx:1.18.0\n        ports:\n        - containerPort: 80/g' | kubectl apply -f -
```

or,
```bash
kubectl create deploy nginx --image=nginx:1.18.0 --replicas=2 --port=80
```

</p>
</details>

### View the YAML of this deployment

<details><summary>show</summary>
<p>

```bash
kubectl get deploy nginx -o yaml
```

</p>
</details>

### View the YAML of the replica set that was created by this deployment

<details><summary>show</summary>
<p>

```bash
kubectl describe deploy nginx # you'll see the name of the replica set on the Events section and in the 'NewReplicaSet' property
# OR you can find rs directly by:
kubectl get rs -l run=nginx # if you created deployment by 'run' command
kubectl get rs -l app=nginx # if you created deployment by 'create' command
# you could also just do kubectl get rs
kubectl get rs nginx-7bf7478b77 -o yaml
```

</p>
</details>

### Get the YAML for one of the pods

<details><summary>show</summary>
<p>

```bash
kubectl get po # get all the pods
# OR you can find pods directly by:
kubectl get po -l run=nginx # if you created deployment by 'run' command
kubectl get po -l app=nginx # if you created deployment by 'create' command
kubectl get po nginx-7bf7478b77-gjzp8 -o yaml
```

</p>
</details>

### Check how the deployment rollout is going

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
```

</p>
</details>

### Update the nginx image to nginx:1.19.8

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.19.8
# alternatively...
kubectl edit deploy nginx # change the .spec.template.spec.containers[0].image
```

The syntax of the 'kubectl set image' command is `kubectl set image (-f FILENAME | TYPE NAME) CONTAINER_NAME_1=CONTAINER_IMAGE_1 ... CONTAINER_NAME_N=CONTAINER_IMAGE_N [options]`

</p>
</details>

### Check the rollout history and confirm that the replicas are OK

<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx
kubectl get deploy nginx
kubectl get rs # check that a new replica set has been created
kubectl get po
```

</p>
</details>

### Undo the latest rollout and verify that new pods have the old image (nginx:1.18.0)

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx
# wait a bit
kubectl get po # select one 'Running' Pod
kubectl describe po nginx-5ff4457d65-nslcl | grep -i image # should be nginx:1.18.0
```

</p>
</details>

### Do an on purpose update of the deployment with a wrong image nginx:1.91

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.91
# or
kubectl edit deploy nginx
# change the image to nginx:1.91
# vim tip: type (without quotes) '/image' and Enter, to navigate quickly
```

</p>
</details>

### Verify that something's wrong with the rollout

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
# or
kubectl get po # you'll see 'ErrImagePull' or 'ImagePullBackOff'
```

</p>
</details>


### Return the deployment to the second revision (number 2) and verify the image is nginx:1.19.8

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx --to-revision=2
kubectl describe deploy nginx | grep Image:
kubectl rollout status deploy nginx # Everything should be OK
```

</p>
</details>

### Check the details of the fourth revision (number 4)

<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx --revision=4 # You'll also see the wrong image displayed here
```

</p>
</details>

### Scale the deployment to 5 replicas

<details><summary>show</summary>
<p>

```bash
kubectl scale deploy nginx --replicas=5
kubectl get po
kubectl describe deploy nginx
```

</p>
</details>

### Autoscale the deployment, pods between 5 and 10, targetting CPU utilization at 80%

<details><summary>show</summary>
<p>

```bash
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
# view the horizontalpodautoscalers.autoscaling for nginx
kubectl get hpa nginx
```

</p>
</details>

### Pause the rollout of the deployment

<details><summary>show</summary>
<p>

```bash
kubectl rollout pause deploy nginx
```

</p>
</details>

### Update the image to nginx:1.19.9 and check that there's nothing going on, since we paused the rollout

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.19.9
# or
kubectl edit deploy nginx
# change the image to nginx:1.19.9
kubectl rollout history deploy nginx # no new revision
```

</p>
</details>

### Resume the rollout and check that the nginx:1.19.9 image has been applied

<details><summary>show</summary>
<p>

```bash
kubectl rollout resume deploy nginx
kubectl rollout history deploy nginx
kubectl rollout history deploy nginx --revision=6 # insert the number of your latest revision
```

</p>
</details>

### Delete the deployment and the horizontal pod autoscaler you created

<details><summary>show</summary>
<p>

```bash
kubectl delete deploy nginx
kubectl delete hpa nginx

#Or
kubectl delete deploy/nginx hpa/nginx
```
</p>
</details>

### Implement canary deployment by running two instances of nginx marked as version=v1 and version=v2 so that the load is balanced at 75%-25% ratio

<details><summary>show</summary>
<p>

Deploy 3 replicas of v1:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v1
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: v1
  template:
    metadata:
      labels:
        app: my-app
        version: v1
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
      initContainers:
      - name: install
        image: busybox:1.28
        command:
        - /bin/sh
        - -c
        - "echo version-1 > /work-dir/index.html"
        volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
      volumes:
      - name: workdir
        emptyDir: {}
```

Create the service:
```
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
  labels:
    app: my-app
spec:
  type: ClusterIP
  ports:
  - name: http
    port: 80
    targetPort: 80
  selector:
    app: my-app
```

Test if the deployment was successful from within a Pod:
```
# run a wget to the Service my-app-svc
kubectl run -it --rm --restart=Never busybox --image=gcr.io/google-containers/busybox --command -- wget -qO- my-app-svc

version-1
```

Deploy 1 replica of v2:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v2
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: v2
  template:
    metadata:
      labels:
        app: my-app
        version: v2
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
      initContainers:
      - name: install
        image: busybox:1.28
        command:
        - /bin/sh
        - -c
        - "echo version-2 > /work-dir/index.html"
        volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
      volumes:
      - name: workdir
        emptyDir: {}
```

Observe that calling the ip exposed by the service the requests are load balanced across the two versions:
```
# run a busyBox pod that will make a wget call to the service my-app-svc and print out the version of the pod it reached.
kubectl run -it --rm --restart=Never busybox --image=gcr.io/google-containers/busybox -- /bin/sh -c 'while sleep 1; do wget -qO- my-app-svc; done'

version-1
version-1
version-1
version-2
version-2
version-1
```

If the v2 is stable, scale it up to 4 replicas and shoutdown the v1:
```
kubectl scale --replicas=4 deploy my-app-v2
kubectl delete deploy my-app-v1
while sleep 0.1; do curl $(kubectl get svc my-app-svc -o jsonpath="{.spec.clusterIP}"); done
version-2
version-2
version-2
version-2
version-2
version-2
```

</p>
</details>

## Use the Helm package manager to deploy existing packages

### Creating a basic Helm chart

<details><summary>show</summary>
<p>

```bash
helm create chart-test ## this would create a helm 
```

</p>
</details>

### Running a Helm chart

<details><summary>show</summary>
<p>

```bash
helm install -f myvalues.yaml myredis ./redis
```

</p>
</details>

### Find pending Helm deployments on all namespaces

<details><summary>show</summary>
<p>

```bash
helm list --pending -A
```

</p>
</details>

### Uninstall a Helm release

<details><summary>show</summary>
<p>

```bash
helm uninstall -n namespace release_name
```

</p>
</details>

### Upgrading a Helm chart

<details><summary>show</summary>
<p>

```bash
helm upgrade -f myvalues.yaml -f override.yaml redis ./redis
```

</p>
</details>

### Using Helm repo

<details><summary>show</summary>
<p>

Add, list, remove, update and index chart repos

```bash
helm repo add [NAME] [URL]  [flags]

helm repo list / helm repo ls

helm repo remove [REPO1] [flags]

helm repo update / helm repo up

helm repo update [REPO1] [flags]

helm repo index [DIR] [flags]
```

</p>
</details>

### Download a Helm chart from a repository 

<details><summary>show</summary>
<p>

```bash
helm pull [chart URL | repo/chartname] [...] [flags] ## this would download a helm, not install 
helm pull --untar [rep/chartname] # untar the chart after downloading it 
```

</p>
</details>

### Add the Bitnami repo at https://charts.bitnami.com/bitnami to Helm
<details><summary>show</summary>
<p>
    
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```
  
</p>
</details>

### Write the contents of the values.yaml file of the `bitnami/node` chart to standard output
<details><summary>show</summary>
<p>
    
```bash
helm show values bitnami/node
```
  
</p>
</details>

### Install the `bitnami/node` chart setting the number of replicas to 5
<details><summary>show</summary>
<p>

To achieve this, we need two key pieces of information:
- The name of the attribute in values.yaml which controls replica count
- A simple way to set the value of this attribute during installation

To identify the name of the attribute in the values.yaml file, we could get all the values, as in the previous task, and then grep to find attributes matching the pattern `replica`
```bash
helm show values bitnami/node | grep -i replica
```
which returns
```bash
## @param replicaCount Specify the number of replicas for the application
replicaCount: 1
```
 
We can use the `--set` argument during installation to override attribute values. Hence, to set the replica count to 5, we need to run
```bash
helm install mynode bitnami/node --set replicaCount=5
```

</p>
</details>
