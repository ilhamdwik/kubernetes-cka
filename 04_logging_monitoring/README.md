# Metrics Server
### https://www.linuxtechi.com/how-to-install-kubernetes-metrics-server/

# Logs Kubernetes
### event-simulator.yaml
```
apiVersion:v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
    - name: event-simulator
      image: kodekloud/event-simulator
    //- name: image-processor
    //  image: some-image-processor
```