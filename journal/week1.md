# Week 1 — App Containerization

## References

Good Article for Debugging Connection Refused
https://pythonspeed.com/articles/docker-connection-refused/


## VSCode Docker Extension

Docker for VSCode makes it easy to work with Docker

https://code.visualstudio.com/docs/containers/overview

> Gitpod is preinstalled with theis extension

## Containerize Backend

### Run Python

```sh
cd backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
python3 -m flask run --host=0.0.0.0 --port=4567
cd ..
```

- make sure to unlock the port on the port tab
- open the link for 4567 in your browser
- append to the url to `/api/activities/home`
- you should get back json



### Add Dockerfile

Create a file here: `backend-flask/Dockerfile`

```dockerfile
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```

### Build Container

```sh
docker build -t  backend-flask ./backend-flask
```

### Run Container

Run 
```sh
docker run --rm -p 4567:4567 -it backend-flask
FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
docker run --rm -p 4567:4567 -it  -e FRONTEND_URL -e BACKEND_URL backend-flask
unset FRONTEND_URL="*"
unset BACKEND_URL="*"
```

Run in background
```sh
docker container run --rm -p 4567:4567 -d backend-flask
```

Return the container id into an Env Vat
```sh
CONTAINER_ID=$(docker run --rm -p 4567:4567 -d backend-flask)
```

> docker container run is idiomatic, docker run is legacy syntax but is commonly used.

### Get Container Images or Running Container Ids

```
docker ps
docker images
```


### Send Curl to Test Server

```sh
curl -X GET http://localhost:4567/api/activities/home -H "Accept: application/json" -H "Content-Type: application/json"
```

### Check Container Logs

```sh
docker logs CONTAINER_ID -f
docker logs backend-flask -f
docker logs $CONTAINER_ID -f
```

###  Debugging  adjacent containers with other containers

```sh
docker run --rm -it curlimages/curl "-X GET http://localhost:4567/api/activities/home -H \"Accept: application/json\" -H \"Content-Type: application/json\""
```

busybosy is often used for debugging since it install a bunch of thing

```sh
docker run --rm -it busybosy
```

### Gain Access to a Container

```sh
docker exec CONTAINER_ID -it /bin/bash
```

> You can just right click a container and see logs in VSCode with Docker extension

### Delete an Image

```sh
docker image rm backend-flask --force
```

> docker rmi backend-flask is the legacy syntax, you might see this is old docker tutorials and articles.

> There are some cases where you need to use the --force

### Overriding Ports

```sh
FLASK_ENV=production PORT=8080 docker run -p 4567:4567 -it backend-flask
```

> Look at Dockerfile to see how ${PORT} is interpolated

## Containerize Frontend

## Run NPM Install

We have to run NPM Install before building the container since it needs to copy the contents of node_modules

```
cd frontend-react-js
npm i
```

### Create Docker File

Create a file here: `frontend-react-js/Dockerfile`

```dockerfile
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```

### Build Container

```sh
docker build -t frontend-react-js ./frontend-react-js
```

### Run Container

```sh
docker run -p 3000:3000 -d frontend-react-js
```

## Multiple Containers

### Create a docker-compose file

Create `docker-compose.yml` at the root of your project.

```yaml
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```

## Adding DynamoDB Local and Postgres

We are going to use Postgres and DynamoDB local in future labs
We can bring them in as containers and reference them externally

Lets integrate the following into our existing docker compose file:

### Postgres

```yaml
services:
  db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
volumes:
  db:
    driver: local
```

To install the postgres client into Gitpod

```sh
  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
```

### DynamoDB Local

```yaml
services:
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
```

Example of using DynamoDB local
https://github.com/100DaysOfCloud/challenge-dynamodb-local

## Volumes

directory volume mapping

```yaml
volumes: 
- "./docker/dynamodb:/home/dynamodblocal/data"
```

named volume mapping

```yaml
volumes: 
  - db:/var/lib/postgresql/data

volumes:
  db:
    driver: local
```

# My Notes
- Containerize your app whenever possible unless you are prototyping
- DockerHub/ECR like a github to store your docker images
- Jfrog - for storing artifacts or binaries like jars or libraries
- Docker VSCode extension recommended
- Each command in Dockerfile creates a layer that gets unified
- Best practice: have a Dockerfile for each environment
- Best practice: build the container separate from running the container in the Dockerfile

**CMD vs RUN**
- CMD will run something in the container
- RUN will build the image and create a layer in the container

**Docker command options**
- the `-t` means tag name which should be in the format name:version otherwise will use name:latest
- you can view your docker images via docker vscode extension
- you can view running ports in VScode
- in the docker extension, you can "Attach Shell" to a container to open a shell for that container

### Build Container

```sh
docker build -t  backend-flask ./backend-flask
```

### Run Container

Run 
```sh
# docker run --rm -p 4567:4567 -it backend-flask
# FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
# docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
docker run --rm -p 4567:4567 -it  -e FRONTEND_URL -e BACKEND_URL backend-flask
# unset FRONTEND_URL="*"
# unset BACKEND_URL="*"
```
- The `--rm` means to run a container and then once it terminates it removes the container. Leave out `--rm` if you want the container to persist
- The `-e` means to pass environment variables from current terminal session to the container. You can also pass override values like `FRONTEND_URL='*'`
- The `-it` will open a terminal session to that container once the container starts running

### Verify it works
- Once it is running, go to VSCode Ports tab and unlock the 4567 port
- Then go to the address listed in the port tab
- Append `/api/activities/home` so it would be like: `https://4567-hngu-awsbootcampcrudd-erzhftk1w1y.ws-us110.gitpod.io/api/activities/home`
- You should see some JSON

### Docker compose
Once you have docker files for BE and FE, you can run them both with docker compose. You can use the command `docker-compose up` or `docker compose up` or right right click the `docker-compose.yml` file in VS code and run docker compose up.

You should see two ports in the ports tab: one for FE and one for BE. Click and open the FE one at port 3000.

Typically, get Dockerfiles ready then have the docker-compose to build and run them

### CloudTrail
- For this free class, avoid creating a cloud trail instance
- If you have to, then select the following to keep costs as low as possible:
  - unselect log file encryption
  - for events, do not select data events and insight events
- these recommendations are only for this class. You should turn on CloudTrail with the recommended settings for a production environment 


### S3 empty bucket cost
- Lesson 1: anyone who knows the name of any of your S3 buckets can ramp up your AWS bill
- Lesson 2: adding a random suffix to your bucket names can enhance security
- Lesson 3: when executing a lot of requests to S3, make sure to explicitly specify the AWS region
- S3 charges for unauthorized requests (4xx) as well. So if you open a terminal and type: `aws s3 cp ./file.txt s3://your-bucket-name/random_key` that counts

# Docker Security Best Practices
- Consider Sync Open Source Vulnerability
- Use AWS Secret Manager for supported AWS service, Hashicorp Vault otherwise
- AWS Secret Manager has a cost to them after 30 days
- Turn on automatic rotation
- Use AWS Inspector to scan for image vulnerabilities (has cost after 15 days)
- Consider Sync Container Security

### Docker Components
<img width="859" alt="Screenshot 2024-05-09 at 1 54 07 PM" src="https://github.com/hngu/aws-bootcamp-cruddur-2023/assets/725417/425cb034-7b24-42e1-83b7-effd1df6e85a">

There are 2 main components: Docker client and Docker server

### Top 10 Best Security Practices
1. Keeping Host and Docker updated to latest security patches
2. Docker container and daemon should run in non-root user mode
3. image vulnerability scanning
4. trusting a private vs public image registry
5. No sensitive data in docker files or images
6. use secret management services to share secret
7. read only file system and volume for Docker
8. separate database for long term storage
9. use DevSecOps practices while building application security
10. ensure all code is tested for vulnerability before production

### Running the app
1. Go to FE code and run `npm i` (not sure why react-scripts isn't picked up in docker). **ANSWER**: because in our `docker-compose.yml` file, we are mounting our local FE directory to the docker container's working directory. That will overwrite anything that is in the container's directory with our work, and since there is no `node_modules` in our host machine, the container won't have it either. This is so that we can use the same Dockerfile for both local and production. You should have a different Dockerfile for both local and production. Building a unified container is only if you are sharing it with other colleagues to make sure their environments matches yours.
2. Then run `docker compose up`
3. Open the app at port 3000
4. If you have not done so, click Join Now
5. Confirmation code is 1234

### DynamoDB Local
Test these commands locally to make sure dynamodb local is working
Create a table
```
aws dynamodb create-table \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema AttributeName=Artist,KeyType=HASH AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --table-class STANDARD
```

Add an item to that table
```
aws dynamodb put-item \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --item \
        '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}}' \
    --return-consumed-capacity TOTAL  
```

List tables
```
aws dynamodb list-tables --endpoint-url http://localhost:8000
```

Get all records
```
aws dynamodb scan --table-name Music --query "Items" --endpoint-url http://localhost:8000
```

### Postgres local
When working with gitpod or other Cloud IDE, you need to install the postgres client to connect to the postgres local server. To do that, you need to update the `gitpod.yml` file with the client driver install.

Additionally, when connecting to postgres in gitpod context, you have to run the following command:
```
psql -Upostgres --host localhost
```
