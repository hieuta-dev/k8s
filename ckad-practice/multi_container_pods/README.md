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
