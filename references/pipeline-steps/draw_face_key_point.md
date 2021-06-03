# DRAW\_FACE\_KEY\_POINT

| Configs | Descriptions |
| :--- | :--- |
| imageToNDArrayConfig | Class to transform image to array. Refer to available configuration from [IMAGE\_TO_\__NDARRAY](image_to_ndarray.md). |
| resizeH | Height value to resize |
| resizeW | Width value to resize |
| drawFaceBox | To draw bounding box around face. If set to `true`: a bounding box will be drawn around the face. If `false`: no bounding box will be drawn. |
| faceBoxColor | Specifies the color of bounding box around face. The color can be a hex or HTML string like `#788E87`, an `RGB` value like - `rgb(128,0,255)` or  it can be from a set of predefined HTML color names: `white`, `silver`, `gray`, `black`, `red`, `maroon`, `yellow`, `olive`, `lime`, `green`, `aqua`, `teal`, `blue`, `navy`, `fuchsia`, `purple`. Default is `lime`. |
| pointColor | Specifies the color of face key points. The color can be a hex or HTML string like`#788E87`, an `RGB` value like -`rgb(128,0,255)` or  it can be from a set of predefined HTML color names: `white`, `silver`, `gray`, `black`, `red`, `maroon`, `yellow`, `olive`, `lime`, `green`, `aqua`, `teal`, `blue`, `navy`, `fuchsia`, `purple`. Default is `red`. |
| pointSize | Size of face key points. Default value is 1. |
| scale | Scaling enum, which can be `NONE`, `AT_LEAST`, `AT_MOST` |
| landmarkArray | Field name, which contain array of key points from previous step. |
| image | An optional field, specifying the name of the image to be drawn on. |
| outputName | Name of the key of face key points from this step. Default is set to `image`. |

