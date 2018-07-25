* Docker
  * Welcome, this is is about Docker as a developer tool
  * Ok, enjoy the nice art. It's all you get, it's text from here on
  * BTW, The whales name is Moby, Moby Dock
* About me
  * I'm Ryan
  * Lead Software Engineer at InVision
  * You can reach at my email address
* Questions
  * If you have any questions raise you hand.
  * The code and slides can be found at thise repos
* Overview
  * Pain points, containers and vms, Docker, and Docker Composer
  * How many here have heard of Docker?
  * How many use Docker regularly as part of their devopment tools?
* Pain Points
  * Docker solves a lot of pain points
  * When you're working on teams people are shipping code
  * Easy to pull other devlopers work between tasks
  * You can easily setup your own local env
  * Once implemented Docker at your company, onboarding new developers can be as easy as checking at a repo and run some commands from a README
* Micro vs Monolitic
  * Docker helps both
  * Monolithic applciations can still have a few services
  * Docker does alleviate a lot of work around setting up a dozen services
* Routine maintence
  * When setting up a new service, managing the database can be annoying.
  * As touched on earlier, not everyone is a sys admin. So you either end up with small instructions that expect a lot of prerequists, or super detailed instructions that take forever to do
  * It's easier to keep developers enviornments more setup with simliar versions
  * Routine updates is a couple of commands and updates all your containers
* Containers vs VMs
  * I used to be strong proponent of VMs, but I've been finding containers much easier to work with
  * I'm not even sure if I will use VMs in my local enviornment anymore
  * The decision comes down to if the application or context requires the additional isolation VMs provide
* Docker Containers
  * Docker is one several options for containers. 
  * Kubernetes are more popular on the production/operations side
  * Because of the shared OS, all containers share storage and networking stack
* Installing
  * Super easy to install, runs on the major OSs
  * Funny thing is Docker for Windows and macOS use a VM to host the containers
* Our repo
  * Stupid simple Node.js app
  * All requests get responded with Hello World and a visit counter
* Directory
  * The files we are insterested in right now are index.js and Dockerfile
  * We will touch on some of the other files and directories later
* Application code
  * We have a db incCount function that returns the number of visits from an IP address
  * We are bound to port 8080
* Dockerfile
  * Some key things, we are using Node.js 10.x. It's possible to specify a more version of Node.js
  * The FROM tell Docker to download/use that specific image
  * Each line is one step in a series of steps to create our image
  * This does not create a our container, that's a later step
* Docker Images
  * In our Docker file we are using Node.js 10.x
  * It's possible to use one of over 10k images available on Docker Hub
  * Nearly all popular services have containers
  * Over the weekend I used a container on Docker Hub to setup a Starbound service on Amazon ECS
  * You can make and publish you own. We are making a new local image with our Dockerfile, but not publishing it
* Building & Running
  * We run docker build to create our image, we taging it with frontent and telling the command to build from the local directory
  * We can see the series of steps, I've removed some additional output to keep this concise
  * And last but not least we see it's been tagged. It's possible version it, instead of using latest
* Running the image
  * Now it's time to actaully create the container
  * We call docker run with the tag we used
  * As we can see the STDOUT from our application
  * We can also run it in the background using the -d switch
  * We won't see STDOUT and we can confirm it's running with docker ps
  * Key things here are the IMAGE, STATUS, PORTS, and NAME
* Ports
  * Ok, our application is running. Lets try to access it.
  * Intersting, docker ps said it was on port 8080
  * So, here we get to one of the first things that trips developer up
  * These ports are exposed by the container with a hostname matching the container name
  * But, they are only available inside of the private docker network
  * If you want to access them from your terminal, browser, postman, whatever you have to map the port to a port on your OS
  * The reason for this you can multiple containers exposing the same port, we have tell docker docker container by container and port by port which ports to expose to us through our OS
  * So, we stop and remove continer and recreate with the -p flag
  * In this example we map our OS's port 8080 on all interfaces to the containers port 8080
  * If we had another container exposing 8080, we could map 8081 to that containers 8080
  * We check that the new container is running, and we try our curl request again
  * Drat, it worked, but we don't have a database!
* Logs
  * Lets take quick look how we would investigate issues withour service
  * We can us docker logs to see STDOUT and STDERR for container by container
  * -f and --tail are super useful
* DB & Migrations
  * Ok, so we need a database
  * We could use apt-get, brew, or whatever tool MS provides (do they even provide a tool for this?) to spin up our new database, then connect to it, create a user, a db, and create our schema
  * This presentation is about Docker, so we are going to use Docker!
  * Postgres has a docker image, we can do all the steps I just said in one line
  * <point out key things>
  * But, we can do much better than this
* Docker Compose
  * This is favorite feature of Docker
  * It makes managment so much easier.
  * We can define one yaml file and starting, stopping, updating all become one liners
* docker-compose.yml
  * So, this yaml file. 
  * It pretty much takes all the switches and details we provide docker run and puts them in a file
  * This file can be checked in to a repo, that repo can be checked out by your new hire, and they can have their dev env up in the time it takes to pull the images and/or build the app.
  * The file contains a version, the Docker devs are actively extending options in this file
  * And a collection of services, in our file we have db, migrations, and frontend
* DB service
  * The service is named db, we use the 9.6 postgres image from Docker Hub
  * It will restart if the container exits
  * We map the port so we can connect to it from our OS
  * And we provide some env vars document on the Docker Hub page
  * These envars will create a user for us and create an empty db
* Migrations serivce
  * Next up we have the migration service
  * This is going to a little hand wavey. Migrations take the DATABASE_URL and use a node package called db-migrate
  * It connects to the DB, checks a table it creates that tracks applied migrations, then runs any unapplied migrations
  * It has a migrations direcotry with migration files that contain up and down functions that apply and remove the migrations
  * This container runs and exits quickly, apply the database migrations
  * It depends on the db service
* Frontend service
  * This is where our code runs, it has the build run switches that saw earlier
  * Depends on db and migrations
  * It's only env var is DATABASE_URL, which are based on the env var we gave the pg container
  * Make sure to notice that hostname is db, the name we gave the services
* Starting in foreground
  * We can run docker-compose up we get all the STDOUT and STDERR from all 3 containers
  * We can see the migrations ran with no migrations (there was migrations, but not worth showing
  * And our app started, we can Ctrl-C and stop all the containers
* Starting in background
  * We can run the same command with a -d.
  * All the same stuff as our last slide happen, but it's in the background.
  * We can use docker-compose ps to look at the status of our defined containers
* Updating the containers
  * Incase someone on our team added a service we git pull
  * Then we pull the latest images
  * And lastely we run docker up with --build to build the in-repo containers
* Developer environments
  * And that's pretty much it, but there are some quick things to keep in mind
  * Your org can have a private image repo
  * Your org's build server can create and upload new images
  * Check your docker-compose.yml in to an env repo
  * It's possible to have the container main your local filesystem, allow hot reload
* What next?
  * Install it, seriously, if you're not already using you're just wasting time
  * Check out the Docker docs for more details, they are super well written
* Questions
* Thanks
 




















