# Penetration Testing

### Cybersecurity First Principles in this lesson

* __Abstraction__: An abstraction is a representation of an object or concept. The gauges in an automobile are an abstraction of the performance of a car. A map is an abstraction of the earth. Abstraction allows us to model an object without considering all of the (possibly infinite) complexity that the object may contain. In this lesson, we use abstraction to model data structures similarly as we did in lesson one.

* __Information Hiding__: Information hiding is any attempt to prevent people from being able to see information. It can be hiding the content of a letter, or it can be applied to hiding how the letter is delivered. Both ways can prevent people from being able to see the information. This lesson looks at how information can be rendered **meaningless** for eavesdroppers using cryptography.

* __Layering__: Cyber security uses multiple layers of defense or protecting information. If one layer is defeated the next layer should catch it. This lesson emphasizes the human element, that is we as humans have to catch problems (like phishing) that make it past the email-level and browser-level filtering mechanisms.

* __Least Privilege__: One of the ways to protect information is by limiting what people can see and do with your information and resources.

* __Minimization__: Minimization refers to having the least functionality necessary in a program or device. The goal of minimization is to simplify and decrease the number of ways that software can be exploited. This can include **turning off ports that are not needed**, reducing the amount of code running on a machine, and/or turning off unneeded features in an application. This lesson focuses specifically on turning off ports that aren't in use.

* __Modularization__: The concept of modularity is like building blocks. Each block (or module) can be put in or taken out from a bigger project. Each module has its own separate function that is interchangeable with other modules.

* __Resource Encapsulation__: A resource can be hardware such as memory, disk drives, or a display screen. It can also be system objects such as semaphores, a linked list, or shared memory. Processes (or programs) need resources to run. Resources have to be separated and used in the way they were intended. Virtual machines encapsulate an operating system and all of the processes and applications it contains.

* __Simplicity__: Simplicity allows a person to better understand hardware and software. Without the clutter of unnecessarily complicated code and interfaces, the software will be more understandable by people that will update the code when requirements change. It will be easier to understand by the testers and they will be able to spot problems sooner. By keeping software as simple and as focused as possible, the reliability and security is greatly increased.

### Introduction
In this module you will probe the server you created before to see how insecure API endpoints can be.

### Goals
By the end of this tutorial, you will be able to:
* Use a REST Client to make malicious `POST` and `GET` requests to an `API`
* Identify `vulnerabilities` in API endpoints
* Trace `vulnerabilities` back to the code `weaknesses`
* Conduct a `risk assessment`


### Materials Required
For this lesson, you will need:

* PC
* Internet connection
* Little bits cloud bit and API Key
* Little bits sensor and actuator

### Prerequisite lessons
You should complete the following lessones before proceeding with this one.
* [Hands on IoT: Little Bits Intro](../hands-on-iot-little-bits-intro/README.md)
* [Hands on IoT: Build an IFTTT IoT app w/ Little Bits](../hands-on-iot-little-bits-ifttt-app/README.md)
* [RESTFul APIs](../restful-api/README.md)
* [Containers](../containers/README.md)
* [Building a server](../building-a-server/README.md)


### Table of Contents


### Step 1: Where we left off
When we left off, you had created an `endpoint` to make the button work to send a message to your `cloudbit`. We had to store our API key on the server to make this work. Our endpoint accepted a POST request to turn on the device. We also integrated our server with `Littleibts API` to `subscribe` to events that occured on the cloudbit.

In this lesson we will take a look at the security implications of this service integration to see how our server isn't well protected against attacks.


### Step 2: Key Penetration Testing Concepts

### Step 3: Getting started testing in POSTMAN
We've created this pretty cool API and nice client-side interface to use it. However, our API is, by default, pretty insecure! In the next sections we will see just how bad it is.

* We will look at them one at a time. First lets start our server. Change into your `nebraska-gencyber-dev-env` folder.
* type `docker-compose up` to run the server
* Now open `POSTMAN` and send a simple `GET` request to your local server at `https://localhost`.
* What do you get?
* Now try sending a `GET` to `https://localhost/api/deviceevents`
* What do you see?

### Step 4: Exploring Authentication and permissions
The first issue on our server, is that it doesn't enforce `authentication`. This violates the __least privilege__ first principle, because anonymous users should only be able to login, not see or interact with data.

* Lets go back to our browser and go to http://localhost.
* If you are logged in, click `logout`
* What do you see?

![data still there!](.img/data-still-there.png)

What gives? Our data is still visible when we are logged out.
This is because our server is not enforcing authentication on its `API endpoints`

### Step 5: Exploring the Device endpoint

### Step 6: Exploring the DeviceEvent endpoint

### Step 7: Exploring Error Handling Behavior

### Step 8: Crafting a cross-site scripting request

### Step 9: Risk Assessment - Summarizing your test results

### Checkpoint
Lets review what we've learned.



               <h2>UNIT 5: Access control and Permissions</h2>
               <h3 class="taskitem">10. See how vulnerable the API is by default</h3>
               Finally,  <br>
               <br>
               <img src="images/insecure-1.png"/><br>
               Thinking about this screenshot, it might occur to you that this is problematic. This issue illustrates the failure of a cybersecurity first principle: Resource encapsulation. The REST APIs are currently an "exposed representation" that can be invoked independent of other interfaces. Right now, all of our API calls can be executed without a user actually being logged in. To prove this and explore just how bad the security of the API is, we will be using a tool called the 'advanced rest client' Open the tool by going <a href="chrome-extension://bljmokabgbdkoefbmccaeficehkmlnao/RestClient.html" target="_parent">here</a>.<br>

               Once opened, lets start with a simple GET request to http://localhost:8000/api/likes/. Enter the url, select GET and hit 'send'. As you can see the data is wide open without authentication or permissions.<br>
               <img src="images/insecure-2.png"/><br>
               Next lets try making an un-authenticated POST request. Select POST, click 'form', and then enter the following key / value pairs in the payload.
               user: 1<br>
               title: &lt;p onmouseover=javascript:alert('well, this is insecure')&gt;test&lt;/p&gt;<br>
               objid: 1000<br>
               secret: hi<br>
               server: hi<br>
               farm: hi<br>
               When done, hit send.
               <img src="images/rest-client-1.png"/><br>
               <img src="images/rest-client-2.png"/><br>
               Notice that it happily accepts the submission and produces the new like for us. This is a totally random, unauthenticated user that just made an arbitrary textual post to another user's profile page. Worse, the post has malicious javascript in it that could run if the real user 1 sees the page. Lets check this out. Going back and signing in as student and going to the student user profile we see <a href=localhost:8000/profile/1>localhost:8000/profile/1</a>:<br>
               <img src="images/rest-client-3.png"/><br>
               Clearly, the attack has been injected into the user's browser - thankfully ember filters out these basic kinds of attacks automatically. That said - we shouldnt rely on in-browser and in-framework protections - we should make the API itself be more secure.<br>
               Lets try two more attacks - a PUT (to overrite existing content) and a DELETE - to remove content entirely. Change the target address to http://localhost:8000/api/likes/&lt;idjustcreated&gt;/ where &lt;idjustcreated&gt; is replaced with the number returned under 'id' in the response for the previous request, for me that number is 27. Change the title to something else - like "see what i did here" and send. As you can see - anonymous users can overrite existing data - a huge problem!<br>&nbsp<br>
               <img src="images/rest-client-4.png"/><br>
               <img src="images/rest-client-5.png"/><br>
               Lastly, lets DELETE this item entirely. Select DELETE and just hit send.<br>
               <img src="images/rest-client-6.png"/><br>
               Poof - there it goes.<br> Check it yourself: <a href=http://localhost:8000/profile/1>http://localhost:8000/profile/1</a>. Log in as student, if you are not already logged in, to notice the deleted "see what i did" post that we have put there before.
             </div><!-- End Task 10-->
             <div id="task11" class="taskitem">
               <h3 class="taskitem">11. Adding access control and permissions for API calls</h3>
               Obviously there are lots of problems with these default security measures. Lets start by actually requiring users to at least authenticate before they can make requests to the API.<br>
               To enable authenticated views only, we need to open /var/www/gencybersite/gencybersite/settings.py in sublime<br>
               Scroll down until you see the REST_FRAMEWORK block and add in the following:<br>
               <pre><code>'DEFAULT_PERMISSION_CLASSES': [
   'rest_framework.permissions.IsAuthenticated'
]

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
