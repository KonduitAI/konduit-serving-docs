# RELATIVE\_TO\_ABSOLUTE

`RelativeToAbsoluteStep` is used for a given set of points, list of point, bounding box} or list of bounding boxes that are defined in relative terms. For example, all values 0.0 to 1.0 in terms of fraction of image height/width, convert these to absolute values \(i.e., pixels\) using:

1. An input image, as specified via the image name configuration, OR
2. An image height, as specified by image height and image width configuration.

Note that an `ImageToNDArrayConfig` can be provided, to account for the fact that the original image may have been cropped before being passed into a network that produced the points or bounding boxes. If the `imageToNDArrayConfig` field is null, it is assumed no cropping has occurred.

| Configs | Descriptions |
| :--- | :--- |
| imageName | Optional - the name of the field in the input data containing the image to use \(to determine height/width\). |
| imageH | If `imageName` is not specified - height of the input image to use. |
| imageW | If `imageName` is not specified - width of the input image to use. |
| imageToNDArrayConfig | Optional - used to account for the fact that the image may have been cropped before being passed into a network that produced the points or bounding box. This allows for them to be offset, so the boxes or coordinates are specified in terms of the original image, not the cropped image. Refer `config` from [IMAGE\_TO_\__NDARRAY](image_to_ndarray.md). |
| toConvert | Optional - the name of the point, list of point, bounding box or list of bounding box fields to convert. If not set, the step will convert any/all fields of those types. |

