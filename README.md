# Kubernetes's configuration file to deploy and manage Docker based "Voting & Result Application" locally and in AWS cloud.

A simple kubernetes configuration file and steps to deploy "Voting & Result Application" build on various tech stack.


### Note:
```
This is based on the original example-voting-app repository from the docker-examples GitHub page and modified it to work on the Kubernetes cluster LOCALLY and in AWS.

Note:
    - Using deployment [with a) *-deploy.yaml & b) *-service.yaml ] to create both Pod and its service is the right way to deploy the apps. 
    - For testing purpose we could create the pod and service separately as well.
    - Here we are using the first approach i.e. deployment approach.
    - Service type note:    
        - NodePort: To allow access from Host Node & its port to Pod services.
        - LoadBalancer: Same as NodePort but used within supported Cloud providers such as AWS, Azure and GCP who automatically creates load balancer.
        - ClusterIp: To allow communication between Services within the cluster nodes ex front-end invoking back-end service through services.
    - While testing in local we could use Service as 'NodePort' instead of 'LoadBalancer' for voting-app and result-app.          
```


### Front-End Apps:
```
Vote-App : Python
Result-App : Node JS
```

### Back-End:
```
In Memory DB: redis
Database : Postgress  
Worker : .Net 
    - Responsible to take data from in memory "redis" datatbase inserted by voting app and to pass on to "postgress" database for the result app to view.
```


## Pre-requisite:
```
1) Docker Desktop Installed 

2) minikube 
   - https://minikube.sigs.k8s.io/docs/start/ - minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes.
   - minikube provisions and manages local Kubernetes clusters optimized for development workflows.
   - 
   - minikube docker installed - Docker image "gcr.io/k8s-minikube/kicbase"
   - https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/
   - "Minikube is a lightweight Kubernetes implementation that creates a VM on your local machine and deploys a simple cluster containing only one node." or deployed as docker container.

3) cygwin installed to emulate unix commands
```


## Git Hub Repo (Kubernetes): 
#### This Kubernetes deployment configurations/scripts:
``` 
git clone https://github.com/gmdjaveed/kubernetes-voting-app-local-and-aws.git
```

## Docker Hub Repo
#### Docker Images used by the deploy yaml manifest file of this Application:
    This is only for your information. No action is required here.
```
 - redis
 - postgres
 - kodekloud/examplevotingapp_vote:v1
 - kodekloud/examplevotingapp_result:v1
 - dockersamples/examplevotingapp_worker
```

## Git Hub Repo:
#### Original Application Used by the Docker Images:
``` 
git clone https://github.com/dockersamples/example-voting-app.git
    - This is only for your information. No action is required here.    
```


## GOAL-1: Deploy all the Pods and Services in LOCAL Kubernetes cluster and It will maintains the required number of pods and replicas.  Finally Test in browser.
```   
    Steps:
    ------
    1) Start minikube and ensure no other pods are running
    2) Deploy a) redis b)postgress c) worker d) voting-app and e) result-app apps in this order.
       - NodePort Service: The range of valid ports is 30000-32767
    3) Test in browser
    4) Research pods/services/deployments in "minkube" dashboard -> $ minikube dashboard
```

1) Start minikube and ensure no other pods are running
```   
   $ minikube start
   $ kubectl get pods 
   $ kubectl get all
```

2) Deploy Pods & Services:
```
    -- Deploy redis
    $ kubectl create -f redis-deploy.yaml
    $ kubectl create -f redis-service.yaml
    
    -- Deploy postgres
    $ kubectl create -f postgres-deploy.yaml
    $ kubectl create -f postgres-service.yaml
    
    -- Deploy worker. Note: There is no worker service since no other service is dependent on this. worker depends on redis and postgress services.
    $ kubectl create -f worker-deploy.yaml

    
    -- Deploy voting-app
    $ kubectl create -f voting-app-deploy.yaml
    $ kubectl create -f voting-app-service.yaml
    
    -- Deploy result-app
    $ kubectl create -f result-app-deploy.yaml
    $ kubectl create -f result-app-service.yaml
    
```
3) View deployments, Pod Instances and Services (NodePort/LoadBalancer, ClusterIP):
    Node: We have two replicas for each Pod and Services.
```
    -- View all
    $ kubectl get all

    NAME                                     READY   STATUS    RESTARTS   AGE
    pod/postgres-deploy-8654c876cc-mvrvh     1/1     Running   0          21m
    pod/postgres-deploy-8654c876cc-p9zz6     1/1     Running   0          21m
    pod/redis-deploy-744ff4944f-qxskv        1/1     Running   0          30m
    pod/redis-deploy-744ff4944f-rfkdt        1/1     Running   0          30m
    pod/result-app-deploy-5bd6cd48f5-p5pmc   1/1     Running   0          20m
    pod/result-app-deploy-5bd6cd48f5-sxv89   1/1     Running   0          20m
    pod/voting-app-deploy-85c4dc78d-4j8vb    1/1     Running   0          20m
    pod/voting-app-deploy-85c4dc78d-jr6qn    1/1     Running   0          20m
    pod/worker-deploy-6d66fd5596-2m6gn       1/1     Running   0          21m
    pod/worker-deploy-6d66fd5596-mmndf       1/1     Running   0          21m

    NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    service/db               ClusterIP      10.107.7.43      <none>        5432/TCP       45s
    service/kubernetes       ClusterIP      10.96.0.1        <none>        443/TCP        43m
    service/redis            ClusterIP      10.99.11.84      <none>        6379/TCP       35s
    service/result-service   LoadBalancer   10.100.66.188    <pending>     80:30005/TCP   7s
    service/voting-service   LoadBalancer   10.104.217.120   <pending>     80:30004/TCP   24s

    NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/postgres-deploy     2/2     2            2           21m
    deployment.apps/redis-deploy        2/2     2            2           30m
    deployment.apps/result-app-deploy   2/2     2            2           20m
    deployment.apps/voting-app-deploy   2/2     2            2           20m
    deployment.apps/worker-deploy       2/2     2            2           21m

    NAME                                           DESIRED   CURRENT   READY   AGE
    replicaset.apps/postgres-deploy-8654c876cc     2         2         2       21m
    replicaset.apps/redis-deploy-744ff4944f        2         2         2       30m
    replicaset.apps/result-app-deploy-5bd6cd48f5   2         2         2       20m
    replicaset.apps/voting-app-deploy-85c4dc78d    2         2         2       20m
    replicaset.apps/worker-deploy-6d66fd5596       2         2         2       21m

```
4) Test in browser:
```
    i) Test voting-service(Automatically opens up in browser with http://192.168.49.2:30004)
    $ minikube service voting-service
        output:
        -------
        $ minikube service voting-service
        |-----------|----------------|-------------|---------------------------|
        | NAMESPACE |      NAME      | TARGET PORT |            URL            |
        |-----------|----------------|-------------|---------------------------|
        | default   | voting-service |          80 | http://192.168.49.2:30004 |
        |-----------|----------------|-------------|---------------------------|
        * Starting tunnel for service voting-service.
        |-----------|----------------|-------------|------------------------|
        | NAMESPACE |      NAME      | TARGET PORT |          URL           |
        |-----------|----------------|-------------|------------------------|
        | default   | voting-service |             | http://127.0.0.1:65340 |
        |-----------|----------------|-------------|------------------------|
        * Opening service default/voting-service in default browser...
        ! Because you are using a Docker driver on windows, the terminal needs to be open to run it.

        
       (OR)

        $ minikube service voting-service --url
            output:
            -------
            $ minikube service voting-service --url
            http://127.0.0.1:65455
            ! Because you are using a Docker driver on windows, the terminal needs to be open to run it.
        
    ii) Test service result-service (Automatically opens up in browser with http://192.168.49.2:30005)
        $ minikube service result-service
            output:
            -------
            minikube service result-service
           |-----------|----------------|-------------|---------------------------|
           | NAMESPACE |      NAME      | TARGET PORT |            URL            |
           |-----------|----------------|-------------|---------------------------|
           | default   | result-service |          80 | http://192.168.49.2:30005 |
           |-----------|----------------|-------------|---------------------------|
           * Starting tunnel for service result-service.
           |-----------|----------------|-------------|------------------------|
           | NAMESPACE |      NAME      | TARGET PORT |          URL           |
           |-----------|----------------|-------------|------------------------|
           | default   | result-service |             | http://127.0.0.1:49199 |
           |-----------|----------------|-------------|------------------------|
           * Opening service default/result-service in default browser...
           ! Because you are using a Docker driver on windows, the terminal needs to be open to run it.
```
## GOAL-2: Deploy all the Pods and Services in AWS Kubernetes cluster and It will maintains the required number of pods and replicas. Finally Test in browser.

    High level Steps:
    -----------------
    1) Login to your AWS console. 
```                 
    - For testing purpose you might use root account but this is not advisable in production.
```     
        
    2) Pre requiste:            
```     
    - Need to have two roles a) EKS cluster role and b) Eks group node 
    - Note: Checkout screenshot folder for IAM Role details        
```

    3) Create Node Cluster in AWS us-west-2 (Ohio region). Note: Control plan is fully managed by AWS so no direct access to ssh here.
```
    - Wait for about 10 minutes to complete creation.
    - https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html
    - https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html
    - role issues workarounds : https://devops.stackexchange.com/questions/11722/cannot-configure-node-group-in-new-eks-cluster-due-to-no-node-iam-role-found    
```     

    4) Create Node Group ( Tab Compute -> node group). You can choose to have optionally ssh by selecting the ec2 key option.
```     
    - Wait for about 10 minutes to complete creation.
    - pre requiste : Need to have two roles a) EKS cluster role and b) Eks group node 
```         

    5) Deploy the Pods & Services:
```     
    a)Once EKS Cluster and Node Group are created, click on "Cloud shell" icon to open the terminal from your EKS cluster screen.
      - Checkout the screenshot folder

    b)git all the deployment manifest file from github repo as follows:
      git clone https://github.com/gmdjaveed/kubernetes-voting-app-local-and-aws.git

    c)Deploy the pods and services:
        -- Deploy redis
        $ kubectl create -f redis-deploy.yaml
        $ kubectl create -f redis-service.yaml

        -- Deploy postgres
        $ kubectl create -f postgres-deploy.yaml
        $ kubectl create -f postgres-service.yaml

        -- Deploy worker. Note: There is no worker service since no other is dependent on this. worker depends on redis and postgress services.
        $ kubectl create -f worker-deploy.yaml

        -- Deploy voting-app
        $ kubectl create -f voting-app-deploy.yaml
        $ kubectl create -f voting-app-service.yaml

        -- Deploy result-app
        $ kubectl create -f result-app-deploy.yaml
        $ kubectl create -f result-app-service.yaml   

    d)Verify the deployed pods and services:   
        $ kubectl get all                
```

6) Test in browser:
```            
    i) Get the Load Balancer URL for Voting-App and cast the vote.     
        - See the screenshot 
        - You can get the URL either by running "kubectl get all" and copying the Voting-App LB URL or copying from EKS cluster detail screens.

    2) Get the Load Balancer URL for Result-App and see the result.
        - See the screenshot 
        - You can get the URL either by running "kubectl get all" and copying the Result-App LB URL or copying from EKS cluster detail screens.
        
    Note: Since Two replicas of the instance is running including database, you might not see the result in first attempt so click few more times!
```