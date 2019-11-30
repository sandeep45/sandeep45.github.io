Fetch
Clean
Prepare
Train Model -> Needs algorithim and compute resources
evaluate the model
deploy the model
it is a continuos cycle. monitor inferences, collect "ground truth" and evaluate the model to identity drift. update the training data with new "ground truth" and retraining the model with new dataset.

Training Job -> S3 for training data, compute resources, S3 for job output (model artifacts), EC2 Container Registry where training code is stored.

reserve some memory for sagemaker otherwise training code me=ay take it all and trigger a crash.

Deploy Model -> S3 with model artificats and path to image that contains inference code.

Batch Transform Job -> takes a trained model and dataset in S3.  inferences go back to S3. it self manages resources required to drive all inferences. 
use case: get inferences upfront and then cache it for later use. also you can use it preprocess data to be used as training data of a new model.

to validate use a "holdout set" or "k-fold validation"
split data set in to "k" parts. then produce "k" models and for each model model use the other "k-1" as training data and kth data as hold out. then aggregate the models to generate the final mode.

High level python library - deploy and fit.
AWS SDK 

Gettings Started - Handwritten numbers images of single digit numbers
MNIST -> 50K training set, 10K validation set and 10K test set.
Algorithm -> XGBoost assigns data into 10 clusters.
S3 bucket to store training data
S3 bucket to store model artifacts that amazon sagemaker creates when it trains the model
Sagemaker Notebook instance to prepare and process data, train and deploy the model
Jupyter notebook to use with Sagemaker notebook to prepare and train and deploy.

To Train, deploy and validate we can use either:
1. high level amazon sagemaker python SDK -> recommended
2. AWS SDK for python (boto 3)

Sagemaker notebook instance is an EC2 compute instance on top of which the Jupyter Notebook App runs.

"Pickling" is the process of converrting a python object in to byte stream. aka Marshaling aka Searilizing
"unPickling" is the process which does the reverse where a byte stream is converted back to a python object.

Never unpickle code you dont trust.

Pickle.load rebuilds the object from a bytes stream
Pickle.dump writes the object as bytes to a file

urllib.request.urlretreive copies a url to a file

gzip module provides GzipFile Class, open, compress and decompress

with - keyword used when working with resources which need cleanup after the code using it finishes. it is syntactical sugar for try/finally

with expression [as variable]:
	with-block

it is handy when we want to run 2 related operations with a block of code between them. In this case its the opening the file and closing the file. In the middle we have a with-block.

tuples are an immulable list

%time is a magic command in jupyter - it gives wall time, CPU time and gives value of the expression

%matplotlib - magic command in jupyter - activates matplotlib interactive support

trainset[0] is numpy array with images
and trainset[1] is numpy array with labels

img = trainset[0][0] # this is the first image and its type is numpy.ndarray
label = trainset[1][0] # is the the label of this image

to see this image in ui:
%matplotlib inline
img_reshape = img.reshape((28,28))
imgplot = plt.imshow(img_reshape, cmap='gray')
plt.show()

tolist() - to convert a series to a list
so train_set[1].to_list() gives an array of labels

another way: 

y = [t for t in train_set[1]]
print(y[0:5])

this converts every element in the series to an array

the XGBoost algorithm expects CSV as input for training data.
The format we have form MNIST is numpy.array
so we need to convert it to CSV and then upload it to S3.

numpy.insert(arr, obj, values, axis=None)
Insert values along the given axis before the given indices.

numpy.savetxt(fname, X, fmt='%.18e', delimiter=' ', newline='n', header='', footer='', comments='# ', encoding=None)[source]
Save an array to a text file.

examples = np.insert(features, 0, labels, axis=1)
here it is inserting labels before each feature
so every row starts with the label value then all the feature values

from XGBoost Algorithim: For CSV training, the algorithm assumes that the target variable is in the first column and that the CSV does not have a header record. For CSV inference, the algorithm assumes that CSV input does not have the label column.

So far we have done work to get the data and prepare it for training the algorithm. Next step is train the model. This can be done by the Amazon Sagemaker Python SDK or AWS SDK for Python (Boto 3)

Amazon Sagemaker Python SDK is easier to use as its high level.

container = get_image_uri(boto3.Session().region_name, 'xgboost')
this gets us the xgboost model

the model will produce output, this could be a directory in the s3 bucket next to the training etc. data 
s3_output_location = 's3://{}/{}/{}'.format(bucket, prefix, 'xgboost_model_sdk')
this is setting up the location where the model output will go.

then the estimator method is called and is provided machine specs and where to put output and most importantly the container. this is image of the model to be used.

the result of this estimator methis our xg_boost model

next we can hyper tune it.

next we call fit on it. git needs s3 location of training and validation data
train_channel = sagemaker.session.s3_input(train_data, content_type='text/csv')
valid_channel = sagemaker.session.s3_input(validation_data, content_type='text/csv')

data_channels = {'train': train_channel, 'validation': valid_channel}
xgb_model.fit(inputs=data_channels,  logs=True)

We used AWS SagemMker Python SDK to:
1. setup container with algorithim of choice
2. call estimate with container, machine specs etc to get model, and where we want output
3. then we tuned the model
4. then we called fit on the model with input data which is s3 locations

The same can also be done via AWS SDK JavaScript, Python etc.
The SDK offers createTrainingJob method. This method will take all items which are:
- algo specification
- hyper params
- s3 for training and validation data
- s3 for output location
- resource config
- roleARN

The only thing we need from the High-level python SDK is the container location of our algorithm.
from sagemaker.amazon.amazon_estimator import get_image_uri
container = get_image_uri(boto3.Session().region_name, 'xgboost')
its value is 811284229777.dkr.ecr.us-east-1.amazonaws.com/xgboost:1
The registry path is <ecr_path>/xgboost:<tag> 
and for us-east-1 -	811284229777.dkr.ecr.us-east-1.amazonaws.com

so we could just compute this value too.

After .fit has finished we now have the models output in s3.
next step is to **deploy** this model with the data we have.
for 1 by 1 inference we can set up an endpoint
for large dataset inferences we need aws sagemaker batch transform

.deploy can be called on the same esimator model object to deploy what the model had built.
it returns a realTimePredictor object on which .predict can be called
but it laos sets up an endpoint which host's the model.
the endpoint can be found in sagemaker console under endpoints

the same can also be done via SDK by:
1. createModel -> this needs the data which after training was put in to s3 and the docker image location
2. createEndpointConfig -> give it the machine on which to deploy the endpoint and the model to deploy.
3. createEndpoint -> give it the config and creates the endpoint

next step is to **validate** the deployed endpoint and the realtime predictor object we have. 

to validate lets first get some test data

```
s3 = boto3.resource('s3')
test_key = "{}/test/examples".format(prefix)
s3.Bucket(bucket).download_file(test_key, 'test_data')
```

here using boto3 library we are dowbloading test data from s3.

lets now read first line of this file and see what it is

```
%matplotlib inline

data = np.loadtxt('test_data', delimiter=',')
img = data[0]
img_reshape = img.reshape((28,28))
imgplot = plt.imshow(img_reshape, cmap='gray')
plt.show()
```

so we just read enitre file back in to numpy array
then we are taking first record
reshaping and plotting it
we can see what number it is

now lets convert this first row back in array and pass it to predictor

```
csv_string = ','.join(['%.5f' % num for num in data[0]])
result = xgb_predictor.predict(csv_string)
print(result)
```

another way could be to just read first line of csv as csv and run it through predictor

```
with open('test_data', 'r') as f:
    single_test = f.readline()
    result = xgb_predictor.predict(single_test)
    print(result)
```

but all this was done by using the real time predictor. in reality we want to work with the endpoint.

```
runtime_client = boto3.client('runtime.sagemaker')
with open('test_data', 'r') as f:
    single_test = f.readline()
    response = runtime_client.invoke_endpoint(EndpointName = 'xgboost-2019-10-30-19-36-04-287',
                                         ContentType = 'text/csv',
                                         Body = single_test)
    result = response['Body'].read().decode('ascii')
    print('Predicted label is {}.'.format(result))

```
invoke_endpoint here is being used to call any endpoint by name, passing in data and getting answers.

Cleanup:
- endpoint -> internally also deletes the ML instances being used to support it
- endpoint config
- model
- notebook instance
- s3 bucket
- iam role
- cloudwatch logs

Now, lets add a Web interface to SageMaker endpoint

this is done by creating a lambda function, which imports boto3 and calls the smae code as above which is invoking the endpoint.

## Flow of Building an App

The very first thing is to collect the data. This is saved in S3. After that we need to decide which algorithm to use. Its documentation will tell how it wants its input, in what files, how to arrange them etc. 

Then we create a training job. As parameters to this job, we provide it the hyperparameters this algo wants, the algo's container uri, box to use for training, input data for the algo, output location and the name of the job. We watch the logs of the training job to see how well the training is going. We can to see loss go away and validation:mAP in the graphs go upwards. This is again specific to this model and documented.

Now we create our own model. For this we take the output of the training job, container uri of the algo, and name for our new model and build our own model.

Next we create an endpoint configuration, here we tell it what machine we will use as endpoint and what model by name we will deploy on that , how much traffic it will get etc and give this config a name. This is really only valuable if we are building multiple machines with different models and some load distribution strategy.

Next we create an endpoint. It just takes the config name and a new name for the endpoint.

Now to test the endpoint, we do invokeendpoint on sagemkerruntime. We give it the name of endpoint and data. It gives us back results. The structure for input and output is specific to the model used. 

Incremental Training or transfer learning is possible, again dependent on the algorithm. In the inputConfig specify another channel called 'model' and its data source would be the artifacts of the already trained model.

References: 
- https://docs.aws.amazon.com/sagemaker/latest/dg/algos.html
- https://github.com/awslabs/amazon-sagemaker-examples/blob/master/introduction_to_amazon_algorithms/object_detection_pascalvoc_coco/object_detection_image_json_format.ipynb
- https://docs.aws.amazon.com/sagemaker/latest/dg/incremental-training.html