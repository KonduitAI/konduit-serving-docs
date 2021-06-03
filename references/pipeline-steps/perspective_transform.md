# PERSPECTIVE\_TRANSFORM

| Configs | Descriptions |
| :--- | :--- |
| inputNames | A list of names of the input fields to process. May be points, bounding boxes, images, or lists of these. If input names are not set, the fields of these types will be inferred automatically. |
| outputNames | A list of names of the output field. If not set, the output has the same name as the input field. |
| sourcePointsName | List of 4 source points: top left, top right, bottom left, bottom right. |
| targetPointsName | List of 4 target points: top left, top right, bottom left, bottom right. |
| referenceImage | When a `referenceImage` is provided, the transform will be adjusted to ensure the entire transformed image fits into the output image \(up to `4096x4096`\). Can also reference a list of images, in which case, only the first image is used as the reference.The adjustment is applied to all inputs of the step. |
| referenceWidth | When a reference width and height are provided, the transform will be adjusted to make sure the entire area fits into the output image \(up to `4096x4096`\). The adjustment is applied to all inputs of the step. |
| referenceHeight | When a reference width and height are provided, the transform will be adjusted to make sure the entire area fits into the output image \(up to `4096x4096`\). The adjustment is applied to all inputs of the step. |
| sourcePoints | Takes exactly 4 source points: top left, top right, bottom left, bottom right. |
| targetPoints | Takes exactly 4 target points: top left, top right, bottom left, bottom right. |
| keepOtherFields | If `true`, other data key and values from the previous step are kept and passed on to the next step as well. Default value is set to `true`. |

