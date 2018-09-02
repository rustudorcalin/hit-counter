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
 
 ### Deployment steps
**I have tested this on Google Kubernetes Engine**

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
