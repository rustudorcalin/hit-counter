# Counter API with Python and Flask
This repo contains files needed to create an API using Python and Flask, in order to count webpage hits. The application is bundled into a container and deployed to a kubernetes cluster. The backend database is Redis.

# API functionality

Description: Retrives number of hits for the current deployment.

Request:       `GET /`

Response:     `HTTP/1.1 200 OK`
`I have been hit 9 times since deployment.`

### Details

The app is written in Python, using Flask framework 

 - `app.py` is the actual app code
 - `requirements.txt` are the dependencies required to run the app
 - `Dockerfile` is used to build docker container
 
 ### Building/testing steps

Download/pull this repository:
`git clone https://github.com/rustudorcalin/hit-counter.git`

Go to the newly created directory
`cd hit-counter`

Build and tag your docker image

    $ docker build -t hit-counter . 
    Sending build context to Docker daemon  7.68 kB
    Step 1/6 : FROM python:2-alpine
    ---> 5082b69714da
    Step 2/6 : WORKDIR /usr/src/app
    ---> bce5e8d2f953
    Removing intermediate container af42586f92d6
    Step 3/6 : COPY requirements.txt ./
    ---> c932521438b6
    Removing intermediate container d57bc7bcc4d0
    Step 4/6 : RUN pip install --no-cache-dir -r requirements.txt
    ...
    Step 5/6 : COPY . .
    ---> d48e58cb701e
    Removing intermediate container cd12af31d77d
    Step 6/6 : CMD python ./app.py
    ---> Running in 9bab103fafb7
    ---> 95982cf9d7c8
    Removing intermediate container 9bab103fafb7
    Successfully built 95982cf9d7c8

Make sure to push the image to docker hub:

    $ docker tag hit-counter calinrus/hit-counter
    
    $ docker push calinrus/hit-counter
    The push refers to a repository [docker.io/calinrus/hit-counter]
    898d7f171bf4: Pushed 
    48f330753248: Pushed 
    8096265db166: Pushed 
    60af20e2cf1f: Pushed 
    9d1f139ac886: Layer already exists 
    029d8a704a27: Layer already exists 
    00023a62e045: Layer already exists 
    73046094a9b8: Layer already exists 
    latest: digest: sha256:b2cc2202c58c182daa399016716b679d5a0b36e402ad21149ebb09472f908383 size: 1993

You can test your application and its dependency (Redis) using docker-compose.

    $ docker-compose ps
    Name   Command   State   Ports
    ------------------------------
    
    $ docker-compose up -d
    WARNING: The Docker Engine you're using is running in swarm mode.

    Compose does not use swarm mode to deploy services to multiple nodes in a swarm. All containers will be scheduled on the current node.

    To deploy your application across the swarm, use the bundle feature of the Docker experimental build.

    More info:
    https://docs.docker.com/compose/bundles

    Creating network "docker_default" with the default driver
    Creating docker_redis-lb_1
    Creating docker_web_1
    
    $ docker-compose ps
    Name                     Command               State          Ports        
    ---------------------------------------------------------------------------------
    docker_redis-lb_1   docker-entrypoint.sh redis ...   Up      6379/tcp            
    docker_web_1        python ./app.py                  Up      0.0.0.0:80->5000/tcp
    
    $ curl localhost
    I have been hit 1 times since deployment.
    
    $ curl localhost
    I have been hit 2 times since deployment.
    
    $ curl localhost
    I have been hit 3 times since deployment.

 ### Deployment steps
I have tested this on Google Kubernetes Engine. Let's check that we have the cluster available:
    
    $ kubectl get all
    NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.39.240.1   <none>        443/TCP   14m
    
    $ kubectl get nodes
    NAME                                       STATUS    ROLES     AGE       VERSION
    gke-cluster-1-default-pool-8fd182f9-2vtv   Ready     <none>    13m       v1.9.7-gke.6
    gke-cluster-1-default-pool-8fd182f9-bxw7   Ready     <none>    13m       v1.9.7-gke.6
    gke-cluster-1-default-pool-8fd182f9-qgq9   Ready     <none>    13m       v1.9.7-gke.6
 
 #### 1. Manually creating pods
 In order to create Pods and Services we need to execute the `kubectl apply` command:
     
    $ cd k8s
    
    $ kubectl apply -f create_pods_service.yml 
    pod/first-pod created
    service/myapp-lb created
    pod/second-pod created
    service/redis-lb created
   
 Now check the resources and get the public IP of the app:
    
    $ kubectl get all
    NAME             READY     STATUS    RESTARTS   AGE
    pod/first-pod    1/1       Running   0          1m
    pod/second-pod   1/1       Running   0          1m

    NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
    service/kubernetes   ClusterIP      10.39.240.1     <none>           443/TCP        16m
    service/myapp-lb     LoadBalancer   10.39.247.151   35.204.171.197   80:30945/TCP   1m
    service/redis-lb     ClusterIP      10.39.253.185   <none>           6379/TCP       1m

Let's test our application hiting the public IP address:

    $ curl 35.204.171.197
    I have been hit 1 times since deployment.
    
    $ curl 35.204.171.197
    I have been hit 2 times since deployment.
    
    $ curl 35.204.171.197
    I have been hit 3 times since deployment.

#### Cleanup
To do some cleanup, run the following commands:

    $ kubectl delete service myapp-lb
    service "myapp-lb" deleted
    
    $ kubectl delete service redis-lb
    service "redis-lb" deleted
    
    $ kubectl delete pods first-pod
    pod "first-pod" deleted
    
    $ kubectl delete pods second-pod
    pod "second-pod" deleted
    
    $ kubectl get all
    NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.39.240.1   <none>        443/TCP   18m

#### 2. Using a _Deployment_ controller
To create a Deployment, run the following command:
    
    $ cd k8s
    
    $ kubectl create -f deployment.yml 
    deployment.apps/myapp-deployment created
    deployment.apps/redis-deployment created

    $ kubectl get deployments
    NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    myapp-deployment   3         3         3            3           59s
    redis-deployment   1         1         1            1           59s
    
We still need to create a _Service_ in order for our app to communicate with Redis and outside world.

    $ kubectl create -f services.yml 
    service/myapp-lb created
    service/redis-lb created
    
    $ kubectl get services
    NAME         TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
    kubernetes   ClusterIP      10.39.240.1     <none>           443/TCP        41m
    myapp-lb     LoadBalancer   10.39.250.243   35.204.129.161   80:30128/TCP   1m
    redis-lb     ClusterIP      10.39.246.222   <none>           6379/TCP       1m
    
Test the Deployment:

    $ curl 35.204.129.161
    I have been hit 17 times since deployment.
   
#### Cleanup
To do some cleanup, run the following commands:

    $ kubectl delete deployment myapp-deployment
    deployment.extensions "myapp-deployment" deleted
    $ kubectl delete deployment redis-depoyment
    deployment.extensions "redis-deployment" deleted
    
    $ kubectl get deployments
    No resources found.
    
    $ kubectl delete service myapp-lb
    service "myapp-lb" deleted
    $ kubectl delete service redis-lb
    service "redis-lb" deleted
    
    $ kubectl get services
    NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
    kubernetes   ClusterIP   10.39.240.1   <none>        443/TCP   46m
