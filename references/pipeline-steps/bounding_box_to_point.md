# BOUNDING\_BOX\_TO\_POINT

`BoundingBoxToPointStep` is a pipeline step in extracting the bounding box to point including center, top left, top right, bottom left and bottom right. 

| Configs | Descriptions |
| :--- | :--- |
| bboxName | Name of the bounding boxes key from the previous step. If set to `null`, it will try to find any bounding box in the incoming data instance. |
| outputName | Name of the point extracted from the input bounding box. If `null`, the input field name is used. |
| keepOtherFields |  If `true`, other data key and values from the previous step are kept and passed on to the next step as well. Default value is `true`. |
| method | The method can choose from the following methods of converting the bounding box to a point: `TOP_LEFT`, `TOP_RIGHT`, `BOTTOM_LEFT`, `BOTTOM_RIGHT`, `CENTER`. Set to `CENTER` by default. |

