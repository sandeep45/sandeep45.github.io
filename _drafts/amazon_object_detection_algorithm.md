## amazon_object_detection_algorithm

file mode - training files - application/x-recordio OR image/png image/jpef application/x-image
pipe mode - training files - application/x-recordio or image files by using augmented manifest format.
Recommended - Apache MXNet RecordIO

For Inference -> application/x-image

#### When training with image format we need to specify:

- train
- validation
- train_annotation
- validation_annotation
Here train and validation are individual image data
and annotation channels are json files

Example json file
```
{
    file: 'path/boo1.jpg,
    image_size: [
        {
            width: 200,
            height: 200,
            depth: 3
        }
    ],
    annotations: [
        {
            class_id: 0,
            left: 0, top: 0, width: 10, height: 10
        }
    ],
    categories: [
        {
            class_id: 0,
            name: dog
        }
    ]
}
```

Each image needs 1 json file whose name is the same as that of the image. So for boo1.png we have boo1.json.
Properties of the json file:
- file -> relative path to image
- image_size -> overall image dimensions
- annotations specifies bounding boxes for various objects
- categories has mappings between class index and class name

Other options of training are recordIO and Augmented Manifest Image Format.

Algorithm identifies and locates all instances of objects in an image from a known collection of object categories.

required parameters for `CreateTrainingJob` Method:
- num_classes
- num_training_samples

Inference: 
takes image/jpeg or image/png
returns a json with all detected objects and their scores and bounding boxes