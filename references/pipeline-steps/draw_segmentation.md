# DRAW\_SEGMENTATION

`DrawSegmentationStep` is a pipeline step that configures how to draw a segmentation mask, optionally on an image.

| Configs | Descriptions |
| :--- | :--- |
| classColors | This is an optional field which specifies the list of colors to use for each class. The color can be a hex or HTML string like `#788E87`, an `RGB` value like - `rgb(128,0,255)` or  it can be from a set of predefined HTML color names: `white`, `silver`, `gray`, `black`, `red`, `maroon`, `yellow`, `olive`, `lime`, `green`, `aqua`, `teal`, `blue`, `navy`, `fuchsia`, `purple`. |
| segmentArray | Name of the NDArray with the class indices, 0 to numClasses-1. In shape of `[1, height, width]`. |
| image | An optional field, specifying the name of the image to draw the segmentation classes onto. If not provided, the segmentation classes are drawn onto a black background image. |
| outputName | Name of the output image. Default is named as `image`. |
| opacity | An optional field, that is used only used when `image` key name is set. This specifies, the opacity, between `0.0` and `1.0`, of the mask to draw on the image. Default value of `0.5` is used if it's not set. Value of 0.0 is fully transparent and 1.0 is fully opaque. |
| backgroundClass | An optional field, specifying a class that's not to be drawn. If not set, all classes will be drawn. |
| imageToNDArrayConfig | Used to account for the fact that n-dimensional array from `ImageToNDArrayConfig` may be used to crop images before passing to the network, when the image aspect ratio does not match the NDArray aspect ratio. This allows the step to determine the subset of the image actually passed to the network that produced the segmentation prediction to be drawn. Refer `config` from [IMAGE\_TO_\__NDARRAY](image_to_ndarray.md). |

