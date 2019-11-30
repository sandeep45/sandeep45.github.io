Serverless Aplication Model , not limited to lambda, and can include API, database, event mappings etc.

SAM Template -> defines the function & its configuration
SAM CLI -> uses the template

SAM  is an extension of AWS CloudFormation

aws s3 ls
aws s3 mb s3://bar-artifacts

`sam init` creates sample app. this app has
1. starter hello world code. 
2. events folder with generic event
3. template.yml which defines the application resources. In this case in its resource it has the hello world function along with 1 trigger event which is of type API.

once we have a base place, we can add code, add more files, require them from hello-world function.
The function is in its own folder, with app.js as main file and its own tests folder and pacakage.json file
We can add more files here in this folder for this function and load more npm pacakages inside this folder.
The application can have more functions and we can add them and update template.yml.

Now when we are ready, we can do `sam build` which copies the functions to a staging folder called `.aws-sam`. It does all npm things required to build resources and copies the needed things to .aws-sam and stages so they can next be zipped and deployed. Here all function folders have been copied along with the template.yml

`sam local start-api` provides a local endpoint where the lambda endpoints can be tested. It has hot reloading, but if you have already built a package it will stick and render that. If there is no built folder then it will render your code directly.

`sam package --output-template-file package.yml --s3-bucket bar-artifacts` this takes the stuff from .aws-sam, zips and moves it to the s3 folder provided and then builds a new package.yml file from template.yml and puts the location of the zip in the package.yml where in template.yml it was pointing to the folder where the function was locally. This makes us ready to deploy code using package.yml.


sam takes help as `--help` where aws takes help as `help` 

then we can deploy code to the cloud by `sam deploy --template-file package.yml --stack-name some-name --capabilities CAPABILITY_IAM`
here we are referring the package.yml as it has location of the s3 spot where zip's of our function's are kept. we give it a stack name to use in cloudformation as sam is building using cloudformation and specify capability_IAM since it is creating IAM users for lambda

Now if we access the production end point it has newly updated code.

If we go to aws lambda console our fuction is there named like - 
`stack-name-function-name-random` 

`aws cloudformation list-stacks --query "StackSummaries[].StackName"` shows all stacks available

`aws cloudformation describe-stacks --stack-name aws-sam-getting-started --query "Stacks[].Outputs"` gives all outputs of this stack which will get us the end-point for our function

we can curl the endpoint and see that it works. if we change code and re-deploy we get new results from the endpoint.

To test locally we can do `sam local start-api`. This gives us an endpoint we can hit it locally. 
To run it only 1 time locally we can do `sam local invoke "FunctionName"` then we can type in input follwed by a `Control+d` on a new empty line to mark end of input or pass a input file by doing --event file.json

Debugging Thoughts
run each functions folder as its own seperate self contained node app
run them usind nodemon --inspect=9229
and then just attach a debugger whenever needed
to invoke function you can use sam local invoke and pass in event. same can also be done from aws toolkit plugin in webstorm. to run just code without lambda function overhead, then that seperate file could be run and code needs to be present to run it.
```
if (require.main === module) {
  const answer = exports.lambdaHandler({});
  answer.then(data => console.log(data));
}
```
otherwise it will actually never run. this can be done by tests or directly calling the code based on how its called and in there some event can be passed in.

get all functions in lamnda - 
`aws lambda  list-functions --query "Functions[].FunctionName"`

now we can use function name and tail logs
`sam logs --name aws-sam-getting-started-HelloWorldFunction-17QFAHTHSFFKN --tail`
be mindful that there is a delay

logs will show instance id and if more than one 1 instance of the function is being used it will show up. this is because instance id is in the stream name and stream name is printed first in the logs.

The main thing here is the SAM template.yml. It declares all the AWS resources of the serverless application.

SAM template defines 5 functions specifically, beyond everything which is in CloudFormation

Types:
- AWS::Serverless:Api
- AWS::Serverless:Application
- AWS::Serverless:Function
- AWS::Serverless:LayerVersion
- AWS::Serverless:SimpleTable

CloudFromation covers all other types like AWS::S3::Bucket and they all work here as SAM Template extends CloudFormation Template

`sam local invoke` will invoke a function 1 time. you can pass in event data by `--event` and `--env-vars` to pass in overriding env variables.

to use `sam local invoke` when there are more than 1 function, specify the function name in quotes after invoke.

`sam local start-api` will provide a local end point where all functions which have an api event get mounted and are accessible.

by default sam uses proxy integration, which means the lambda response should have at least `statusCode`, `headers` or `body`. 

It also accepts `--env-vars` to pass in a file with env to override template defined ones. It format is for each function

{
	"Func1": {
		"param1": "value1",
		"param2": "value2"
	},
	"Func2": {
		"param1": "value1",
		"param2": "value2"
	}
}

AWS Layers are a spot where we can put common code and use that in lambda. e.g. could be that npm packages can go there and then the layer is used in the lambda function.
Layers are put in the /opt dierctory in the function execution environment. Then the nodejs run time environment will look for stuff inside opt.

Node looks for libraries inside nodejs/node_modules, so this is how the layer should be organized

Get lambda function names and ARN
`aws lambda list-functions | jq ".Functions[].FunctionName, .Functions[].FunctionArn"`
`aws lambda list-functions | jq ".Functions[] | \"\(.FunctionName) --> \(.FunctionArn)\""`

To run a lambda function already deployed in the cloud for an integation test:

```
const AWS = require('aws-sdk');
var lambda = new AWS.Lambda();

var params = {
  FunctionName: "aws-sam-getting-started-HelloWorldFunction-17QFAHTHSFFKN"
};

lambda.invoke(params, function(err, data) {
  if (err) console.log(err, err.stack); // an error occurred
  else     console.log(data);           // successful response
});
```

basically you are getting the function name and then doing `lambda.invoke`.

now to run the same test start lambda locally by doing `sam local start-lambda`. and then in your test when constructing the lambda function pass the local endpoint where you have lambda running

```
var lambda = new AWS.Lambda({
  endpoint:"http://127.0.0.1:3001",
});
```

now when you do `lambda.invoke` it will run from your local instance of lambda.

`sam local start-lambda` enables progromatic invocation of lambda functions locally via SDK. whereas `sam local start-api` creates a local http server that hosts all functions for local debugging and testing. It also supports hot reloading. whereas `sam local invoke FunctionName` runs function once and quits. this is good for async non http functions which normally run from s3 event or kinesis event etc. you can generate these event bodies and then either pipe them or send them via file to `sam local invoke`.

to make it easy do `sam local generate-event` and generate whatever event request payload for any event needed for testing - `sam local generate-event s3 put`


`sam logs` can be used to get logs of an lambda function either logical function name or by function name as in template.yml along with stack name.

debuggin lambda functions locally -
run them like `sam local start-api -d 9229`. this is very similar to how we run nodejs apps like `nodemon app.js --inspect=9229`

after that open a debugger from chrome inspector and have it connect to localhost:9229 and then hit api. It will automatically stop at the first line and then stop at any line which has debugger; or breakpoint on it.
you can also run a debugger from webstorm. set one up under attach to node.js/chrome option. specify localhost:9229. then run it. then hit the api and it will run and auto stop at first line and debugger/breakpoints.

the same can also be done with `sam local invoke -d 9229 functioName`

`sam validate` will validate changes you make to template.yml

sam has built in support for codeDeploy. 

you can pipe events to lambda invokations like
`sam local generate-event dynamodb update | sam local invoke ReadDynamoDbFunction` 

I dont believe you need a new bucket for every sam app. the same bucket can be used accross many apps and functions since each file in the bucket is uniquely named and then referenced from package.yml

When the file is in yaml format, you can do `!Ref logicalName` to get the value of the resource named `logicalName`. you can also do `Ref: logicalName`

On the lambda web Ui console under actions there is export function which allows to download template.yml which reflects how the serverless app is currently confifured including its triggers and downstream things and policies etc. this is great to see how lambda builds this file and use the UI for cumbesome changes and then only hand-edit whats needed. 

note: even though `sam local start-api` supports hot reloading, once you have done `sam build`, now you have a folder called `.aws-sam` and now it will be used by the http server spinned of by `start-api` and this built code is not being updated as you are writing code so in essense there is no more hot reloading. to solve this you can either delete this folder or always be building it upon changes.

Install AWS Toolkit in to Webstorm. Its an EAP so you need manually add the URI for the plugin - https://plugins.jetbrains.com/plugins/eap/11349. After that if you search AWS Toolkit, it will show up. Then you can install it.

the outputs section of template.yml is optional and is used to:
- declate output values that can be imported in to aother stacks
- return in response to describe stack calls
- viewed on AWS cloudformation console

With AWS Toolkit, when you open a lambda function,  a lambda symbol appears which lets you quickly run any lambda function either locally or remotely. It provides option to send in events as request.

behind the scenese it just doing sam local invoke "FunctionName" --event event.json

run and debug configuration for each file from lambda is also now under edit configuration in webstorm

When lambda function is ran after edit config page, it does a build first and in this case it only build this function and gets rid of all other functions.