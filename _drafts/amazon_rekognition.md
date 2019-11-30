Amazon Recognition

search images and videos to discover objects
compare live images against reference image for face based authentication
emotional and sentimental analysis of a face like sad, happy, surprise etc.
search against a collection of faces
detect unsafe content
celebrity detection
text detection - e.g. license plate of vehicles

S3 event -> Lambda -> Recognition API

Labels: objects, events, concepts, activites (video only).

SDK and HTTP(CLI). recommend SDK.

** [STORED] IMAGES **

aws rekognition detect-labels --image "S3Object={Bucket=video-playback,Name=pic_6.jpg}" | jq
aws rekognition detect-labels --image "{\"S3Object\":{\"Bucket\":\"video-playback\",\"Name\":\"pic_6.jpg\"}}"

```javascript
const rekognition = new AWS.Rekognition({apiVersion: '2016-06-27'});
const inputParams = {
    Image: {
      S3Object: {
        Bucket: "video-playback",
        Name: 'mjpeg/pic_6.jpg'
      }
    }
  };
rekognition.detectLabels(inputParams, (err, data) => {
	if(err) logger.error('there has been an error detecting faces!!', err);
	console.log('results: \n', util.inspect(data, {depth: 5}));
});
```

Bounding Box measurements are percentage based.
so if it says top: 0.2 and the height of the image is 100px, then top is at 20px.
each value is between 0 and 1 and is a ratio to the overall image
so if it says height: 0.4 then the height of the bounding box is 40% of the overal image size.

when sending file via JS SDK no need to base 64 encode, just send buffer of file's binary data.

```
const rekognition = new AWS.Rekognition({apiVersion: '2016-06-27'});
const file = await new AWS.S3().getObject({Bucket: 'video-playback', Key: 'mjpeg/pic_6.jpg'}).promise();
const inputParams = {
    Image: {
      Bytes: file.Body
    }
  };
rekognition.detectLabels(inputParams, (err, data) => {
	if(err) logger.error('there has been an error detecting faces!!', err);
	console.log('results: \n', util.inspect(data, {depth: 5}));
});
```

** STORED VIDEOS **

Video Analysis
startLabelDetection takes s3 of videom notification channle of SNS and a role which allows rekognition to access SNS
SNS can notify SQS
We can poll SQS and get finishes jobs along with their job id and status.
we can delete the job notification off SQS using the ReceiptHandle
getLabelDetection call takes jobId, and by default gives 1000 labels and sorts them by time.
result has 1000 max results and token for pagination.

On video side, analysis are async so you have to call startLabelDetection, then have it post to SNS when its done. After its done, you can call getLabelDetection.
Get operations return when entities are detected throughout the input. this is the time marker.

start video analysis by passing in S3 object of the video file, a clientRequestToken for idempotency and NotificationChannel which is a SNS channel which will be notified when video analysis is done.
the response will be a jobId. Using this id, we can getResults.

SNS can then let a SQS or Lambda know when the analysis are done. The payload will have jobId.

We can now have the lambda get analysis or be polling the SQS and then get analysis with jobId.

Camera -> S3 -> Lamabda (Start Label) -> Recognition -> SNS -> Lambda (Get Label) -> DynamoDB -> Website

Lambda needs s3 access eventhough its just passing the s3 information to recognition api
since lambda can get multiple records, if you do forEach on records, beware that it may not work how you expect it to with async/await

Lambda also needs access to dynamoDB to be able to write to it.

You setup a lambda to be called by setting its trigger on the left part of the configuration section. Nothing needs to be done in S3. Its in lambda that we say to trigger itself from S3.
The same happens for the 2nd Lambda function which is triggered of a SNS. Although after doing this it will show up as a subscriber in SNS. To test things you can also add an email address in SNS.

DynamoDB putItem will create and update both. it will replace all old values. 

** STREAMING VIDEOS **

Amazon Recognition Video has to read from Kinesis Video Stream and write to Kinesis Data Stream.
To give Recognition Video access, prepend the name of streams with AmazonRekognition. 
Then an Service Role which uses a reko policy state which gives access to all streams which start with AmazonRekognition.


Create a StreamProcessor with:
1. Kinesis video stream as input
2. Kinesis data stream as output 
3. IAM Service role which gets access to both
4. Settings with Face Collection

Then Start/Stop the stream processor. 

To put data from Camera to Video Stream which is input use producer library as PutMedia API is limited to Java & C++. For RaspberryPi there are pre-built C++ sample apps to stream camera (/dev/video0) to any stream by name.

Collections -> Stored server side. give it a face, the detected information is stored here server side. With a collection, we can then search images, videos, streams etc. for known faces.

IndexFaces -> Storage based API. does two things:
1. detects faces in an image
2. persists information about facial features into a collection. it doesnt store the actual image, but extracted facial features as a feature vector.
3. accepts passing in of an optional ExternalImageId. It is associated with all faces it detects in the image and is returned when ListFaces is called.
4. By passing in 1 as MaxFaces, it will only index the largest face.
5. returns an array of metadata for all detected faces. This has the FaceId and ImageId.
6. Input image can be passed in as base64 bytes of reference to S3 object.
7. pass collection id to IndexFaces

Request:

{
   "CollectionId": "string",
   "ExternalImageId": "string",
   "Image": { 
      "Bytes": blob, // OR S3 Obj
      "S3Object": { 
         "Bucket": "string",
         "Name": "string",
         "Version": "string"
      }
   },
   "MaxFaces": number, // 1
}

Response:

{
	"FaceRecords": [ 
		{
			"Face": {
				"FaceId": string,
				"ExternalImageId": string,
				"ImageId" : string
			}
		}
	]
}


To search we have SearchFacesByImage, StartFaceSearch/GetFaceSearch & CreateStreamProcessor. All of them take a collection id.

StreamProcessor will place JSON frame record on Kinesis data stream for each analyzed frame. Not its not analysing all frames passed via video stream. 
The JSON frame record has informatin on
1. which video stream fragement, the frame is in
2. where the frame is, inside the framegement
3. the faces that are recognized in that frame.

"InputInformation": {
    "KinesisVideo": {
      "StreamArn": "arn:aws:kinesisvideo:us-west-2:nnnnnnnnnnnn:stream/stream-name",
      "FragmentNumber": "91343852333289682796718532614445757584843717598",
      "ServerTimestamp": 1510552593.455,
      "ProducerTimestamp": 1510552593.193,
      "FrameOffsetInSeconds": 2
    }
  },
	"MatchedFaces": [
        {
          "Similarity": 88.863960,
          "Face": {
            "BoundingBox": {
              "Height": 0.557692,
              "Width": 0.749838,
              "Left": 0.103426,
              "Top": 0.206731
            },
            "FaceId": "ed1b560f-d6af-5158-989a-ff586c931545",
            "Confidence": 99.999201,
            "ImageId": "70e09693-2114-57e1-807c-50b6d61fa4dc",
            "ExternalImageId": "matchedImage.jpeg"
          }
        }
      ]

FaceSearchResponse -> faces in the streaming video that match collection. -> contains DetectedFaces -> contains array of MatchedFaces

Need to have mapping technique to connect frames in kinesis video with frames in kinesis data stream

Processing data off of Kinesis Data Stream
Kinesis Data -> Lambda
Http/2 connection
default lambda invokes function as soon as records are available on the stream.
with a batch window setup, lambda continues to read records off the stream and invokes the fuction with the entire batch.
Lambda is mapped to to a data stream(shard-iterator) or to a consumer of a stream(enhanced fan-out).

shard iterator -> lambda polls each shared of the kinesis stream and processes batches.
data stream consumber -> RegisterStreamConsumer - http/2 

lambda processes records from a shard with records in order. for more throughput add more shards and ensure that lambda function can scale concurrently. Lambda concurrency should match or increase number of shards.

Role for lambda - AWSLambdaKinesisExecutionRole

Designer section of Lamda Console can be used to add a trigger. The same can also be done by calling CreateEventSourceMapping function on the lambda API

Custom App -> Kinesis Data Stream -> Lambda Event Source Mapping -> Lambda (Execution Role) (Buffers) -> Lambda Function Executed

Seems like the EventTrigger part of lambda is doing the heavy lifting here of getting data to us from Kinesis.
MatchedFaces is only populated when it matches a face.

Timestamp Kinesis Video Stream Frame = ProducerTimestamp + FrameOffsetInSeconds

FragementNumber = this is the fragement which contains the frame we are referring to in JSON.
ProducerTimestamp = producer side unix timestamp of the fragement.
FrameOffsetInSeconds = the offset of the frame in seconds inside the fragement

InputInformation:{
  KinesisVideo:{
    StreamArn:'arn:aws:kinesisvideo:us-east-1:616179486975:stream/AmazonRekognitionVideoPlayback/1571259851519',
    FragmentNumber:'91343852333181447436129420019654522058332548706',
    ServerTimestamp:1571343137.532,
    ProducerTimestamp:1571343137.302,
    FrameOffsetInSeconds:1.0019999742507935
  }
},
StreamProcessorInformation:{
  Status:'RUNNING'
},
FaceSearchResponse:[
  {
    DetectedFace:{
      BoundingBox:{
        Height:0.28164068,
        Width:0.17375536,
        Left:0.5173088,
        Top:0.25113347
      },
      Confidence:100,
      Landmarks:[
        {
          X:0.54102266,
          Y:0.33397445,
          Type:'eyeLeft'
        },
        {
          X:0.6236772,
          Y:0.32471767,
          Type:'eyeRight'
        },
        {
          X:0.5531065,
          Y:0.45175633,
          Type:'mouthLeft'
        },
        {
          X:0.6213913,
          Y:0.44382644,
          Type:'mouthRight'
        },
        {
          X:0.5785355,
          Y:0.37454572,
          Type:'nose'
        }
      ],
      Pose:{
        Pitch:18.803076,
        Roll:-6.57956,
        Yaw:-5.5567317
      },
      Quality:{
        Brightness:70.57527,
        Sharpness:96.61495
      }
    },
    MatchedFaces:[
      {
        Similarity:95.62916,
        Face:{
          BoundingBox:[
            Object
          ],
          FaceId:'a5e84def-f5a0-410e-9e98-290f7950feac',
          Confidence:100,
          ImageId:'4afb2989-08e2-3799-94cf-5ec51cfa90b2',
          ExternalImageId:'sunny'
        }
      }
    ]
  }
]
}

aws rekognition describe-stream-processor --name myProcessor 
and ensure thats its off now

adding images of multiple people with same external id, will map all of their occurences to the same external id.

Best Practices

Yaw - horizontal rotational angle of the camera
Pitch - angle up and down (above and below hoirzon)
Roll - Rotation angle around the lens axis
An image positioned in Yaw=0 and Pitch=0 means the source image optical axis is in the canvas center.

- sending an image as Bytes is faster than uploading to S3 and then referencing it, unless image is already in S3.
- pitch should be less than 30 degress face down and 45 degrees face up
- yaw should be less that 45 degrees in either direction
- use at least 5 images per person. use similar external ids like joe_1.jpg, joe_2.jpg, joe_3.jpg, joe_4.jpg

DetectLabels for Images
StartLabelDetection & GetLabelDetection for Videos

Face Detection 
Face Recognition

client.detectFaces(params, (err, data) => {});
client.compareFaces(params, (err, data) => {}); -> takes 2 images

startFaceDetection/getFaceDetection -> to check a video for a face image.

indexFaces -> Used to detect faces in an image and store information about facial features that are detected in a collection.
Note actual imagge/face bytes are not stores but instead extracts facial attributes into a feature vector for each face and then stores that.
CreateCollection -> create a collection and then pass the collection ARN when calling indexFaces
SearchFacesByImages and StartFaceSearch/GetFaceSearch and CreateStreamProcessor

use DetectFaces to confirm that only 1 image is detected in an image, then call indexFaces for that image.

searchFaces -> can be used similarly to SearchFacesByImage. you add both or more faces to a collection then call searchFaces with faceId and it tells if the passed in person matches any of the faces already in the collection. for this to work we should have saved the FaceId on our end when it was added to the collection. 
SearchFacesByImage -> ipnut image is s3 or bytes and then its used to search a collection of images. similar to searchFaces, but doesnt require FaceId.
compareFaces -> takes largest face from source file and compares with up to 100 faces in the target file

Detect Text -> gets the content (DetectedText), relations between words and lines(id & parent id) and bounding box (geometry).

streamProcessor gave us in kinesis data stream:

- ServerTimestamp
- ProducerTimestamp
- FragementNumber
- FrameOffsetInSeconds
- DetectedFace
- MatchedFaces

syncing playing of a kinesis video stream with rekognition data we have in dynamo db.
two ways:

1. get data's ProducerTimestamp and FrameOffsetInSeconds. add them to get time.
then look at what time was played and current offset time of video. add them. 
when time matches we have synced data.

2. HLS video.js is setup to show fragement number. this means the media playlist which lists all the piecces, will have FragmentNumber in QS of every mp4 file. then somehow when video is playing, you need to get which fragement is playing and time offset. and then match it to get synced data.


p.currentTime(); p.remainingTime(); p.duration();
p.on('timeupdate', () => {})
p.play(); p.pause(); p.paused();
