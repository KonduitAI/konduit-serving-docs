# CROP\_FIXED\_GRID

`CropFixedGridStep` is similar to the `CropGridStep` with the difference that the `x` or `y` location values are hard coded into the configuration, instead of coming dynamically from the input data instance.

| Configs | Descriptions |
| :--- | :--- |
| imageName | Name of the input image key from the previous step. If set to `null`, it will try to find any image in the incoming data instance. |
| points | A list of point of length 4, the corners, in order: `topLeft`, `topRight`, `bottomLeft`, `bottomRight`. |
| gridX | The number of grid segments between \(top left and top right\) and \(bottom left and bottom right\). |
| gridY |  The number of grid segments between \(top left and bottom left\) and \(top right and bottom right\) |
| coordArePixels | If `true`, the points are in pixels coordinates \(0 to width-1\) and \(0 to height-1\); if `false`, they are 0.0 to 1.0 \(fraction of image height/width\) |
| boundingBoxName | Name of the output bounding boxes key. |
| outputCoordinates | If `true`, the two lists are returned which contains the data of grid horizontal and vertical coordinates, respectively. |
| keepOtherFields | If `true`, other data key and values from the previous step are kept and passed on to the next step as well. |
| aspectRatio | If set, the smaller dimensions will be increased to keep the aspect ratio correct \(which may crop outside the image border\). |
| outputName | Name of the key of all the cropped output images from this step. |

