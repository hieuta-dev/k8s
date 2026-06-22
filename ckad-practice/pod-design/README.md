###### Create a pod with two container both with image busy box and command "echo hello; sleep 3600". Connect to the second container and run 'ls'
```yaml
k run twocp --image=busybox --dry-run=client -o yaml -- sh -c "echo hello; sleep 3600" > pod.yaml

# After running command above, editing the yaml file

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: twocp
  name: twocp
spec:
  containers:
  - args:
    - sh
    - -c
    - echo hello; sleep 3600
    image: busybox
    name: first
  - args:
    - sh
    - -c
    - echo hello; sleep 3600
    image: busybox
    name: second
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```



###### Create a pod with an nginx container exposed at port 80. Add a busy box init container which download a page using "echo "Test" > /work-dir/index.html". Make a volume of type emptyDir and mount it in both containers. For nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP"

#1
```bash
k run nginx --image=nginx --port=80 --dry-run=client -o yaml > nginx-pod.yaml
```

#2
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  initContainers:
  - image: busybox
    name: init-html
    command: ['sh', '-c', 'echo "Hello" > /work-dir/index.html']
    volumeMounts:
      - mountPath: /work-dir
        name: html
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: html
    ports:
    - containerPort: 80
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
    - name: html
      emptyDir: {}
status: {}

```

#3
```bash
export NGINX_IP=$(k get pods nginx -o json| jq .status.podIP)
k run busybox-test --image=busybox --env=NGINX_IP=$NGINX_IP -it --rm --restart=Never -- sh -c "wget -O- $NGINX_IP"
```


###### Create 3 pods with name nginx1, nginx2, nginx3. All of them have the label app = v1
```bash
k run nginx1 --image=nginx -l app=v1
k run nginx2 --image=nginx -l app=v1
k run nginx3 --image=nginx -l app=v1
# or

for i in {1..3} ;do k run nginx$i --image=nginx -l app=v1 --restart=Never ; done
```

###### Show all labels of pods

```bash
k get pods --show-labels
# or
k get pods -o json | jq .metadata.labels
```

###### Change labels of pod nginx2 to be app=v2
```bash
k label pods nginx2 app=v2 --overwrite
```

###### Get the label 'app' for the pods

```bash
k get pods -L app
```

###### Get only the app=v2 pods
```bash
k get pods -l app=v2
```

###### Get app=v2 and not tier=frontend
```bash
k get pods -l app=v2,tier!=frontend
```

###### add a new labels tier=web to all pods having 'app=v2' or app=v1 labels

```bash
k label pods -l app=v2,app=v1 tier=web
```

###### Add an annotation 'owner: marketing' to all pods having app=v2 label

```bash
k annotate pod -l app=v2 owner=marketing
```

###### Remove the 'app' label from the pods we created before
```bash
k label pods -l app app- --overwrite
```

###### Annotate pods nginx{1..3} with "description='my description'"

```bash
k annotate pods nginx{1..3} description='my description'
```


###### Check the annotations for pod nginx1
```bash
k annotate pod nginx1 --list
# or
k get pods nginx -o custom-colums=custom-columns=Name:metadata.name,ANNOTATIONS:metadata.annotations.description
```

###### Remove the annotations for these three pods
```bash
k annotate pods description- owner-
```


###### Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'
```bash
k run pod-with-node-selector --image=... --dry-run=client -o yaml > pod-template.yaml

vi pod-template.yaml
```

###### Taint a node with key tier and value frontend with the effect NoSchedule. Then, create a pod that tolerates this taint.
```bash
k taint node minikube tier=frontend:NoSchedule
# create pod template
k run pod-with-tolerants --image=nginx --dry-run=client -o yaml > pod-with-tolerants.yaml
# edit pod template 
vi pod-with-tolerants.yaml
```


###### Create a deployment with image nginx:1.18.0, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)
```bash
k create deployment deployment --image=nginx:1.18.0 --replicas=2 --port=80 --dry-run=client -o yaml > multi_container_pods/deployments/deployment-template.yaml
```

###### View the YAML of Deployments
```bash
k get deployments [name] -o yaml
```

###### View the YAML of the replica set that was created by this deployment
```bash
k get rs [name] -o yaml 
```

###### Check how the deployment rollout is going
```bash
k rollout status deployment [name]
```

###### Update the nginx image to nginx:1.19.8
```bash
# edit running deployment
# or
k set-image deployment [name] containerInPodTemplate=[newImage]
```

###### Undo the latest rollout and verify that new pods have the old image (nginx:1.18.0)
```bash
k rollout undo deploy [name] #empty or to-revision=

```

###### Autoscale the deployment, pods between 5 and 10, targeting CPU utilization at 80%
```bash
k autoscale deploy [name] --min=5 --max=10 --cpu-percent=80
k get hpa
```

###### Implement canary deployment by running two instances of nginx marked as version=v1 and version=v2 so that the load is balanced at 75%-25% ratio

```bash
k create deployments dep-v1 --image=nginx --replicas=3 --dry-run=client -o yaml > dep-v1.yaml
vi dep-v1.yaml
k create svc clusterip --tcp 80:80 --dry-run=client -o yaml > canary-svc.yaml
vi canary-svc.yaml
k creaate deployments dep-v2 --image=nginx --replicas=1 --dry-run=client -o yaml > dep-v2.yaml
vi dep-v2.yaml
```

###### Create a job named pi with image perl:5.34 that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"
``` bash
k create job pi --image=perl:5.34 -- sh -c "perl -Mbignum=bpi -wle 'print bpi(2000)'"
```

###### Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'
```bash
k create job busybox --image=busybox -- sh -c "echo hello; sleep 30; echo world"
```
###### Create the same job, make it run 5 times, one after one.
```bash
k create job busybox --image=busybox --dry-run=client -o yaml -- sh -c "echo hello; sleep 30; echo world" > five-times-job.yaml
vi five-times-jobs.yaml # add completions to the spec with a value of 5. 
k create -f five-times-jobs.yaml 
```
###### Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

```bash
k create cronjob busybox-cronjob --image=busybox --schedule="*/1 * * * *" -- sh -c "date; echo Hello from the Kubernetes cluster" 
```

###### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to start execution after its scheduled time (i.e. the job missed its scheduled time).
```bash
k create cronjob busybox-cronjob --image=busybox --schedule="*/1 * * * *" --dry-run=client -o yaml -- sh -c "date; echo Hello from the Kubernetes cluster"  > pod-design/cronjobs/cronjob-with-activedeadlineseconds.yaml
vi pod-design/cronjobs/cronjob-with-activedeadlineseconds.yaml # add startingDeadlineSeconds in spec with a value of 17
### Manifest YAML
apiVersion: batch/v1
kind: CronJob
metadata:
  name: busybox-cronjob
spec:
  startingDeadlineSeconds: 17
  jobTemplate:
    metadata:
      name: busybox-cronjob
    spec:
      template:
        metadata: {}
        spec:
          containers:
          - command:
            - sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: busybox-cronjob
            resources: {}
          restartPolicy: OnFailure
  schedule: '*/1 * * * *'
```

###### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it successfully starts but takes more than 12 seconds to complete execution.
```bash
k create cronjob busybox-cronjob --image=busybox --schedule="*/1 * * * *" --dry-run=client -o yaml -- sh -c "date; echo Hello from the Kubernetes cluster"  > pod-design/cronjobs/cronjob-with-activedeadlineseconds.yaml
vi pod-design/cronjobs/cronjob-with-activedeadlineseconds.yaml # add activeDeadlineSeconds in jobTemplate with a value of 12
### Manifest YAML
apiVersion: batch/v1
kind: CronJob
metadata:
  name: busybox-cronjob
spec:
  jobTemplate:
    metadata:
      name: busybox-cronjob
    spec:
      activeDeadlineSeconds: 17
      template:
        metadata: {}
        spec:
          containers:
          - command:
            - sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: busybox-cronjob
            resources: {}
          restartPolicy: OnFailure
  schedule: '*/1 * * * *'
bash