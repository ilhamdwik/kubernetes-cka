# Other Topics

Pre-Requisites - JSON PATH
In the upcoming lecture we will explore some advanced commands with kubectl utility. But that requires JSON PATH. If you are new to JSON PATH queries get introduced to it first by going through the lectures and practice tests available here.

https://kodekloud.com/p/json-path-quiz



Once you are comfortable head back here:



I also have some JSON PATH exercises with Kubernetes Data Objects. Make sure you go through these:

https://mmumshad.github.io/json-path-quiz/index.html#!/?questions=questionskub1

https://mmumshad.github.io/json-path-quiz/index.html#!/?questions=questionskub2


### JSON PATH Example
```
kubectl get pods -o=jsonpath='{ .items[0].spec.containers[0].image }'

kubectl get nodes -o=jsonpath='{.items[*].metadata.name}'

kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.architecture}'

kubectl get nodes -o=jsonpath='{.items[*].status.capacity.cpu}'

kubectl get nodes -o=jsonpath='{.items[*].metadata.name}{.items[*].status.capacity.cpu}'

kubectl get nodes -o=jsonpath='{.items[*].metadata.name} {"\n"} {.items[*].status.capacity.cpu}'
```

NB:
{"\n"} : New Line
{"\t"} : Tab


Loops - Range
```
FOR EACH NODE                                          | '{range}'

	PRINT NODE NAME \t PRINT CPU COUNT \n          |

END FOR                                                |
```


