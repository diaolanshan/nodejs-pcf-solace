A *Nodejs* app which target to demo how to use webMessaging and smf protocol to connect to a solace instance in the Bosch IoT Cloud.

The source can be downloaded from [here]().

The app use 
 - [solclientjs](https://www.npmjs.com/package/solclientjs) lib, so add solclientjs 10.1.0 to your *package.json*.
 - [node-cfenv](https://github.com/cloudfoundry-community/node-cfenv) in order to easily access the cf enviroment variables, so also add cfenv 1.1.0 to your *package.json*.
 - [restify](http://restify.com/) in order to have rest api, so also add that dependency to your *package.json*.

and finally, your *package.json* will looks like this:
```json
{
    "name":"nodejs-pcf-examples",
    "version":"0.0.1",
    "description":"Cloud Foundry + Solace + amqp example.",
    "main":"app.js",
    "author":"Eric Zhang",
    "dependencies": {
        "restify": "7.2.2",
        "solclientjs": "10.1.0",
        "cfenv": "1.1.0",
        "basic-auth": "2.0.1"
    }
}
```

**How to use it:**

- Download and push the app to the cloud foundry platform.
- Provision an solace instance and bind to the app.
- Restart/Restage the app.

**How to test it:**

The app contains three urls,

- http://{link_to_app}/amqp/solace/queue, this will trigger the solace [SEMP v2](https://docs.solace.com/API-Developer-Online-Ref-Documentation/swagger-ui/index.html) 
to create a new durable queue which is : nodejs-pcf-amqp-solace-queue.
- http://{link_to_app}/amqp/solace/pub publish 1 message to the queue: _nodejs-pcf-amqp-solace-queue_
- http://{link_to_app}/amqp/solace/sub subscribe 1 message from the queue: _nodejs-pcf-amqp-solace-queue_

and also you can check the app logs and the connections in the soladmin/gui to verify.
 
To create the queue:

```javascript
    // Create a queue which named as nodejs-pcf-amqp-solace-queue
    function create_queue(req, res, next){
        const http = require('http')
        var querystring = require('querystring');
    
        var contents = JSON.stringify({
            'queueName': queue_name,
            'accessType': 'non-exclusive',
            'permission': 'consume',
            'ingressEnabled': true,
            'egressEnabled': true
        });
    
        var options = { 
            hostname: management_host, 
            path: '/SEMP/v2/config/msgVpns/'+ msg_vpn +'/queues',
            method: 'POST',   // indicate this is a POST request.
            auth: management_username + ":" + management_password,
            headers: {
              'Content-Type': 'application/json',
              'Content-Length': Buffer.byteLength(contents)
            }
        }; 
    
        var request = http.request(options, function(response){
            response.setEncoding('utf8');
            response.on('data', function (data) {
               console.log('did get data: ' + data);
            });
            response.on('end', function () {
                console.log('request complete!');
            });
            response.on('error', function (error) {
                console.log('\n Error received: ' + error);
            });
        });
    
        request.write(contents);
        request.end();
    
        res.send(200 , 'success');
        return next();
    }
```

How to publish message, check the code under QueueProduer.py

How to subscribe message, check the QueueConsumer.py