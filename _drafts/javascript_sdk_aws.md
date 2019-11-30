nodejs sdk auto base64 encodes, so no need to to do new Buffer('foo').toString('base64');
endpoints are https://{{service}}.{{region}}.amazonaws.com
when running in lambda, it uses the functions associated execution role IAM and uses that roles credentials.

.aws/credentials file can have many profiles and then the env varilable AWS_PROFILE is used to pick a profile other than default. this can be setting process.env.AWS_PROFILE or by selecting credential provider.

var credentials = new AWS.SharedIniFileCredentials({profile: 'work-account'});
AWS.config.credentials = credentials;

AWS.config.loadFromPath('./config.json');
where config.json be like 
{ "accessKeyId": <YOUR_ACCESS_KEY_ID>, "secretAccessKey": <YOUR_SECRET_ACCESS_KEY>, "region": "us-east-1" }

Guide on using AWS SDK JAVASCRIPT / nodejs
https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide

pass callback as last param to the request object creation
add event listener on request object and then call `send`
call promise() on the request object, it will immediately start the service and return a promise.

call `createReadStream` to get stream of data and bypass SDK processing.
Use event and callbacks on the stream instead of the request object.


The response object has data property. It also has the original request object so you can do
```
response.data
response.request.params.Key
```

JSON is an object which is an unordered collection of name value pairs. It is also an array which is an ordered collection of values.

SNS (Simple Notification Service) is a web service to manage delivery of messages from subscribers to consumers. Publishers asynchronously send message to a topic and consumers can subscribe to topics and will be notified. Subscribers can be a lambda function, email address, SMS enabled phone number, push message enabled mobile application.

SQS (Simple Queue Service) message queuing serice. has standard FIFO queue with high throughput and at-least once processing. consumers can poll or pull data.

Get Lambda Logs locally
sam logs --name boo -t
