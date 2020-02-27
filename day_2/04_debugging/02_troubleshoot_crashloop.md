# Troubleshooting Crashloops

Because containers are designed to run, and follow a single process until it dies, and Kubernetes can be configured to restart containers that crash, its easy to find yourself looking at a crashloop.

The biggest challenge in troubleshooting a crashloop, is that since the container keeps restarting, its not possible to maintain a connection via exec, nor does any change thats being made, persist across restarts.

We can still connect to the container, and do some inspection, but for these cases we will need to modify the entrypoint command, such that it wont fail for some predetermined time, allowing us to troubleshoot in that window.

Suppose your deployment looks like this:

```
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

Because no command or args are supplied to the container, its using the default entrypoint/command defined in the Dockerfile, when it starts up.

If we edit the yaml, we can ovveride the command to sleep for 10 minutes, which will allow us time to get in and inspect.

```
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
        command: ["sleep"]
        args:
        - "10000"
```

Now, this container wont serve its normal process, but I can connect to the container and manually run that default command and watch the result in realtime.

`kubectl get pods` to get the pod name.

`kubectl exec -it POD_NAME -- sh`

