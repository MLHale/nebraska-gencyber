# RESTFul APIs and Little bits

### Cybersecurity First Principles in this lesson
 (delete this line and all bullets that do not apply )

* __Abstraction__: An abstraction is a representation of an object or concept. The gauges in an automobile are an abstraction of the performance of a car. A map is an abstraction of the earth. Abstraction allows us to model an object without considering all of the (possibly infinite) complexity that the object may contain. In this lesson, we use abstraction to model data structures similarly as we did in lesson one.

* __Information Hiding__: Information hiding is any attempt to prevent people from being able to see information. It can be hiding the content of a letter, or it can be applied to hiding how the letter is delivered. Both ways can prevent people from being able to see the information. This lesson looks at how information can be rendered **meaningless** for eavesdroppers using cryptography.

* __Layering__: Cyber security uses multiple layers of defense or protecting information. If one layer is defeated the next layer should catch it. This lesson emphasizes the human element, that is we as humans have to catch problems (like phishing) that make it past the email-level and browser-level filtering mechanisms.

* __Least Privilege__: One of the ways to protect information is by limiting what people can see and do with your information and resources.

* __Minimization__: Minimization refers to having the least functionality necessary in a program or device. The goal of minimization is to simplify and decrease the number of ways that software can be exploited. This can include **turning off ports that are not needed**, reducing the amount of code running on a machine, and/or turning off unneeded features in an application. This lesson focuses specifically on turning off ports that aren't in use.

* __Modularization__: The concept of modularity is like building blocks. Each block (or module) can be put in or taken out from a bigger project. Each module has its own separate function that is interchangeable with other modules.

* __Resource Encapsulation__: A resource can be hardware such as memory, disk drives, or a display screen. It can also be system objects such as semaphores, a linked list, or shared memory. Processes (or programs) need resources to run. Resources have to be separated and used in the way they were intended. Virtual machines encapsulate an operating system and all of the processes and applications it contains.

* __Simplicity__: Simplicity allows a person to better understand hardware and software. Without the clutter of unnecessarily complicated code and interfaces, the software will be more understandable by people that will update the code when requirements change. It will be easier to understand by the testers and they will be able to spot problems sooner. By keeping software as simple and as focused as possible, the reliability and security is greatly increased.

### Introduction
In this module, you will learn how to build a server of your own and connect it up to Littlebits.

### Goals
By the end of this tutorial, you will be able to:
* Build and deploy a `Django` server into a `container`
* Create a `REST endpoint` on the `application server`
* Connect your server to Littlebits to signal your `cloudbit` to turn on


### Materials Required
For this lesson, you will need:

* PC
* Internet connection
* Little bits cloud bit and API Key
* Little bits sensor and actuator

### Prerequisite lessons
You should complete the following lessons before attempting this lesson.
* [Hands on IoT: Little Bits Intro](../hands-on-iot-little-bits-intro/README.md),
* [Hands on IoT: Build an IFTTT IoT app w/ Little Bits](../hands-on-iot-little-bits-ifttt-app/README.md),
* [Web Services and REST](../restful-api/README.md)
* [Containers](../containers/README.md)

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:true -->
- [RESTFul APIs and Little bits](#restful-apis-and-little-bits)
    - [Cybersecurity First Principles in this lesson](#cybersecurity-first-principles-in-this-lesson)
    - [Introduction](#introduction)
    - [Goals](#goals)
    - [Materials Required](#materials-required)
    - [Prerequisite lessons](#prerequisite-lessons)
    - [Table of Contents](#table-of-contents)
    - [Step 1: Review - Where are we so far?](#step-1-review---where-are-we-so-far)
    - [Step 2: No, you won't be starting from scratch](#step-2-no-you-wont-be-starting-from-scratch)
    - [Step 3: Setup the server](#step-3-setup-the-server)
    - [Step 4: Run the server](#step-4-run-the-server)
    - [Step 5: Explore the server](#step-5-explore-the-server)
    - [Step 6: Press the button](#step-6-press-the-button)
    - [Step 7: Make a new REST endpoint to make the client button work with the backend](#step-7-make-a-new-rest-endpoint-to-make-the-client-button-work-with-the-backend)
    - [Step 8: Hook your server up to the Littlebits API](#step-8-hook-your-server-up-to-the-littlebits-api)
    - [Step 9: Profit!](#step-9-profit)
    - [Step 10: Get events from Littlebits](#step-10-get-events-from-littlebits)
    - [Checkpoint](#checkpoint)
    - [Additional Resources](#additional-resources)
    - [Acknowledgements](#acknowledgements)
    - [License](#license)

<!-- TOC END -->


### Step 1: Review - Where are we so far?
Before we get started, lets talk about what we've done so far. First, we looked at `Littlebits` and saw that we could plug and play the parts together to create new inventions. We wired these up and hooked them to `IFTTT` to let the web control our `cloudbit`. Then, in the `REST` lesson, we looked behind-the-scenes to see how `web services` like IFTTT and Littlebits actually work. We played around with `POSTMAN` and interacted with the Littlebits `API`.

The last thing we left off with was hooking the Littlebits API up to our own server (instead of IFTTT) so we could make our own home-automation controller Dashboard. We saw how `containers` could be used to host `isolated` servers on another host machine.

Now, in this lesson, we will examine how to create our own server and deploy it in a container.

### Step 2: No, you won't be starting from scratch
The process of creating a new application server from the ground up takes some time and attention. Instead of having you start from the ground up, we are providing you with some **starter** skeleton code. This code does the basics of accepting requests and storing data that comes in. Instead of building it, we will look at and examine how it operates before modifying it to make it more secure.

Let's get started by using `git` to clone the skeleton code repository and get it in onto our local machine.

```bash
git clone --recursive https://github.com/MLHale/nebraska-gencyber-dev-env.git

```

Now change directories into the code folder and then use `docker` to build the `image` that our container will use:
```bash
cd nebraska-gencyber-dev-env
docker-compose build
```

With this, we should be able to type the following and see our new image.

```bash
docker images
```
It will be called something like `nebraskagencyberdevenv_django`.

### Step 3: Setup the server
This server is completely new, so we need to do some setup to get it initially configured. Execute the following to run the server and open up a bash terminal to interact with it.

```bash
docker-compose run django bash
```

In this terminal that opens in the container, we need to tell our `Django` server to setup the database and create a new user account for us. The first two lines below setup the database by creating a `database Schema` that our SQL server can use to store data. The third line creates a new superuser account. Specify a password for admin. In development, you can use something simple (e.g. admin1234) for simplicity. In practice, you would want to use a much more secure password - since the server could be accessed from the wider internet.

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser --username admin --email admin
exit
```

### Step 4: Run the server
With the database initialized, you should be able to easily run the app. All you need to do is:

```bash
docker-compose up
```

This docker command executes the container using the `docker-compose.yml` file located in your `/nebraska-gencyber-dev-env/` folder.
* It works by mapping `port 80` on the `host` to `port 8000` in the container.
* Inside the container, Django executes its `runserver` utility - which works like a web server.
* There is also a second container that starts up and runs our `postgres` database server.
* You can take a look at the `Dockerfile` in your `/nebraska-gencyber-dev-env/` folder to learn more about what happens behind the scenes.

With the server running, you should be able to visit [http://localhost](http://localhost) to see your server. You should see a messages that says `Hello World` and has a giant button. We will come back to the button in a minute.

### Step 5: Explore the server

### Step 6: Press the button

### Step 7: Make a new REST endpoint to make the client button work with the backend

### Step 8: Hook your server up to the Littlebits API

### Step 9: Profit!
It works!

### Step 10: Get events from Littlebits
Oops Littlebits can't talk to our server. Lets fix that by opening up some ports. We will explore that in the [Hardening: OS Level](../firewall/README.md) lesson.

### Checkpoint
Lets review what we've learned.






### Additional Resources
For more information, investigate the following.

* [http://developers.littlebitscloud.cc/](http://developers.littlebitscloud.cc/) - API reference for the Littlebits web service.

### Acknowledgements
Special thanks to [Dr. Robin Gandhi](http://faculty.ist.unomaha.edu/rgandhi/), Andrew Li, and April Guerin for reviewing and editing this module.

### License
[Nebraska GenCyber](https://github.com/MLHale/nebraska-gencyber) <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.

Overall content: Copyright (C) 2017  [Dr. Matthew L. Hale](http://faculty.ist.unomaha.edu/mhale/), [Dr. Robin Gandhi](http://faculty.ist.unomaha.edu/rgandhi/), and [Doug Rausch](http://www.bellevue.edu/about/leadership/faculty/rausch-douglas).

Lesson content: Copyright (C) [Dr. Matthew Hale](http://faculty.ist.unomaha.edu/mhale/) 2017.  
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">This lesson</span> is licensed by the author under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
