#WebRTC ignaling server for nodeJS
This is a signaling server based on the demos of WebRTC-Experiments.
Based on #### [WebSocket over Node.js](https://github.com/muaz-khan/WebRTC-Experiment/blob/master/websocket-over-nodejs) 

You can see three node.js files:

1. signaler.js - HTTP based websocket signaling along with creating websocket channels i.e. rooms
2. ssl.js - HTTPs i.e. SSL based websocket signaling along with creating websocket channels i.e. rooms

=

#### How to use?

Following code explains how to override [`openSignalingChannel`](http://www.rtcmulticonnection.org/docs/openSignalingChannel/) method in your HTML pages; `openSignalingChannel` is useful only for RTCMultiConnection.js and DataChannel.js. For other WebRTC Experiments, please check next section.

```javascript
// wss://wsnodejs.nodejitsu.com:443 (Secure port: HTTPs)
// ws://wsnodejs.nodejitsu.com:80 (Ordinary port: HTTP)

var SIGNALING_SERVER = 'wss://wsnodejs.nodejitsu.com:443';
connection.openSignalingChannel = function(config) {
    config.channel = config.channel || this.channel;
    var websocket = new WebSocket(SIGNALING_SERVER);
    websocket.channel = config.channel;
    websocket.onopen = function() {
        websocket.push(JSON.stringify({
            open: true,
            channel: config.channel
        }));
        if (config.callback)
            config.callback(websocket);
    };
    websocket.onmessage = function(event) {
        config.onmessage(JSON.parse(event.data));
    };
    websocket.push = websocket.send;
    websocket.send = function(data) {
        websocket.push(JSON.stringify({
            data: data,
            channel: config.channel
        }));
    };
}
```

=

#### How to use for `openSocket`?

`openSocket` is used in all standalone WebRTC Experiments. You can define this method in your `ui.js` file or in your HTML page.

```javascript
// wss://wsnodejs.nodejitsu.com:443 (Secure port: HTTPs)
// ws://wsnodejs.nodejitsu.com:80 (Ordinary port: HTTP)

var SIGNALING_SERVER = 'wss://wsnodejs.nodejitsu.com:443';
var config = {
    openSocket = function (config) {
        config.channel = config.channel || 'main-public-channel';

        var websocket = new WebSocket(SIGNALING_SERVER);
        websocket.channel = config.channel;
        websocket.onopen = function () {
            websocket.push(JSON.stringify({
                open: true,
                channel: config.channel
            }));
            if (config.callback)
                config.callback(websocket);
        };

        websocket.onmessage = function (event) {
            config.onmessage(JSON.parse(event.data));
        };

        websocket.push = websocket.send;
        websocket.send = function (data) {
            websocket.push(JSON.stringify({
                data: data,
                channel: config.channel
            }));
        };
    }
};
```

=


#### Dependencies

1. WebSocket - for websocket over node.js connection
2. Node-Static - for serving static resources i.e. HTML/CSS/JS files

=

#### Install via `npm`

```
npm install websocket-over-nodejs
```

and run the `signaler.js` nodejs file:

```
node node_modules/websocket-over-nodejs/signaler.js
```

Now, you can open port "12034" on your ip address/domain; or otherwise on localhost: `http://localhost:12034/`

=
`

Now, you can open port `12034` on your ip address/domain; or otherwise on localhost: `http://localhost:12034/`

It is using port `12034`; you can edit this port using following commands:

```
vi signaler.js

# now edit port 12034
# and save changes & quit

# press "insert" key; then press "Esc" key and the type
:wq
```

`:wq` command saves changes in the file and exits editing mode. If you don't want to save changes; simply type:

```
# if you don't want to save changes however want to exit editing mode
:q
```

Common Error: `Error: listen EADDRINUSE`. It means that same port is used by another application. You can close all existing processes running on same port:

```
// list all active processes running on same port
sudo fuser -v 12034/tcp

// kill all processes running on port "12034"
sudo fuser -vk 12034/tcp

// list again to verify closing ports
sudo fuser -v 12034/tcp
```

You can delete "directory" and re-install:

```
rm -rf websocket-over-nodejs
mkdir websocket-over-nodejs

# and following above steps to "wget" and "tar" then "node" to run!
```

Following error doesn't matter!!! Simply skip it!

```
Warning: Native modules not compiled.  XOR performance will be degraded.
Warning: Native modules not compiled.  UTF-8 validation disabled.
```

=


#### Signaler.js or SSL.js

```javascript
// setting room name
var channelName = location.href.replace(/\/|:|#|%|\.|\[|\]/g, ''); // using URL as room-name

// setting up websocket connection
var websocket = new WebSocket('ws://localhost:12034');

// capturing "onopen" event
websocket.onopen = function () {
    // suggesting node.js server to open a websocket room i.e. channel!
    websocket.push(JSON.stringify({
        open: true,
        channel: channelName
    }));
};

// overriding "send" method to customize default behavior!
websocket.push = websocket.send;
websocket.send = function (data) {
    websocket.push(JSON.stringify({
        data: data,
        channel: channelName
    }));
};

// capturing "onmessage" event!
websocket.onmessage = function (e) {
    console.log(JSON.parse(e.data));
};
```

