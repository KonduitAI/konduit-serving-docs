# DRAW\_FIXED\_GRID

`DrawFixedGridStep` is a pipeline step that draws a grid on an image. This is similar to `DrawGridStep` but the corner `x` or `y` location values are hard coded into the configuration \(via points\), instead of coming dynamically from the input data instance.

| Configs | Descriptions |
| :--- | :--- |
| imageName | Name of the input image key from the previous step. If set to `null`, it will try to find any image in the incoming data instance. |
| points | A list of point of length 4, the corners, in order: top left, top right, bottom left, bottom right. |
| gridX | The number of grid segments between \(top left and top right\) and \(bottom left and bottom right\). |
| gridY | The number of grid segments between \(top left and bottom left\) and \(top right and bottom right\). |
| coordArePixels | If `true`, the points are in pixels coordinates \(0 to width-1\) and \(0 to height-1\). If `false`, they are 0.0 to 1.0 \(fraction of image height or width\). |
| borderColor | Color of the border. The color can be a hex/HTML string like `#788E87`, an `RGB` value like - `rgb(128,0,255)` or  it can be from a set of predefined HTML color names: `white`, `silver`, `gray`, `black`, `red`, `maroon`, `yellow`, `olive`, `lime`, `green`, `aqua`, `teal`, `blue`, `navy`, `fuchsia`, `purple`. |
| gridColor | Color of the grid. If not set, the border color will be used. The color can be a hex/HTML string like `#788E87`, an `RGB` value like - `rgb(128,0,255)` or  it can be from a set of predefined HTML color names: `white`, `silver`, `gray`, `black`, `red`, `maroon`, `yellow`, `olive`, `lime`, `green`, `aqua`, `teal`, `blue`, `navy`, `fuchsia`, `purple`. |
| borderThickness | Line thickness to use to draw the border in pixels. Default value is `1`. |
| gridThickness | Line thickness to use to draw the border in pixels. If `null` then the same value as the `borderThickness` is used. |

