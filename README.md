# Streaming Stack

Production ready Docker Stack based on excelent [OvenMediaEngine](https://www.ovenmediaengine.com/ome), Traefik reverse proxy [Traefik](https://traefik.io) to create your own platforms/services that live stream to large audiences on Web, Mobile.
The Stack enable WebRTC P2P streaming when it's available and also support to broadcast stream to Facebook, Youtube Live Channels.


# Environement:

|            |  |
|------------------|------------------------------------|
| DOMAIN           | main domain name, subdomaine names |
| LE_ACM_EMAIL     | Let's encrypt ACM email            |
| API_ACCESS_TOKEN | access token to enable API server  |


# Setup

Just run the command to import stack.yml into portainer web portal.

  docker stack deploy -c stack.yml streaming

# Endpoints

| Publisher | https://$DOMAIN?room=ROOM_ID |  |
| --- | --- | --- |
| Player | https://$DOMAIN/player.html?room=ROOM_ID |   |
| API | https://api.$DOMAIN | authorization="Basic $BASE64_ACCESS_TOKEN” |

Usage example in iframe:

```jsx
<iframe src="https://$DOMAIN/player.html?room=stream1" width="640" height="480" allow="camera 'src';microphone 'src'">
```


## API Example

$API_ACCESS_TOKEN is a token for authentication, and when you invoke the API, you must put "Basic base64encode($API_ACCESS_TOKEN)" in the "Authorization" header of HTTP request.


### Available Streams

To get available streams : 

```jsx
var myHeaders = new Headers();
myHeaders.append("authorization", "Basic BASE64_ACCESS_TOKEN");

var requestOptions = {
  method: 'GET',
  headers: myHeaders,
  redirect: 'follow'
};

fetch("https://$DOMAIN/v1/vhosts/default/apps/app/streams", requestOptions)
  .then(response => response.text())
  .then(result => console.log(result))
  .catch(error => console.log('error', error));
/*
{
    "message": "OK",
    "response": [
        "test"
    ],
    "statusCode": 200
}
*/
```

### Push stream to Youtube

Push a selected stream to remove RTMP destination : 
Notes that stream name most append to the name “_youtube”, example, if you your room name is “test”, stream to push to youtube will be “**test_youtube**”

```jsx
var headers = new Headers();
headers.append("authorization", "Basic BASE64_ACCESS_TOKEN");
headers.append("Content-Type", "application/json");

var raw = JSON.stringify({
  "id": "**UNIQUE_USER_ID_HERE**",
  "stream": {
    "name": "**ROOM_NAME_HERE_youtube**"
  },
  "protocol": "rtmp",
  "url": **"rtmp://a.rtmp.youtube.com/live2"**,
  "streamKey": **"STREAM_KEY_HERE"**
});

var requestOptions = {
  method: 'POST',
  headers: headers,
  body: raw,
  redirect: 'follow'
};

fetch("https://api.$DOMAIN/v1/vhosts/default/apps/app:startPush", requestOptions)
  .then(response => response.text())
  .then(result => console.log(result))
  .catch(error => console.log('error', error));
```

### Push stream to Facebook

For facebook **RTMPS,** use this payload instead **:** 

```jsx
{
    "id": "**UNIQUE_USER_ID_HERE**",
    "stream": {
        "name": "**ROOM_NAME_HERE_facebook**"
    },
    "protocol": "rtmp",
    "url": "rtmps://live-api-s.facebook.com:443/rtmp",
    "streamKey": "**STREAM_KEY_HERE**"
}
```

### top pushed stream :

```jsx
var headers = new Headers();
headers.append("authorization", "Basic BASE64_ACCESS_TOKEN");
headers.append("Content-Type", "application/json");

var raw = JSON.stringify({
  "id": **"UNIQUE_USER_ID_HERE"**
});

var requestOptions = {
  method: 'POST',
  headers: headers,
  body: raw,
  redirect: 'follow'
};

fetch("https://api.$DOMAIN/v1/vhosts/default/apps/app:stopPush", requestOptions)
  .then(response => response.text())
  .then(result => console.log(result))
  .catch(error => console.log('error', error));
```

### List Pushed streams :

```jsx
var myHeaders = new Headers();
myHeaders.append("authorization", "Basic BASE64_ACCESS_TOKEN");

var requestOptions = {
  method: 'GET',
  headers: myHeaders,
  redirect: 'follow'
};

fetch("https://api.$DOMAIN/v1/vhosts/default/apps/app:pushes", requestOptions)
  .then(response => response.text())
  .then(result => console.log(result))
  .catch(error => console.log('error', error));

// Response :
/*
{
    "message": "OK",
    "response": [
        {
            "app": "app",
            "createdTime": "2022-01-16T12:48:25.458+0000",
            "finishTime": "1970-01-01T00:00:00.000+0000",
            "id": "UNIQUE_USER_ID",
            "protocol": "rtmp",
            "sentBytes": 2256710,
            "sentTime": 39517,
            "sequence": 0,
            "startTime": "2022-01-16T12:48:25.598+0000",
            "state": "Pushing",
            "stream": {
                "name": "ROOM_NAME",
                "tracks": []
            },
            "streamKey": "XXXXXXXXXXXXXXX",
            "totalsentBytes": 2256710,
            "totalsentTime": 39517,
            "url": "rtmp://a.rtmp.youtube.com/live2",
            "vhost": "default"
        }
    ],
    "statusCode": 200
}
*/
```

### List recording

To list exiting or ongoing recording session :

```jsx
var headers = new Headers();
headers.append("authorization", "Basic BASE64_ACCESS_TOKEN");
headers.append("Content-Type", "application/json");

var requestOptions = {
  method: 'POST',
  headers: headers,
  redirect: 'follow'
};

fetch("https://api.$DOMAIN/v1/vhosts/default/apps/app:records", requestOptions)
  .then(response => response.text())
  .then(result => console.log(result))
  .catch(error => console.log('error', error));

/*
{
    "message": "OK",
    "response": [
       
        {
            "state": "stopped",
            "id": "custom_id_4",
            "vhost": "default",        
            "app": "app",
            "stream": {
                "name": "stream_o",
                "tracks": []
            },
            "filePath": "app_stream_o_0.ts",
            "infoPath": "app_stream_o.xml",
            "segmentationRule": "discontinuity",
            "sequence": 0,
            "recordBytes": 1907351,
            "recordTime": 6968,
            "totalRecordBytes": 1907351,
            "totalRecordTime": 6968,
            "startTime": "2021-08-31T21:05:01.567+0900",            
            "createdTime": "2021-08-31T21:05:01.171+0900",            
            "finishTime": "2021-08-31T21:15:01.567+0900"
        }
    ],
    "statusCode": 200
}
*/
```

### Start a recording

```jsx
var headers = new Headers();
headers.append("authorization", "Basic BASE64_ACCESS_TOKEN");
headers.append("Content-Type", "application/json");

var raw = JSON.stringify({
  "id": "**CUSTOM_ID**",
  "stream": {
    "name": "**ROOM_ID**"
  }
});

var requestOptions = {
  method: 'POST',
  headers: headers,
  body: raw,
  redirect: 'follow'
};

fetch("https://api.$DOMAIN/v1/vhosts/default/apps/app:startRecord", requestOptions)
  .then(response => response.text())
  .then(result => console.log(result))
  .catch(error => console.log('error', error));
```

Videos will be recorded in /home/kzexo/recording

### Stop a recording

```jsx
var headers = new Headers();
headers.append("authorization", "Basic BASE64_ACCESS_TOKEN");
headers.append("Content-Type", "application/json");

var raw = JSON.stringify({
  "id": "**CUSTOM_ID**"
});

var requestOptions = {
  method: 'POST',
  headers: headers,
  body: raw,
  redirect: 'follow'
};

fetch("https://api.$DOMAIN/v1/vhosts/default/apps/app:stopRecord", requestOptions)
  .then(response => response.text())
  .then(result => console.log(result))
  .catch(error => console.log('error', error));
```