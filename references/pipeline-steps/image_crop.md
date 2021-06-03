# IMAGE\_CROP

`ImageCropStep` is used to crop an image to the specified rectangular region. The crop region may be specified in one of two ways:

1. Via a bounding box, or
2. Via a list of points of length 2, containing the top-left and bottom-right crop locations.

These may be specified statically \(i.e., fixed crop region\) via `cropBox` or `cropPoints` property, or dynamically via `cropName` \(which may specify a bounding box or list of point in the input data instance\).

Furthermore, the bounding box and corner point coordinates may be specified in terms of either pixels or fraction of image - specified via the `coordsArePixels` property. Note that if the crop region falls partly outside the input image region, black padding will be added as necessary to keep the requested output size.

| Configs | Descriptions |
| :--- | :--- |
| imageName | Name of the `Image` or in the list, `List<Image>` field to crop. |
| cropName | Name of the input Data field used for dynamic cropping. May be a `BoundingBox` or `List<Point>`. |
| cropPoints | Static crop region defined as a list of point, `List<Point>`. |
| cropBox | Static crop region defined as a bounding box, `BoundingBox`. |
| coordsArePixels | Weather the crop region \(`BoundingBox` / `List<Point>`\) are specified in pixels, or fraction of image. |

