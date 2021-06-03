# DRAW\_POINTS

`DrawPointsStep` is one of pipeline step that configures how to draw 2-D points on an image.

| Configs | Descriptions |
| :--- | :--- |
| noClassColor | Color belongs to non-label data. `lime` is default color for `noClassColor`. |
| classColors | This is an optional field which specifies the mapping of colors to use for each class. The color can be a hex or HTML string like `#788E87`, an `RGB` value like - `rgb(128,0,255)` or  it can be from a set of predefined HTML color names: `white`, `silver`, `gray`, `black`, `red`, `maroon`, `yellow`, `olive`, `lime`, `green`, `aqua`, `teal`, `blue`, `navy`, `fuchsia`, `purple`. |
| points | Name of the input data fields containing the points to be drawn. Accepts both single points and lists of points. Accepts both relative and absolute addressed points. |
| radius | Optional. Point radius on drawn image in pixels. Default is `5` pixels. |
| image | An optional field, specifying the name of the image to be drawn on. |
| width | Must be provided when `image` is not set. Used to resolve position of points with relative addressing \(dimensions between 0 and 1\). |
| height | Must be provided when `image` is not set. Used to resolve position of points with relative addressing \(dimensions between 0 and 1\). |
| imageToNDArrayConfig | Used to account for the fact that n-dimensional array from `ImageToNDArrayConfig` may be used to crop images before passing to the network, when the image aspect ratio does not match the NDArray aspect ratio. This allows the step to determine the subset of the image actually passed to the network. Refer `config` from [IMAGE\_TO_\__NDARRAY](image_to_ndarray.md). |
| ouputName | Name of the output image. Default is named as `image`. |

