# Connecting to cloudbit api from a rest client notes

* Go to http://control.littlebitscloud.cc/
* Create an account or login
* click on your cloudbit
* go to the settings menu
* find your access token and device id, you will need them in next steps
* Launch postman
* build a new GET request to: https://api-http.littlebitscloud.cc/v2/devices
* select headers and add the following

```
Authorization: Bearer <your access token>
```

* you should receive a response back with the name and meta information for each of your cloudbits currently connected

looks kind of like:

```
[
    {
        "label": "mlhale-cloudbit",
        "id": "00e04c036f15",
        "subscriptions": [],
        "subscribers": [],
        "user_id": 175306,
        "is_connected": true,
        "input_interval_ms": 200
    }
]
```

* use the device id to get the same info about the device
* make a GET request to https://api-http.littlebitscloud.cc/v2/devices/<your-device-id>
* e.g. https://api-http.littlebitscloud.cc/v2/devices/00e04c036f15

* make a POST request to set the voltage output on the cloudbit for a few seconds https://api-http.littlebitscloud.cc/v2/devices/<your-device-id>/output - make sure to include a content type and send the variables you want to set
* e.g. https://api-http.littlebitscloud.cc/v2/devices/00e04c036f15/output

Headers:
```
Authorization: Bearer <your access token>
Content-type: application/json
```

Body:
```
{
	"percent": 100,
	"duration_ms": 5000
}
```

You should get back:

```
{
    "success": true
}
```

and you should see your led light up.

Lets add a subscriber to catch input events going to the cloudbit:
* make a POST request to: https://api-http.littlebitscloud.cc/v2/subscriptions
* in our case we want to make a server listen for the cloudbit, so lets use a URI endpoint as the subscriber

same headers as before
body:
```
{
	"publisher_id": "<your device id>",
	"subscriber_id": "http://ourserver.com/endpoint"
}
```

## Large NOTE TO SELF: Need to figure out a way around the VPN issue that the servers will be behind if we want to do this - maybe proxy the request through the Gencyber server =)) Mike grove will love me for that.
