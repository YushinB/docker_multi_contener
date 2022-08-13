# Build multi application with docker
Build the multi appication with front end, back end  and mongo database. 

## dockerizing mongo database service. <br>
![image](https://user-images.githubusercontent.com/34083808/184476501-a9d216dd-c713-4b5d-884e-6c75bfbf8858.png)

please refer to [this site](https://hub.docker.com/_/mongo) on docker hub website for more information on how to use mongo database with docker.

```
docker run --name mongodatabase -d --rm -p 27017:27017 mongo
```

## dockerizing backend server. <br>
create the dockerfile to build the backend image. 
```
FROM node:14

WORKDIR /app

COPY package.json /app

RUN npm install

COPY . /app

EXPOSE 80

CMD ["node","app.js"]

```
modify the localhost:27017 :arrow_right: host.docker.internal:27017 to able to connect with mongo database.
```
docker run --name backend_app -d --rm -p 80:80 backend:1.1
```

## dockerizing front end server. <br>

create the dockerfile to build the frontend image. 
```
FROM node:14

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

```
docker run --name frontend_app -d --rm -p 3000:3000 frontend:1.1
```

## make docker networks.
create docker networks.
```
docker network create app-net
```
run the container with networks.
before running the back end server, we need to change the IP address of mongo db to container name in order to comunicate with mongodb container. 
```host.docker.internal:27017``` :arrow_right: ```mongodb:27017```
the IP adress in the front end app also need to change to the backend container name. 
``` localhost/goals``` :arrow_right: ``` backend_app/goals```
```
docker run --name mongodb -d --rm --network goals-net mongo

docker run --name backend_app -d --rm -p 80:80 --network goals-net backend:1.1

docker run --name frontend_app -d --rm  -p 3000:3000 frontend:1.1
```

> there is one problem if we stop MongoDB and run it again, the data from it will be removed and we lost our data. As a result, we need to add the volume to prevent data from being lost when the container is stopped. For inspecting purposes, a bind mount can also be used to store data on a local host machine. <br>  
```
docker run --name mongodb -d --rm -v data:/data/db --network goals-net mongo
```
> mongodb image also supports ```MONGO_INITDB_ROOT_USERNAME``` and ```MONGO_INITDB_ROOT_PASSWORD``` for creating a simple user with the role root in the admin authentication database, as described in the Environment Variables section above.<br>

```
docker run --name mongodb -d --rm -v data:/data/db --network goals-net -e MONGO_INITDB_ROOT_USERNAME=Yushin -e  MONGO_INITDB_ROOT_PASSWORD=admin mongo
```
> Create volume for persit the log data from back end app and bind mount to update source code from our host machine. 
```
docker run --name backend_app -d --rm -p 80:80 -v "D:\Source\Git\Docker\multi-01-starting-setup\backend":/app -v logs:/app/logs  -v :/app/node_modules --network goals-net backend:1.1
```

> we can add environment variable to control the authentication of mongodb from outside of container. the environment variable can be declare in docker file and replace with username and password in sever.js file. <br>
```
EXPOSE 80

ENV MONGODB_USERNAME=root
ENV MONGODB_PASSWORD=admin

CMD ["npm","start"]
```

``` javascript
mongoose.connect(
  `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals?authSource=admin`,
  {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  },
```
```
docker run --name backend_app -d --rm -p 80:80 -v "D:\Source\Git\Docker\multi-01-starting-setup\backend":/app -v logs:/app/logs  -v :/app/node_modules --network goals-net -e MONGODB_USERNAME=yushin backend:1.1
```
> Create volume for persit the log data from front end app and bind mount to update source code from our host machine. 
```
docker run -v "D:\Source\Git\Docker\multi-01-starting-setup\frontend\src":/app/src --name frontend_app -d --rm  -p 3000:3000 frontend:1.1
```

## Development vs Production/ Deployment.<br>
![image](https://user-images.githubusercontent.com/34083808/184497138-1828ce18-071d-4d46-9ea3-caa2c7811dba.png)

## Room for Improvement.<br>
![image](https://user-images.githubusercontent.com/34083808/184497160-92b34fab-34ae-43f1-b636-651898714022.png)
