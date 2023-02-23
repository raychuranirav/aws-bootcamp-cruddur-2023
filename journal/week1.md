# Week 1 — App Containerization

## REQUIRED HOMEWORK 

Note : In some code you will find explaination so that you can know that I did the research for those code the actual code is without explanation 

I watched the live video and done things which I saw on the video and did some changes into the gitpod.

* Created Dockerfile into the backend-flask folder and written this code into the file


```
# This code will fetch the specified python image from the docker hub
FROM python:3.10-slim-buster

WORKDIR /backend-flask

# This will copy the requirements.txt file into the image. This txt file is currently inside the backend-flask folder
COPY requirements.txt requirements.txt

# This will install all programs/softwares which is required to run our backend 
RUN pip3 install -r requirements.txt

# This will copy every folder inside the image which I am trying to make
COPY . .


ENV FLASK_ENV=development

# This will expose the port specified in our case its 4567
EXPOSE ${PORT}

# This will run our backend via flask module of python
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```
* From the Dockerfile I tried to make an docker container to run our backend in order to do that I run following command
```docker
docker build -t  backend-flask ./backend-flask
```
* Tried to run the container via following command

```sh
# This command is to run the docker will change the entrypoint to backend-flask via -it, remove the image after completing test/whene I exit via --rm, 
# Creating and passing the environment variables FRONTEND_URL and BACKEND_URL via -e , exposing the port 4567 from the inside to outside (Basically its port forwarding) running the container backend-flask at the end I can run the same command with -d to run it in background but I will not do it right now
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
```

After running the command I can open the backend into the browser via URL which is present into the gitpod "PORTS". I made the URL public and I opened the URL and found that its giving me 404 error which is good sign it means the backend is running

![BACKEND IS RUNNING](assets/backend-test.png)

* Created Dockerfile into the frontend-react-js folder and save it with following code

```docker
# This will fetch the specified node image
FROM node:16.18

# This will set the environment variable 
ENV PORT=3000

# This will copy the whole frontend-reeact-js folder into the container 
COPY . /frontend-react-js

WORKDIR /frontend-react-js

# This will install the npm software/program into the container
RUN npm install

# This will expose the port
EXPOSE ${PORT}

# This will start the frontend
CMD ["npm", "start"]
```

Before running the docker build command I have to install the npm into gotpod workspace so I did it via following command

```
npm install
```
As I did to test the backend I run following command to run the frontend 

```
# To build the image
docker build -t frontend-react-js ./frontend-react-js

# To run the container exposing the port 3000
docker run -p 3000:3000 frontend-react-js
```
After the above command I can see the frontend so its working but it wasn't connected to the backend. In order to run both frontend and backend togather I am making a docker-compose.yml file in the project directory with following code

```yml
# This is the version which I am using to run the docker-compose file 
version: "3.8"

# Here I am declaring services which is backend-flask and frontend-react-js
services:
  backend-flask:
  
    # Here I am passing the environtment variables which we have to pass to use this image 
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    
    # This will build/rebuild the image if there is no image or I made some changes into the code
    build: ./backend-flask
    
    # Exposing the ports here which is basically port forwarding
    ports:
      - "4567:4567"
   
   # Here I am attaching the volume of outside of the container to the inside of container its basically used for persistent data    
    volumes:
      - ./backend-flask:/backend-flask

  # All the things will be same for frontend as backend 
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# Here I am making network for docker
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```

After that it will be easy to run backend and frontend at the same time I have to just use to following command to run those

```
# Here I am using the -d for running it to the background
docker compose up -d
```
I can see that frontend is connacted to the backend

![FRONTEND IS CONNACTED TO THE BACKEND](assets/frontend-test.png)
