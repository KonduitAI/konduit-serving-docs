# FRAME\_CAPTURE

`CameraFrameCaptureStep` is a pipeline step that specifies an input that's taken from a camera feed.

| Configs | Descriptions |
| :--- | :--- |
| camera | ID of the camera from which the input is taken from. Each system cameras is assigned an ID, which is usually 0 for the first device, 1 for the second and so on. Default value is `0` which is first device. |
| width | Width of the incoming image frame. This will scale the original resolution width to the specified value. Default is set to `640`. |
| height | Height of the incoming image frame. This will scale the original resolution height to the specified value. Default is set to `480`. |
| outputKey | Name of the output key that will contain and carry the image frame data to the later pipeline steps. Default is `image`. |

