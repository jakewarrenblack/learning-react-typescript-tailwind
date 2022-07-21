With docker, we can package our app with everything it needs, eg node and mongodb, so we avoid the problem of 'it doesn't run on my machine'.

another advantage, if somebody joins your team, they dont have to spend two days setting everything up. they can use a docker package to set up their entire environment all at one time.

we can also remove an application with all of its dependencies and clean up our machine with no worries about breaking anything.

so docker lets us consistently build, run, and deploy software applications.

---

## virtual machines vs containers

### vm

- vm = abstraction of a physical machine

- each vm needs a full-blown OS

- slow to start, has to load the whole OS

- resource intensive

### containers

- allow running multiple apps in isolation

- lightweight

- uses the OS of the host

- starts quickly

- needs less hardware resources

### how does docker work

client talks to server using a REST API.

the server is called the 'docker engine'.

it sits in the background and takes care of building 'containers'.. containers really are just processes running on the computer.

remember that unlike VMs, containers share the OS of their host.

windows is now shipped with the linux kernel, so we can run linux and windows containers on windows.

macOS uses a lightweight linux VM to run linux containers, because macOS's kernel doesn't support containers.

## Development workflow

We take our application and 'dockerise' it. This means we add a dockerfile, simple. It's a plaintext file which instructs docker on how to package the app into an image containing a cut-down OS, runtime env like node, app files, libraries, and environment variables.

Docker can then start a container using that image, which is just a process.

Instead of then directly launching the app, we tell docker to run it inside the container.

Once we have the image, we can push it to the 'docker hub' which is like github for images. We can then put it on any machine running docker. Then everything is the same on any other machine.

## dockerfile

We start with a base image:

```
FROM node:alpine
COPY . /a
```

The above is a linux image with node installed on it.

## Pulling down / Running an image

```
docker container run -d -p 8080:80 --name myapache httpd
```

`-d` = detached. This means run in the background. Opposite would be `-it` for interactive.

`-p` = port

8080 = the port we'll use locally

80/whatever it is = the port the app uses internally. Apache uses port 80.

`--name myapache` = naming the image.

`httpd` = the name of the package. This is the name of the apache package on docker hub.

## Using environment variables

```
docker container run -d -p 3306:3306 --name mysql --env MYSQL_ROOT_PASSWORD=12345678 mysql
```

## Stop a container

```
docker container stop mysql
```

## Remove a container

```
docker container rm myapache -f
```

The -f means it'll still be removed even if it's running right now.

## Enter a container

```
docker container exec -it mynginx bash
```

Might have to add 'winpty' at the start if using git bash on windows.

## Creating a docker file

```dockerfile
# We get everything that comes with the nginx image, latest version
FROM nginx:latest

# Working directory will be where nginx has its webroot
WORKDIR /usr/share/nginx/html

# We're saying that when the image builds, copy everything from the current directory into the image, so '. .' = copy all from here
COPY . .
```

CD into the folder where the image is, and then:
`docker image build -t username/nginx-website .`

This will push the image to your dockerhub account.

# Exploring Docker part 2

Taking an existing nodejs/mongodb application and dockerising it.

```dockerfile
# If node gets updated, :latest could mess our application up
FROM node:16

# Where our app will live in the container
WORKDIR /usr/src/app

# Copy the package.json into our working directory, so we can install the packages
# The * wildcard means we'll also move the package-lock.json
COPY package*.json ./

RUN npm install

# Copy whatever's in this directory into our container
COPY . .

# Exposing the port that our app runs on
EXPOSE 3000

# If we have a 'start' script which runs 'node app.js', then we can do this
CMD ["npm", "start"]
```

As this is right now, it won't work because it'll be looking for mongodb.

## Docker-compose files

Docker compose builds a stack of applications to run a complete service. We can add sections to our docker-compose file, each representing a single container, which we can then combine with the other containers, creating the service.

E.g. we could have a docker-compose file which has a web and db section. The web section being the server and the db section being the database, e.g. nginx and mysql. Also think of node or mongodb.

### The structure of the docker-compose.yml file

```yaml
# Instructing Docker Compose as to which version of the tool we're using
version: "3"

# Telling Docker Compose which services we're deploying
services:
  # Telling Docker Compose to deploy a container using the nginx image.
  web:
    image: nginx
  db:
    image: mysql
    # Defining the internal and external ports to use for the database
    ports:
      - "3306:3306"
    # Providing configuration options for the database
    environment:
      - MYSQL_ROOT_PASSWORD=password1
      - MYSQL_USER=user
      - MYSQL_PASSWORD=password2
      - MYSQL_DATABASE=database
```

Brad's example:

```yaml
version: "3"
services:
  # This name is arbitrary, just makes makes sense as the name for the node part
  app:
    container_name: docker-node-mongodb
    # If it fails, restart it
    restart: always
    # Look in the current directory for a dockerfile, use that to build the image
    build: .
    # When going to a separate line like this, use a hyphen
    ports:
      - "80:3000"
    # Link our app container to our mongo container
    links:
      - mongo
  mongo:
    container_name: mongo
    # Pull the image from dockerhub
    image: mongo
    ports:
      - "27017:27017"
```

### Dockerignore

Just like a gitignore. We don't want to add these things to our docker container.

```Dockerignore
node_modules
npm-debug.log
```
