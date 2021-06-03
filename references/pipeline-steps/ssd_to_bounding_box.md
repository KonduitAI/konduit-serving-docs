# SSD\_TO\_BBOX

`SSDToBoundingBoxStep` is a pipeline step that configures extraction of bounding boxes from a \(Single-Shot Detector\) SSD model output.

| Configs | Descriptions |
| :--- | :--- |
| classLabels | A list of class labels |
| keepOtherValues | `true` by default. If `true`, other data key and values from the previous step are kept and passed on to the next step as well.  |
| threshold | Threadshold to the output of the SSD models for fetching bounding boxes for. Default threshold is `0.5`. |
| scale | An optional way to increase the size of the bounding boxes by some fraction. If specified, a value of `1.0` is equivalent to no scaling. A scale of `2.0` means the center is unchanged, but the width and height are now two times larger than it would be. |
| aspectRatio | An optional way to control the output shape \(aspect ratio\) of the bounding boxes. Defined in terms of width or height. If specified, an aspect ratio of `1.0` gives a square output; an aspect ratio of `2.0` gives twice as wide as it is high. Note that for making the output the correct aspect ratio, one of the height or width will be increased; the other dimension will not change. The pre-aspect-ratio-corrected box will be contained fully within the output box. |
| outputName | Output key name where the bounding box will be contained in. The default name is `bounding_box`. |

