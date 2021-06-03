# IMAGE\_RESIZE

`ImageResizeStep` is a pipeline step that resizes an image, scaling up or down as needed to comply with the specified output height/width. Usually, both height and width are specified. However, if only one is specified, the other value is calculated based on the aspect ration of the input image. When both height and width are specified, and the aspect ratios of the input doesn't match the aspect ratio of the output, \(for example, 100x200 in, 200x200 out\) the `aspectRatioHandling` setting is used to determine how to handle this situation. 

Note that the names of the inputs data fields to resize may or may not be specified. If no value is provided for `inputNames` configuration, all input images fields in the input Data instance will be resized, regardless of name. If `inputNames` is specified, only those fields with those names will be resized.

| Configs | Descriptions |
| :--- | :--- |
| inputNames | Name of the input keys whose values contain images from the previous step. |
| height | Resize height. |
| width | Resize width. |
| aspectRatioHandling | An enum to define how to handle the aspect ratio when the aspect ratio doesn't match with that of the input image. Default value is `STRETCH`.  |

