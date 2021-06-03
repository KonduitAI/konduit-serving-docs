# EXTRACT\_BOUNDING\_BOX

`ExtractBoundingStep` is a a pipeline step that extracts sub-images from an input image, based on the locations of input bounding boxes. Returns list of image for the cropped image regions.

| Configs | Descriptions |
| :--- | :--- |
| imageName | Name of the input image key from the previous step. If set to `null`, it will try to find any image in the incoming data instance. |
| bboxName | Name of the bounding boxes key from the previous step. If set to `null`, it will try to find any bounding box in the incoming data instance. |
| outputName | Name of the output key that will contain the output as images the input image region covered by the bounding boxes. |
| keepOtherFields | If `true`, other data key and values from the previous step are kept and passed on to the next step as well. Default is set to `true`. |
| aspectRatio | If set, the smaller dimensions will be increased to keep the aspect ratio correct which may crop outside the image border. |
| resizeH | If specified, the cropped images will be resized to the specified height. |
| resizeW | If specified, the cropped images will be resized to the specified width. |
| imageToNDArrayConfig | Used to account for the fact that n-dimensional array from `ImageToNDArrayConfig` may be used to crop images before passing to the network, when the image aspect ratio does not match the NDArray aspect ratio. This allows the step to determine the subset of the image actually passed to the network that produced the bounding boxes. Refer `config` from [IMAGE\_TO_\__NDARRAY](image_to_ndarray.md). |

