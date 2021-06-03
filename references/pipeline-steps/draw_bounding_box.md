# DRAW\_BOUNDING\_BOX

`DrawBoundingBoxStep` is a pipeline step that configures how to draw a bounding box onto an image. The bounding box data, that's to be drawn, is taken from the previous step's data instance.

| Configs | Descriptions |
| :--- | :--- |
| imageName | Name of the input image key from the previous step. If set to `null`, it will try to find any image in the incoming data instance. |
| bboxName | Name of the bounding boxes key from the previous step. If set to `null`, it will try to find any bounding box in the incoming data instance. |
| drawLabel | If `true`, then draw the class label on top of the bounding box. |
| drawProbability | If `true`, then draw the class probability on top of the bounding box. |
| classColors | Specifies the color of different classes or labels that are drawn. The color can be a hex or HTML string like `#788E87`, an `RGB` value like - `rgb(128,0,255)` or  it can be from a set of predefined HTML color names: `white`, `silver`, `gray`, `black`, `red`, `maroon`, `yellow`, `olive`, `lime`, `green`, `aqua`, `teal`, `blue`, `navy`, `fuchsia`, `purple`. |
| color | The default color to use in case a color for a label or class is not defined. The color can be a hex or HTML string like `#788E87`, an `RGB` value like - `rgb(128,0,255)` or  it can be from a set of predefined HTML color names: `white`, `silver`, `gray`, `black`, `red`, `maroon`, `yellow`, `olive`, `lime`, `green`, `aqua`, `teal`, `blue`, `navy`, `fuchsia`, `purple`. Default color is `lime`. |
| lineThickness | Line thickness to use to draw the bounding box \(in pixels\). The thickness of line is 1 by default. |
| scale | The scaling policy to use for scaling the bounding boxes. Default value is `NONE`. |
| resizeH | Height threshold to be used with the scaling policy. |
| resizeW | Width threshold to be used with the scaling policy. |
| imageToNDArrayConfig | Used to account for the fact that n-dimensional array from `ImageToNDArrayConfig` may be used to crop images before passing to the network, when the image aspect ratio doesn't match the NDArray aspect ratio. This allows the step to determine the subset of the image actually passed to the network. |
| drawCropRegion | If `true`, the cropped region based on the image array is drawn. Set to`false` by default. |
| cropRegionColor | Color of the crop region. Only used if `drawCropRegion = true`. |

