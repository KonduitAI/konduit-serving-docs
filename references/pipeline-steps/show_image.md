# SHOW\_IMAGE

`ShowImageStep` is a pipeline step that configures how to show or render an image from a previous step in an application frame. Usually only used for testing and debugging locally, not when serving from HTTP/GRPC etc endpoints.

| Configs | Descriptions |
| :--- | :--- |
| imageName | Name of the incoming input image key. Default is `image`. |
| displayName | Image display name. Default is `Image`. |
| width | Height of the displayed image frame. If `null`: same size as image is used |
| height | Width of the image. If `null`: same size as the image |
| allowMultiple | Allow multiple images to be shown. Default is set to `false`. |



