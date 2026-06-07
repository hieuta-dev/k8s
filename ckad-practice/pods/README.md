
###### Create a namespaced called 'mynamespace' and a pod with image nginx called nginx on this namespace

```
k ns mynamespace && k run nginx --image=nginx -n mynamespace
```

###### Create the pod that was just described using YAML
```
k run nginx --image=nginx --dry-run=client -o yaml | k apply/create -n mynamespace -f -
```
OR
``` yaml
apiVersion: v1
kind: Pod
metadata:
    name: 
spec.
    containers: 
    - image:
      name: 
```

###### Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output

```bash
k run busybox --image=busybox --restart=Never --command   -it --rm -- sh -c "env"
```

###### Create a busybox pod that runs the command "env"
```bash
k run busybox --image=busypod --restart=Never --dry-run=client -o yaml --command -- env  > busybox.yaml && k create -f busybox.yaml
```

```bash
vi busybox.yaml # Open yaml
```


###### Get YAML for a new namespace called 'myns' without creating it

```bash
k create ns myns --dry-run=client -o yaml > myns.yaml
cat myns.yaml
```

###### Get YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods

```bash
k create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml > myrq.yaml
```

###### Get pods on all namespaces

```bash
k get pods -A
k get pods --all-namespaces
```

###### Create a pod with image nginx called nginx and expose traffic on port 80

```bash
k run nginx --image=nginx --port=80 --restart=Never
```

###### Change pod's image to nginx:1.24.0. Observe that the container will be restarted as soon as the image gets pulled
```bash
k set image pods nginx nginx=nginx:1.24.0 # containerName=newImage
```

###### Get nginx pod's ip created in previous step use a temp busybox image to wget its

```bash
# Get nginx pod's ip
$NGINX_IP=$(k get pods nginx -o jsonpath='{.status.podIP}')
# Run busybox pod with busybox image to wget nginx's IP
k run busybox --image=busybox --env="NGINX=$NGINX_IP" --rm -it --restart=Never -- sh -c 'wget -O- $NGINX' # use sh because we need expand NGINX variable
```

###### Get pods phase

```bash
k get pods nginx -o yaml | grep phase
```