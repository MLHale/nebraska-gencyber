# RESTFul APIs and Little bits

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
    - [Checkpoint](#checkpoint-1)
    - [Additional Resources](#additional-resources)
    - [Acknowledgements](#acknowledgements)
    - [License](#license)

<!-- TOC END -->


### Step 1: Review - Where are we so far?
Before we get started, lets talk about what we've done so far. First, we looked at `Littlebits` and saw that we could plug and play the parts together to create new inventions. We wired these up and hooked them to `IFTTT` to let the web control our `cloudbit`. Then, in the `REST` lesson, we looked behind-the-scenes to see how `web services` like IFTTT and Littlebits actually work. We played around with `POSTMAN` and interacted with the Littlebits `API`.

The last thing we left off with was hooking the Littlebits API up to our own server (instead of IFTTT) so we could make our own home-automation controller Dashboard. We saw how `containers` could be used to host `isolated` servers on another host machine.

Now, in this lesson, we will examine how to create our own server and deploy it in a container.

For reference, this is the overall design we are looking at. On the left side, you have littlebits and the `Littlebits API`. We previously worked with the API using `IFTTT`. In the container lesson you saw how we can setup a new standalone server. In this lesson we will begin building the item marked `custom web API`. It will have features to support authentication, managing our `cloudbit`, and logging events.
![Web App Architecture](./img/web-app-architecture.png)


### Step 2: No, you won't be starting from scratch
The process of creating a new application server from the ground up takes some time and attention. Instead of having you start from the ground up, we are providing you with some **starter** skeleton code. This code does the basics of accepting requests and storing data that comes in. Instead of building it, we will look at and examine how it operates before modifying it to make it more secure.

Let's get started by changing into the Desktop directory and then using `git` to clone the skeleton code repository and get it in onto our local machine.

```bash
C:
cd /Users/student/Desktop/
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
* Leave this terminal running
* It works by mapping `port 80` on the `host` to `port 8000` in the container.
* Inside the container, Django executes its `runserver` utility - which works like a web server.
* There is also a second container that starts up and runs our `postgres` database server.
* You can take a look at the `Dockerfile` in your `/nebraska-gencyber-dev-env/` folder to learn more about what happens behind the scenes.

With the server running, you should be able to visit [http://localhost](http://localhost) to see your server. You should an interface that looks something like the following.  

![skeleton client app](./img/skeleton-client.png)

This is a `web client` (also called a `frontend`) that we've built for demo purposes to work with our server. You will be making the server work better with the client.

### Step 5: Explore the server
Since our focus is the `backend` - lets take a look over our server environment. First. Lets explore the file tree.

* Open `Atom` on your desktop,
* go to the File -> "Add Project Folder..."

![Add folder](./img/add-folder.png)
> note that your interface may look slightly different on windows.

* Find your `nebraska-gencyber-dev-env` folder (it should be located at `C:/Users/student/Desktop/`)
* Upon opening it you should see:

![File Tree](./img/file-tree1.png)

* click `backend` to explore the actual files our server is using
* Click `api` and `django_backend` to expand out the folders and see what we have.
* This code is built using a `Model View Controller` framework called `Django`.
  * `Models` (in `models.py`) are `abstraction` mechanisms that help you represent your data without worrying about the nitty gritty of database technologies.
  * `Controllers` (in `controllers.py`) an `Views` are modularization mechanisms that separate user-facing code (like user interfaces) from backend code that handles number crunching and getting data to the views.
* Look over these three files, first `models.py`, then `urls.py`, then `controllers.py`

#### Models.py
* In `models.py` you will see that we have defined two `models` `Device` and `DeviceEvent`. Both of these are `schema` that have fields in them for each of the types of data they hold.
* In our `Device` model we have fields for the `owner` and the `deviceid`. This model will hold data about our cloudbit so that our backend server can make use of it.

```python
class Device(models.Model):
    owner = models.CharField(max_length=1000, blank=False)
    deviceid = models.CharField(max_length=1000, blank=False)

    def __str__(self):
        return str(self.deviceid)

    class JSONAPIMeta:
        resource_name = "devices"
```

* In the `DeviceEvent` model we have fields for:
  * `device` (i.e. the cloudbit the device event was created for),
  * the `eventtype` which describes what occurred,
  * the `power` which identifies the power level on the device when the event occurred
  * `timestamp` (when it happened)
  * `userid` which is the Littlebits id of the user
  * and `requestor` which logs the IP of the Littlebits server that sent the message

```python
class DeviceEvent(models.Model):
    device = models.ForeignKey(Device, on_delete=models.CASCADE, related_name='events')
    eventtype = models.CharField(max_length=1000, blank=False)
    power = models.IntegerField()
    timestamp = models.DateTimeField()
    userid = models.CharField(max_length=1000, blank=True, unique=True)
    requestor = models.GenericIPAddressField(blank=False)

    def __str__(self):
        return str(self.eventtype) + str(self.device)
```

* Both of the models also include a `__str__` function which outputs a string if the model is converted to a string.

#### Urls.py
Next lets look at `urls.py`. This file tells Django which urls are accessible on the server. If a url entry isn't included in a urls.py file, then the method cannot be accessed.

* The important part of this file, below, identifies all of the `url patterns` that are acceptable for `Django` to server up to any would-be requestors
* Each is a regular expression.
* Each maps to a function in the `controllers.py` file. Basically when someone attempts to visit a url, Django goes through its list of acceptable patterns. If it matches a pattern it executes the corresponding code in that method. If it doesn't maatch any acceptable pattern, it gives the user a `HTTP 404` error (not found).
* in this case, `api/urls.py` is a sub set of patterns that are mapped behind `/api/` as given in the file `django_backend/urls.py`.

**api/urls.py**
```python
urlpatterns = [
    url(r'^session/', controllers.Session.as_view()),
    url(r'^register', csrf_exempt(controllers.Register.as_view())),
    url(r'^deviceevents', csrf_exempt(controllers.DeviceEvents.as_view())),
    url(r'^', include(router.urls)),

    #Django Rest Auth
    url(r'^auth/', include('rest_framework.urls')),

]

```

**django_backend/urls.py**
```python
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework')),
    url(r'^api/', include(api_urls)),
    url(r'^', views.home),
]
```

#### Controllers.py
Next, lets look at the `controllers.py` file to see what the server does when a URL is visited.

There is a lot of code in this file. Lets look at the function that handles requests to the `/api/deviceevents` url.

* Find the code below.
* Notice that this is a `class` that extends the django rest class `APIView`.
* An `APIView` allows you to define functions that handle `GET` (single), `GET`(list), `POST`, `PUT`, and `DELETE` requests that might arrive at `/api/deviceevents`
* The `GET` (single) request is used whenever a user wants to get a single item (typically by id), something like `/api/deviceevents/4` would return the event with id 4.
* The `GET` (list) request is used whenever a user wants to get all of the events.
* The `POST` request is used whenever a user wants to make a new event.
* The `PUT` request is used whenever a user wants to modify an existing event.
* Finally, the `DELETE` request is used whenever a user wants to delete an existing event.

> These conventions are not specific to `Django` they are based on `RESTful API` design standards.

* In our `APIView` we have created two `REST endpoints` for handling `POST` requests and `GET` (list) requests.

* The `post` function looks at the incoming request,
* extracts the data fields from it,
* gets the stored `Device` record (if one exists) or creates a new `Device` record (if one does not exist),
* and then creates and stores a new `DeviceEvent` record based on the incoming request data

* The `get` function simply queries the database for all `DeviceEvent` objects and returns them to the requestor in `JSON` format

```python
class DeviceEvents(APIView):
    permission_classes = (AllowAny,)
    parser_classes = (parsers.JSONParser,parsers.FormParser)
    renderer_classes = (renderers.JSONRenderer, )

    def post(self, request, *args, **kwargs):
        json_req = json.loads(request.POST.get('request'))

        eventtype = json_req.get('payload').get('delta')
        power = json_req.get('payload').get('percent')
        timestamp = json_req.get('timestamp')
        userid = json_req.get('user_id')
        requestor = request.META['REMOTE_ADDR']

        try:
            device = Device.objects.get(deviceid=json_req.get('bit_id'))
        except Device.DoesNotExist:
            #device not created - Create it
            device = Device(
                deviceid=json_req.get('bit_id'),
                owner=userid
            )
            device.save()

        newEvent = DeviceEvent(
            device=device,
            eventtype=eventtype,
            power=power,
            timestamp=datetime.datetime.fromtimestamp(timestamp/1000, pytz.utc),
            userid=userid,
            requestor=requestor
        )

        try:
            newEvent.clean_fields()
        except ValidationError as e:
            print e
            return Response({'success':False, 'error':e}, status=status.HTTP_400_BAD_REQUEST)

        newEvent.save()
        print 'New Event Logged from: ' + json_req.get('bit_id')
        print json_req.get('payload')
        return Response({'success': True}, status=status.HTTP_200_OK)

    def get(self, request, format=None):
        events = DeviceEvent.objects.all()
        json_data = serializers.serialize('json', events)
        content = {'deviceevents': json_data}
        return Response(content)
```


#### Checkpoint
1. Is the URL `<myserver>/api/deviceevents` a valid URL?
1. Is the URL `<myserver>/api/session` a valid URL?
1. What function gets called when the user visits `<myserver>/api/register`?
1. What would be the result of making a `DELETE` request to `<myserver>/api/deviceevents`?
1. What would be the result of making a `POST` request to `<myserver>/api/deviceevents`?

### Step 6: Press the button
Ok, so you now have a loose familiarity with the skeleton `backend` code that was provided to you. Lets build upon it.

Earlier, you saw a green button that said **turn cloudbit on** when you visited `localhost`. Time to push it!


* connect the `power module` to the pink `button` input module
* connect the `button` module to the `cloudbit` module
* connect the `cloudbit` module to the `LED` module

* in your browser, go to http://localhost
* press the button
* what happened?
* How can we tell?

Lets use the `chrome development tools` to take a closer look.

* Press F12 (windows) or Right Click on the page -> Select `Inspect`
* This will bring up the `chrome development tools` which have a number of very helpful capabilities for debugging websites and inspecting the behind-the-scenes action that happens when you interact with web content

### Step 7: Chrome Dev Tools - Your new best friend
Instead of me reinventing the wheel, head over to [https://developers.google.com/web/tools/chrome-devtools/](https://developers.google.com/web/tools/chrome-devtools/) to learn the basics of what Chrome Development tools can do.

When you've looked over the different features. Come back and click on the `network` tab to inspect what is happening with our button.
You should see:

[button](./img/button-request.png)

If you click on `activatecloudbit` you will see the exact request that is getting sent.

[button](./img/button-failing.png)

If you click over to the `response` tab you will see the raw html that the server is returning when this button is clicked.

### Step 8: Make a new REST endpoint to make the client button work with the backend
Currently, the server doesn't know that it needs to do anything special with the url `/api/activatecloudbit` so it is just rendering the home page (what we have been looking at this whole time) in response. What we need is for our server to **recognize that a new event has occurred** from the client and then **do something to handle it**, in this case contact the `Littlebits API`.

For this to work, we need to create a new REST Endpoint controller to handle the request. Open up your `controllers.py` file and add a new entry called `ActivateCloudbit`. This entry will only expose a `POST` endpoint. The goal is to:

* capture the info from the client
* Talk to `Littlebits API` to retrieve the device info for the current user
* Use the retrieved device info to craft a `request` to turn on the `Cloudbit`
* Turn on the cloudbit and log the resulting event locally

```python
class ActivateCloudbit(APIView):
    permission_classes = (AllowAny,)
    parser_classes = (parsers.JSONParser,parsers.FormParser)
    renderer_classes = (renderers.JSONRenderer, )

    def post(self,request):
        print 'REQUEST DATA'
        print str(request.data)

        eventtype = request.data.get('eventtype')
        timestamp = int(request.data.get('timestamp'))
        requestor = request.META['REMOTE_ADDR']
        api_key = ApiKey.objects.all().first()

        #get device info from Littlebits API
        r = requests.get('https://api-http.littlebitscloud.cc/v2/devices/', headers= {
            'Authorization' : 'Bearer ' + api_key.key
        })
        print 'Retrieving List of Devices from Littlebits:'
        print r.json()
        userid = r.json()[0].get('user_id')
        deviceid= r.json()[0].get('id')

        try:
            device = Device.objects.get(deviceid=deviceid)
        except Device.DoesNotExist:
            #device not created - Create it
            device = Device(
                deviceid=deviceid,
                owner=userid
            )
            device.save()

        print "Creating New event"

        newEvent = DeviceEvent(
            device=device,
            eventtype=eventtype,
            power=-1,
            timestamp=datetime.datetime.fromtimestamp(timestamp/1000, pytz.utc),
            userid=userid,
            requestor=requestor
        )

        print newEvent
        print "Sending Device Event to: " + str(deviceid)

        #send the new event (to turn on the device) to littlebits API
        event_req = requests.post('https://api-http.littlebitscloud.cc/v2/devices/'+deviceid+'/output', headers= {
            'Authorization' : 'Bearer ' + api_key.key
        })
        print event_req.json()

        #check to ensure the device was on and received the event
        if (event_req.json().get('success')!='true'):
            return Response({'success':False, 'error':event_req.json().get('message')}, status=status.HTTP_503_SERVICE_UNAVAILABLE)

        #check that the event is safe to store in the databse
        try:
            newEvent.clean_fields()
        except ValidationError as e:
            print e
            return Response({'success':False, 'error':e}, status=status.HTTP_400_BAD_REQUEST)

        #log the event in the DB
        newEvent.save()
        print 'New Event Logged'
        return Response({'success': True}, status=status.HTTP_200_OK)
```
Now that we have the endpoint defined we need to make it available on the web server. Modify `api/urls.py` to include a new line in the `urlpatterns`:

```python
urlpatterns = [
  url(r'^activatecloudbit', csrf_exempt(controllers.ActivateCloudbit.as_view())),
  ...other stuff...
]
```
> Note: don't include the ...other stuff... portion.

This will make the endpoint available on the webserver. Now go back to http://localhost and try to click the button. What happens?

#### Stray observations
* Our new endpoint is a `module` that exemplifies the `modularization` cybersecurity first principle. It doesn't rely on the other modules (endpoints).
* We did not hardcode our `api key` in the code to protect it from static lookup - this is an example of the `information hiding` cybersecurity first principle.
* We made use of `abstraction` and `resource encapsulation` as they relate to the API design for our server and for the `littlebits API`

### Step 9: Get events from Littlebits
The next step, is to not only `send` events to Littlebits, but also to `subscribe` to and `receive` events that are output from the `cloudbit.` To do that, we need to use `POSTMAN` to add a subscriber. This was the last step where we left off in the [REST API](../restful-api/README.md) tutorial. Now we are ready!

Lets add a subscriber to catch input events going to the cloudbit:
* make a POST request, using `POSTMAN` to: https://api-http.littlebitscloud.cc/v2/subscriptions
* in our case we want to make a server listen for the `cloudbit`, so lets use a URI endpoint as the subscriber
* Make sure you use the same headers that you used in the `REST` tutorial. If you don remember it should be:

headers:
```json
{
	"Authorization": "Bearer <your api key here without the angled brackets>",
  "Content-type": "application/json"
}
```

This time, in the body, we are going to use:

body:
```json
{
	"publisher_id": "<your device id without the angled brackets>",
	"subscriber_id": "http://gencyber2017.unomaha.edu/api/proxy/<your-server-ip without the angled brackets>/api/deviceevents",
  "publisher_events": [“amplitude:delta:ignite”]
}
```

* to get your server ip, you need to open a `powershell` and type:
```bash
ipconfig --all
```
* find your ipv4 address on the ethernet card attached to your machine
* alternatively, you can go to http://google.com and search for 'my ip address'
* put the ip in the body of the request above and send the message to the `Littlebits API`
* If the request works, you should see your request echoed back to you

#### Stray observations
* We are using http://gencyber2017.unomaha.edu/api/proxy so the requests from littlebits can make it to your servers in this room.
* UNO, like other universities and companies, uses a `defense in depth` strategy that includes `perimeter network firewalls`. This is an example of `layering`. In this case, we are filtering external requests through a proxy to check their conformity. If they are clean, they are forwarded through our network firewall to your individual servers.
* The `publisher_events` field allows you to subscribe to several different types of events. You can find a full list here: http://developers.littlebitscloud.cc/#create-subscription
* Please do not denial of service the proxy portal - your ip might get automatically banned if you do.

### Step 10: Profit!
Pretty neat. Observe your handy work.

* connect the `power module` to the pink `button` input module
* connect the `button` module to the `cloudbit` module
* connect the `cloudbit` module to the `LED` module

Now, press the button on `button` module. Watch as your server get the events from the `Littlebits API`, logs them locally (creating a database record), stores them for later, and then loads them into the client app for you to see.

> Note: the client is designed to check for updates every 3 seconds.


### Checkpoint
Lets review what we've learned.

quiz-goes-here




### Additional Resources
For more information, investigate the following.

* [http://developers.littlebitscloud.cc/](http://developers.littlebitscloud.cc/) - API reference for the Littlebits web service.
* [https://developers.google.com/web/tools/chrome-devtools/](https://developers.google.com/web/tools/chrome-devtools/) - Chrome Dev Tools overview

#### Checkpointed code
If you want to fast forward between lessons, I have provided you with a complete solution through step 10 of this lesson. You can update to that version using the following command from the terminal where your `nebraska-gencyber-dev-env` is running:

```bash
git checkout tags/step10-server
```

This will update your local code copy to the tagged release version. You can merge it back into your master branch by making a `pull request` into your master branch using `Github`.


### Acknowledgements
Special thanks to [Dr. Robin Gandhi](http://faculty.ist.unomaha.edu/rgandhi/), Andrew Li, and April Guerin for reviewing and editing this module.

### License
[Nebraska GenCyber](https://github.com/MLHale/nebraska-gencyber) <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.

Overall content: Copyright (C) 2017  [Dr. Matthew L. Hale](http://faculty.ist.unomaha.edu/mhale/), [Dr. Robin Gandhi](http://faculty.ist.unomaha.edu/rgandhi/), and [Doug Rausch](http://www.bellevue.edu/about/leadership/faculty/rausch-douglas).

Lesson content: Copyright (C) [Dr. Matthew Hale](http://faculty.ist.unomaha.edu/mhale/) 2017.  
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">This lesson</span> is licensed by the author under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
