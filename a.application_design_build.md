# Application Design and Build (20%)

The first domain in the exam is Application Design and Build having a 20% weight. Before moving on to the next domain make sure that you're fluent with the following. 

- [x] Define, build, and modify container images
- [x] Understand Jobs and CronJobs
- [x] Understand multi-container Pod design patterns (e.g. sidecar, init, and others)
- [x] Utilize persistent and ephemeral

## Set the context and the namespace.
A "Context" is a combination of a cluster, user, and namespace. It is a way to specify the cluster you want to interact with, the user or authentication credentials you want to use, and the default namespace for that user. Setting a context is important because it helps you manage multiple Kubernetes clusters and switch between them easily.

**Use Cases:**  Multi-cluster Management, User and Authentication, isolate and organize resources, simplifies command-line operations.

**Hint:** Don't change the namespace, change the context as per the question. If there is a namespace add -n `namespace` to lessen the risk of error. Set context before each question using the kubectl config command, and switch between contexts using kubectl config use-context. 


## Define, build, and modify container images

- [Containerd basic Commands and Usage](https://teckbootcamps.com/containerd-basic-commands-and-usage/)<sup>Blog</sup>

**1-)** Create an nginx pod using YAML that runs the command "env" and display the logs.

<span style="color:green;">
<details closed>
  <summary>
  Answer
  </summary>

```bash
k run nginx --image=nginx --restart=Never --dry-run=client -oyaml -- env >nginx.yaml
k logs nginx

```

![](assets/20231214124611.png)

</details>
</span>

<br>


**2-)** Create a busybox pod named bbox pod using YAML that runs the command "hello world", display the date and then sleep for 3600 seconds. Lastly display the logs.

<span style="color:green;">
<details closed>
  <summary>
  Answer
  </summary>

```bash
k run bbox --image=busybox --restart=Never --dry-run=client -oyaml -- /bin/sh -c "echo 'hello world'; date;sleep 3600 ">bbox.ya
ml
```

![](assets/20231214125455.png)

![](assets/20231214125019.png)

</details>
</span>

<br>

**3-)**
Create a Kubernetes Pod named "loop" in the "true-b" namespace. The Pod runs a container using the "busybox" image, with a looping script echoing "Hello $i times" for values of i from 1 to 10.

<span style="color:green;">
<details closed>
  <summary>
  Answer
  </summary>

```bash
k create ns true-b
kubectl run loop --image=busybox \
  -n true-b $do --restart=Never \
  --command --  'for i in $(seq 1 10); \
  do echo "Hello $i times"; done' > pod.yaml
```

</details>
</span>

<br>


**3-)**
Run an nginx pod named `ng` living in `mynamespace` ns. Don't restart it if it exits. Get a shell inside the container and display the environment variables.

<span style="color:green;">
<details closed>
  <summary>
  Answer
  </summary>

```bash
k config set-context --current --namespace=mynamespace
k run ng --image=nginx --restart=Never -it -- /bin/sh
env

```

</details>
</span>

<br>



```sh
docker image ls
docker image rm
docker history <imageid> OR image:tag
docker tag <current-image> <target-registry or whatever you want>
docker build -t
docker commit
docker save -o myimage.tar
docker load --input
docker inspect example-voting-app-result | grep -i -C 5 layers
```

### Best Practices for writing Dockerfiles

**1. Use a Smaller Base Image:**

- [How to Reduce the size of the Docker Image ?](https://teckbootcamps.com/how-to-reduce-the-size-of-the-docker-image/)<sup>Blog</sup>

Consider using a smaller base image (e.g., Alpine) to reduce the image size at the outset.

*WHY*: Choosing a smaller base image, like Alpine, significantly reduces the initial image size. Smaller base images contain only essential components, minimizing the overall image footprint and speeding up builds.

```bash
# Before
FROM ubuntu
```

```bash
# After (Using Alpine)
FROM alpine
```

**2. Avoid Debug Tools in Production:**

*WHY*: Install debugging tools only during development to prevent unnecessary increases in image size. Debugging tools are often unnecessary in production and can inflate image sizes significantly. Install them only during development for troubleshooting purposes.

```bash
# Before
RUN apt-get update && apt-get install -y curl vim
```

```bash
# After (Install only in development)
RUN apk add --no-cache curl vim
```

**3. Minimize Layers:**

*WHY*: Combining commands into a single RUN statement helps reduce the number of layers, optimizing the image size. Each RUN command creates a new layer in the image. Reducing layers not only simplifies the Dockerfile but also minimizes the image size and enhances build performance.

```bash
# Before
FROM alpine
RUN apk add --no-cache packageA
RUN apk add --no-cache packageB
```

```bash
# After (Combine in one RUN command)
FROM alpine
RUN apk add --no-cache packageA packageB
```

**4. Use `--no-install-recommends` on apk:**

*WHY*: Reduce image size by excluding recommended packages during installation. Recommended packages are often unnecessary for the intended use of the image. Omitting them during installation contributes to a leaner image.

```bash
# Before
RUN apk add --no-cache some-package
```

```bash
# After (with --no-install-recommends)
RUN apk add --no-cache --no-progress --purge --clean-protected some-package
```

**5. Clean Up After Install:**

*WHY*: Removing temporary files and package caches after installation helps keep the image size minimal and avoids unnecessary bloat. Installing tools for specific tasks and removing them immediately after use helps maintain a slim image without unnecessary components.

```bash
# After apk add
RUN apk add --no-cache wget \
    && rm -rf /var/cache/apk/*
```

```bash
# After wget or curl installation
RUN apk add --no-cache wget \
    && wget https://example.com/some-package.deb \
    && dpkg -i some-package.deb \
    && apk del wget
```

**6. Utilize fromlatest.io:**

*WHY*: Use fromlatest.io as an external tool to lint Dockerfiles for additional size reduction steps. External tools like fromlatest.io analyze Dockerfiles for potential improvements, helping identify additional steps to reduce image size. Regular linting contributes to ongoing optimization efforts.

**7. Implement Multi-Stage Builds:**

*WHY*: Use multi-stage builds to keep the final image focused on necessary artifacts, reducing overall image size. Multi-stage builds allow for separation of build and runtime environments. The final image contains only essential artifacts, resulting in a smaller and more efficient container.

```bash
# Before (Single-Stage)
FROM alpine
```

```bash
# After (Multi-Stage)
FROM alpine as builder
RUN build-command

FROM alpine as base
COPY --from=builder /path/to/artifact /app/
```

## Podman basics

### Create a Dockerfile to deploy an Apache HTTP Server which hosts a custom main page

<details><summary>show</summary>
<p>

```Dockerfile
FROM docker.io/httpd:2.4
RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html
```

</p>
</details>

### Build and see how many layers the image consists of

<details><summary>show</summary>
<p>

```bash
:~$ podman build -t simpleapp .
STEP 1/2: FROM httpd:2.4
STEP 2/2: RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html
COMMIT simpleapp
--> ef4b14a72d0
Successfully tagged localhost/simpleapp:latest
ef4b14a72d02ae0577eb0632d084c057777725c279e12ccf5b0c6e4ff5fd598b

:~$ podman images
REPOSITORY               TAG         IMAGE ID      CREATED        SIZE
localhost/simpleapp      latest      ef4b14a72d02  8 seconds ago  148 MB
docker.io/library/httpd  2.4         98f93cd0ec3b  7 days ago     148 MB

:~$ podman image tree localhost/simpleapp:latest
Image ID: ef4b14a72d02
Tags:     [localhost/simpleapp:latest]
Size:     147.8MB
Image Layers
├── ID: ad6562704f37 Size:  83.9MB
├── ID: c234616e1912 Size: 3.072kB
├── ID: c23a797b2d04 Size: 2.721MB
├── ID: ede2e092faf0 Size: 61.11MB
├── ID: 971c2cdf3872 Size: 3.584kB Top Layer of: [docker.io/library/httpd:2.4]
└── ID: 61644e82ef1f Size: 6.144kB Top Layer of: [localhost/simpleapp:latest]
```

</p>
</details>

### Run the image locally, inspect its status and logs, finally test that it responds as expected

<details><summary>show</summary>
<p>

```bash
:~$ podman run -d --name test -p 8080:80 localhost/simpleapp
2f3d7d613ea6ba19703811d30704d4025123c7302ff6fa295affc9bd30e532f8

:~$ podman ps
CONTAINER ID  IMAGE                       COMMAND           CREATED        STATUS            PORTS                 NAMES
2f3d7d613ea6  localhost/simpleapp:latest  httpd-foreground  5 seconds ago  Up 6 seconds ago  0.0.0.0:8080->80/tcp  test

:~$ podman logs test
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.0.2.100. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.0.2.100. Set the 'ServerName' directive globally to suppress this message
[Sat Jun 04 16:15:38.071377 2022] [mpm_event:notice] [pid 1:tid 139756978220352] AH00489: Apache/2.4.53 (Unix) configured -- resuming normal operations
[Sat Jun 04 16:15:38.073570 2022] [core:notice] [pid 1:tid 139756978220352] AH00094: Command line: 'httpd -D FOREGROUND'

:~$ curl 0.0.0.0:8080
Hello, World!
```

</p>
</details>

### Run a command inside the pod to print out the index.html file

<details><summary>show</summary>
<p>

```bash
:~$ podman exec -it test cat /usr/local/apache2/htdocs/index.html
Hello, World!
```
</p>
</details>

### Tag the image with ip and port of a private local registry and then push the image to this registry

<details><summary>show</summary>
<p>

> Note: Some small distributions of Kubernetes (such as [microk8s](https://microk8s.io/docs/registry-built-in)) have a built-in registry you can use for this exercise. If this is not your case, you'll have to setup it on your own.

```bash
:~$ podman tag localhost/simpleapp $registry_ip:5000/simpleapp

:~$ podman push $registry_ip:5000/simpleapp
```

</p>
</details>

 Verify that the registry contains the pushed image and that you can pull it

<details><summary>show</summary>
<p>

```bash
:~$ curl http://$registry_ip:5000/v2/_catalog
{"repositories":["simpleapp"]}

# remove the image already present
:~$ podman rmi $registry_ip:5000/simpleapp

:~$ podman pull $registry_ip:5000/simpleapp
Trying to pull 10.152.183.13:5000/simpleapp:latest...
Getting image source signatures
Copying blob 643ea8c2c185 skipped: already exists
Copying blob 972107ece720 skipped: already exists
Copying blob 9857eeea6120 skipped: already exists
Copying blob 93859aa62dbd skipped: already exists
Copying blob 8e47efbf2b7e skipped: already exists
Copying blob 42e0f5a91e40 skipped: already exists
Copying config ef4b14a72d done
Writing manifest to image destination
Storing signatures
ef4b14a72d02ae0577eb0632d084c057777725c279e12ccf5b0c6e4ff5fd598b
```

</p>
</details>

### Run a pod with the image pushed to the registry

<details><summary>show</summary>
<p>

```bash
:~$ kubectl run simpleapp --image=$registry_ip:5000/simpleapp --port=80

:~$ curl ${kubectl get pods simpleapp -o jsonpath='{.status.podIP}'}
Hello, World!
```

</p>
</details>

### Log into a remote registry server and then read the credentials from the default file


<details><summary>show</summary>
<p>

> Note: The two most used container registry servers with a free plan are [DockerHub](https://hub.docker.com/) and [Quay.io](https://quay.io/).

```bash
:~$ podman login --username $YOUR_USER --password $YOUR_PWD docker.io
:~$ cat ${XDG_RUNTIME_DIR}/containers/auth.json
{
        "auths": {
                "docker.io": {
                        "auth": "Z2l1bGl0JLSGtvbkxCcX1xb617251xh0x3zaUd4QW45Q3JuV3RDOTc="
                }
        }
}
```

</p>
</details>

### Create a secret both from existing login credentials and from the CLI

<details><summary>show</summary>
<p>

```bash
:~$ kubectl create secret generic mysecret --from-file=.dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json --type=kubernetes.io/dockeconfigjson
secret/mysecret created
:~$ kubectl create secret docker-registry mysecret2 --docker-server=https://index.docker.io/v1/ --docker-username=$YOUR_USR --docker-password=$YOUR_PWD
secret/mysecret2 created
```

</p>
</details>

### Create the manifest for a Pod that uses one of the two secrets just created to pull an image hosted on the relative private remote registry

<details><summary>show</summary>
<p>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: $YOUR_PRIVATE_IMAGE
  imagePullSecrets:
  - name: mysecret
```

</p>
</details>

### Clean up all the images and containers

<details><summary>show</summary>
<p>

```bash
:~$ podman rm --all --force
:~$ podman rmi --all
:~$ kubectl delete pod simpleapp
```

</p>
</details>


## Understand Jobs and CronJobs

- [Understanding Kubernetes Jobs and CronJobs](https://teckbootcamps.com/understanding-kubernetes-jobs-and-cronjobs/)<sup>Blog</sup>


### Create a job named pi with image perl:5.34 that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"

<details><summary>show</summary>
<p>

```bash
kubectl create job pi  --image=perl:5.34 -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

</p>
</details>

### Wait till it's done, get the output

<details><summary>show</summary>
<p>

```bash
kubectl get jobs -w # wait till 'SUCCESSFUL' is 1 (will take some time, perl image might be big)
kubectl get po # get the pod name
kubectl logs pi-**** # get the pi numbers
kubectl delete job pi
```
OR

```bash
kubectl get jobs -w # wait till 'SUCCESSFUL' is 1 (will take some time, perl image might be big)
kubectl logs job/pi
kubectl delete job pi
```
OR

```bash
kubectl wait --for=condition=complete --timeout=300s job pi
kubectl logs job/pi
kubectl delete job pi
```

</p>
</details>

### Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'
```

</p>
</details>

### Follow the logs for the pod (you'll wait for 30 seconds)

<details><summary>show</summary>
<p>

```bash
kubectl get po # find the job pod
kubectl logs busybox-ptx58 -f # follow the logs
```

</p>
</details>

### See the status of the job, describe it and see the logs

<details><summary>show</summary>
<p>

```bash
kubectl get jobs
kubectl describe jobs busybox
kubectl logs job/busybox
```

</p>
</details>

### Delete the job

<details><summary>show</summary>
<p>

```bash
kubectl delete job busybox
```

</p>
</details>

### Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml
vi job.yaml
```

Add job.spec.activeDeadlineSeconds=30

```bash
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  activeDeadlineSeconds: 30 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - while true; do echo hello; sleep 10;done
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```
</p>
</details>

### Create the same job, make it run 5 times, one after the other. Verify its status and delete it

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'echo hello;sleep 30;echo world' > job.yaml
vi job.yaml
```

Add job.spec.completions=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  completions: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
```

Verify that it has been completed:

```bash
kubectl get job busybox -w # will take two and a half minutes
kubectl delete jobs busybox
```

</p>
</details>

### Create the same job, but make it run 5 parallel times

<details><summary>show</summary>
<p>

```bash
vi job.yaml
```

Add job.spec.parallelism=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  parallelism: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
kubectl get jobs
```

It will take some time for the parallel jobs to finish (>= 30 seconds)

```bash
kubectl delete job busybox
```

</p>
</details>


kubernetes.io > Documentation > Tasks > Run Jobs > [Running Automated Tasks with a CronJob](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)

### Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

<details><summary>show</summary>
<p>

```bash
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

</p>
</details>

### See its logs and delete it

<details><summary>show</summary>
<p>

```bash
kubectl get po # copy the ID of the pod whose container was just created
kubectl logs <busybox-***> # you will see the date and message 
kubectl delete cj busybox # cj stands for cronjob
```

</p>
</details>

### Create the same cron job again, and watch the status. Once it ran, check which job ran by the created cron job. Check the log, and delete the cron job

<details><summary>show</summary>
<p>

```bash
kubectl get cj
kubectl get jobs --watch
kubectl get po --show-labels # observe that the pods have a label that mentions their 'parent' job
kubectl logs busybox-1529745840-m867r
# Bear in mind that Kubernetes will run a new job/pod for each new cron job
kubectl delete cj busybox
```

</p>
</details>

### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to start execution after its scheduled time (i.e. the job missed its scheduled time).

<details><summary>show</summary>
<p>

```bash
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
vi time-limited-job.yaml
```
Add cronjob.spec.startingDeadlineSeconds=17

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: time-limited-job
spec:
  startingDeadlineSeconds: 17 # add this line
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: time-limited-job
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}
```

</p>
</details>

### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it successfully starts but takes more than 12 seconds to complete execution.

<details><summary>show</summary>
<p>

```bash
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
vi time-limited-job.yaml
```
Add cronjob.spec.jobTemplate.spec.activeDeadlineSeconds=12

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: time-limited-job
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: time-limited-job
    spec:
      activeDeadlineSeconds: 12 # add this line
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}
```

</p>
</details>

### Create a job from cronjob.

<details><summary>show</summary>
<p>

```bash
kubectl create job --from=cronjob/sample-cron-job sample-job
```
</p>
</details>


### Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

<details><summary>show</summary>
<p>

Easiest way to do it is create a pod with a single container and save its definition in a YAML file:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'echo hello;sleep 3600' > pod.yaml
vi pod.yaml
```

Copy/paste the container related values, so your final YAML should contain the following two containers (make sure those containers have a different name):

```YAML
containers:
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    name: busybox2
```

```bash
kubectl create -f pod.yaml
# Connect to the busybox2 container within the pod
kubectl exec -it busybox -c busybox2 -- /bin/sh
ls
exit

# or you can do the above with just an one-liner
kubectl exec -it busybox -c busybox2 -- ls

# you can do some cleanup
kubectl delete po busybox
```

</p>
</details>

### Create a pod with an nginx container exposed on port 80. Add a busybox init container which downloads a page using "wget -O /work-dir/index.html http://neverssl.com/online". Make a volume of type emptyDir and mount it in both containers. For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP"

<details><summary>show</summary>
<p>

Easiest way to do it is create a pod with a single container and save its definition in a YAML file:

```bash
kubectl run box --image=nginx --restart=Never --port=80 --dry-run=client -o yaml > pod-init.yaml
```

Copy/paste the container related values, so your final YAML should contain the volume and the initContainer:

Volume:

```YAML
containers:
- image: nginx
...
  volumeMounts:
  - name: vol
    mountPath: /usr/share/nginx/html
volumes:
- name: vol
  emptyDir: {}
```

initContainer:

```YAML
...
initContainers:
- args:
  - /bin/sh
  - -c
  - "wget -O /work-dir/index.html http://neverssl.com/online"
  image: busybox
  name: box
  volumeMounts:
  - name: vol
    mountPath: /work-dir
```

In total you get:

```YAML

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: box
  name: box
spec:
  initContainers: 
  - args: 
    - /bin/sh 
    - -c 
    - "wget -O /work-dir/index.html http://neverssl.com/online"
    image: busybox 
    name: box 
    volumeMounts: 
    - name: vol 
      mountPath: /work-dir 
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    volumeMounts: 
    - name: vol 
      mountPath: /usr/share/nginx/html 
  volumes: 
  - name: vol 
    emptyDir: {} 
```

```bash
# Apply pod
kubectl apply -f pod-init.yaml

# Get IP
kubectl get po -o wide

# Execute wget
kubectl run box-test --image=busybox --restart=Never -it --rm -- /bin/sh -c "wget -O- $(kubectl get pod box -o jsonpath='{.status.podIP}')"

# you can do some cleanup
kubectl delete po box
```

</p>
</details>

## Utilize persistent and ephemeral

- [Understanding Kubernetes Volumes and Configuration Data](https://teckbootcamps.com/understanding-kubernetes-volumes-and-configuration-data/)<sup>Blog</sup>


### Create busybox pod with two containers, each one will have the image busybox and will run the 'sleep 3600' command. Make both containers mount an emptyDir at '/etc/foo'. Connect to the second busybox, write the first column of '/etc/passwd' file to '/etc/foo/passwd'. Connect to the first busybox and write '/etc/foo/passwd' file to standard output. Delete pod.

<details><summary>show</summary>
<p>

*This question is probably a better fit for the 'Multi-container-pods' section but I'm keeping it here as it will help you get acquainted with state*

Easiest way to do this is to create a template pod with:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'sleep 3600' > pod.yaml
vi pod.yaml
```
Copy paste the container definition and type the lines that have a comment in the end:

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox2 # don't forget to change the name during copy paste, must be different from the first container's name!
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  volumes: #
  - name: myvolume #
    emptyDir: {} #
```
In case you forget to add ```bash -- /bin/sh -c 'sleep 3600'``` in template pod create command, you can include command field in config file

```YAML
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
```

Connect to the second container:

```bash
kubectl exec -it busybox -c busybox2 -- /bin/sh
cat /etc/passwd | cut -f 1 -d ':' > /etc/foo/passwd # instead of cut command you can use awk -F ":" '{print $1}'
cat /etc/foo/passwd # confirm that stuff has been written successfully
exit
```

Connect to the first container:

```bash
kubectl exec -it busybox -c busybox -- /bin/sh
mount | grep foo # confirm the mounting
cat /etc/foo/passwd
exit
kubectl delete po busybox
```

</p>
</details>


### Create a PersistentVolume of 10Gi, called 'myvolume'. Make it have accessMode of 'ReadWriteOnce' and 'ReadWriteMany', storageClassName 'normal', mounted on hostPath '/etc/foo'. Save it on pv.yaml, add it to the cluster. Show the PersistentVolumes that exist on the cluster

<details><summary>show</summary>
<p>

```bash
vi pv.yaml
```

```YAML
kind: PersistentVolume
apiVersion: v1
metadata:
  name: myvolume
spec:
  storageClassName: normal
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  hostPath:
    path: /etc/foo
```

Show the PersistentVolumes:

```bash
kubectl create -f pv.yaml
# will have status 'Available'
kubectl get pv
```

</p>
</details>

### Create a PersistentVolumeClaim for this storage class, called 'mypvc', a request of 4Gi and an accessMode of ReadWriteOnce, with the storageClassName of normal, and save it on pvc.yaml. Create it on the cluster. Show the PersistentVolumeClaims of the cluster. Show the PersistentVolumes of the cluster

<details><summary>show</summary>
<p>

```bash
vi pvc.yaml
```

```YAML
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mypvc
spec:
  storageClassName: normal
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```

Create it on the cluster:

```bash
kubectl create -f pvc.yaml
```

Show the PersistentVolumeClaims and PersistentVolumes:

```bash
kubectl get pvc # will show as 'Bound'
kubectl get pv # will show as 'Bound' as well
```

</p>
</details>

### Create a busybox pod with command 'sleep 3600', save it on pod.yaml. Mount the PersistentVolumeClaim to '/etc/foo'. Connect to the 'busybox' pod, and copy the '/etc/passwd' file to '/etc/foo/passwd'

<details><summary>show</summary>
<p>

Create a skeleton pod:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'sleep 3600' > pod.yaml
vi pod.yaml
```

Add the lines that finish with a comment:

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes: #
  - name: myvolume #
    persistentVolumeClaim: #
      claimName: mypvc #
status: {}
```

Create the pod:

```bash
kubectl create -f pod.yaml
```

Connect to the pod and copy '/etc/passwd' to '/etc/foo/passwd':

```bash
kubectl exec busybox -it -- cp /etc/passwd /etc/foo/passwd
```

</p>
</details>

### Create a second pod which is identical with the one you just created (you can easily do it by changing the 'name' property on pod.yaml). Connect to it and verify that '/etc/foo' contains the 'passwd' file. Delete pods to cleanup. Note: If you can't see the file from the second pod, can you figure out why? What would you do to fix that?



<details><summary>show</summary>
<p>

Create the second pod, called busybox2:

```bash
vim pod.yaml
# change 'metadata.name: busybox' to 'metadata.name: busybox2'
kubectl create -f pod.yaml
kubectl exec busybox2 -- ls /etc/foo # will show 'passwd'
# cleanup
kubectl delete po busybox busybox2
kubectl delete pvc mypvc
kubectl delete pv myvolume
```

If the file doesn't show on the second pod but it shows on the first, it has most likely been scheduled on a different node.

```bash
# check which nodes the pods are on
kubectl get po busybox -o wide
kubectl get po busybox2 -o wide
```

If they are on different nodes, you won't see the file, because we used the `hostPath` volume type.
If you need to access the same files in a multi-node cluster, you need a volume type that is independent of a specific node.
There are lots of different types per cloud provider [(see here)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes), a general solution could be to use NFS.

</p>
</details>

### Create a busybox pod with 'sleep 3600' as arguments. Copy '/etc/passwd' from the pod to your local folder

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -- sleep 3600
kubectl cp busybox:/etc/passwd ./passwd # kubectl cp command
# previous command might report an error, feel free to ignore it since copy command works
cat passwd
```

</p>
</details>
