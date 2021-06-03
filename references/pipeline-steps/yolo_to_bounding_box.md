# YOLO\_BBOX

`YoloToBoundingBoxStep` is used to convert an NDArray for the predictions of a YOLO model to a list of bounding box.The NDArray is assumed to be in standard YOLO output format, after activation functions \(sigmoid/softmax\) have been applied. 

Input must be a float or double NDArray with shape `[minibatch, B*(5+C), H, W]` if `nchw` is `true` or `[minibatch, H, W, B*(5+C)]` if `nchw` is `false` where B is number of bounding box priors, C is number of classes, H is output/label height and W is output/label width.

Along the channel dimension for each box prior, we have the following values: 

* 0: px = predicted x location within grid cell, 0.0 to 1.0
* 1: py = predicted y location within grid cell, 0.0 to 1.0
* 2: pw = predicted width, in grid cell, for example 0.0 to H \(for example, pw = 2.0 -&gt; 2.0/W fraction of image\)
* 3: ph = predicted height, in grid cell, for example 0.0 to H \(for example, ph = 2.0 -&gt; 2.0/H fraction of image\)
* 4: c = object confidence - i.e., probability an object is present or not, 0.0 to 1.0
* 5 to 4+C = probability of class \(given an object is present\), 0.0 to 1.0, with values summing to 1.0

Note that the height/width dimensions are grid cell units - for example, with `416x416` input, 32 down sampling by the network we have `13x13` grid cells \(each corresponding to 32 pixels in the input image\). Thus, a center of X of 5.5 would be `xPixels = 5.5x32 = 176 pixels` from left. Widths and heights are similar: in this example, a width of 13 would be the entire image \(416 pixels\), and a height of 6.5 would be `6.5/13 = 0.5` of the image \(208 pixels\).

| Configs | Descriptions |
| :--- | :--- |
| input | Name of the input - optional. If not set, the input is inferred \(assuming a single NDArray exists in the input\). |
| output | Name of the input - optional. If not set, `bounding_boxes` is used |
| nchw | The data format - NCHW \(true\) or NHWC \(false\) as known as `channels first`for true or `channels last` for false. |
| threshold | The threshold, in range `0.0` to `1.0`. Any boxes with object confidence less than this will be ignored. Default value is `0.5`. |
| nmsThreshold | Non-max suppression threshold to use, to filter closely overlapping objects. Default value is `0.50`. |
| numClasses | Number of classes. Not required if `classLabels` are provided. |
| classLabels | Optional - names of the object classes. |
| keepOtherValues |  If `true`: keep all other input fields in the data instance. If `false`: only return the list of bounding box. Default is set to `true`. |

