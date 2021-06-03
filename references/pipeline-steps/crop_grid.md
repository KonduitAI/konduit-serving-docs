# CROP\_GRID

`CropGridStep` is a pipeline step that crops sub images out of a larger image, based on a grid. The four corner coordinates are defined as points, and come from list of point of targeted data, in the following order: top left, top right, bottom left, bottom right. The output contains a list of cropped images from the grid, in order: \(row,col\) = \(0,0\), \(0, 1\), ..., \(0, C-1\), ..., \(R-1, C-1\).

| Configs | Descriptions |
| :--- | :--- |
| imageName | Name of the input image key from the previous step. If set to `null`, it will try to find any image in the incoming data instance. |
| pointsName | Name of the list points specifying the corners, in order: `topLeft`, `topRight`, `bottomLeft`, `bottomRight`. |
| gridX | The number of grid segments between \(top left and top right\) and \(bottom left and bottom right\). |
| gridY | The number of grid segments between \(top left and bottom left\) and \(top right and bottom right\). |
| coordsArePixels | If `true`, the lists are in pixels coordinates, not from 0 to 1. |
| boundingBoxName | Name of the output bounding boxes key. |
| outputCoordinates | If `true`, the two lists are returned which contains the data of grid horizontal and vertical coordinates, respectively. |
| keepOtherFields | If `true`, other data key and values from the previous step are kept and passed on to the next step as well. |
| aspectRatio | If set, the smaller dimensions will be increased to keep the aspect ratio correct \(which may crop outside the image border\). |
| outputName | Name of the key of all the cropped output images from this step. |

