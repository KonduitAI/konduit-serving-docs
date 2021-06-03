# DRAW\_HEATMAP

`DrawHeatMapStep` is a a pipeline step that configures how to draw a 2-D heat map on an image.

| Configs | Descriptions |
| :--- | :--- |
| points | Name of the input data fields containing the points used for the heat map. Accepts both single points and lists of points. Accepts both relative and absolute addressed points. |
| radius | Size of area influenced by a point. |
| fadingFactor | Fading factor. 0 is no fade and 1 is instant fade. |
| image | An optional field, specifying the name of the image to draw on. |
| opacity |  Opacity of the heat map. Between 0 and 1. 0 is fully transparent and 1 is fully opaque. Default is 0.5. |
| width | Must be provided when `image` is not set. Used to resolve position of points with relative addressing \(dimensions between 0 and 1\). |
| height | Must be provided when `image` is not set. Used to resolve position of points with relative addressing \(dimensions between 0 and 1\). |
| outputName | Name of the output image. Default name is set as `image`. |
| keepOtherValues | `true` by default. If `true`, copy all the other \(non-converted/non-image\) entries in the input data to the output data. |
| imageToNDArrayConfig | Used to account for the fact that n-dimensional array from `ImageToNDArrayConfig` may be used to crop images before passing to the network, when the image aspect ratio doesn't match the NDArray aspect ratio. This allows the step to determine the subset of the image actually passed to the network. Refer `config` from [IMAGE\_TO_\__NDARRAY](image_to_ndarray.md). |

