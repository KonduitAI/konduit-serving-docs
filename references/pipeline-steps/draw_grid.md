# DRAW\_GRID

`DrawGridStep` is used to draw a grid on the specified image, based on the `x` or `y` coordinates of the corners, and the number of segments within the grid in both directions. The four corner coordinates are defined as points, and come from list of point targeted data, in the following order: top left, top right, bottom left, bottom right.

| Configs | Descriptions |
| :--- | :--- |
| imageName | Name of the input image key from the previous step. If set to `null`, it will try to find any image in the incoming data instance. |
| pointsName | Name of the list of points specifying the corners, in order: top left, top right, bottom left, bottom right. |
| gridX | The number of grid segments between \(top left and top right\) and \(bottom left and bottom right\). |
| gridY | The number of grid segments between \(top left and bottom left\) and \(top right and bottom right\). |
| coordsArePixels | If `true`, the lists are in pixels coordinates, not from 0 to 1. |
| borderColor | Color of the border. The color can be a hex/HTML string like `#788E87`, an `RGB` value like - `rgb(128,0,255)` or  it can be from a set of predefined HTML color names: `white`, `silver`, `gray`, `black`, `red`, `maroon`, `yellow`, `olive`, `lime`, `green`, `aqua`, `teal`, `blue`, `navy`, `fuchsia`, `purple`. |
| gridColor | Color of the grid. The color can be a hex/HTML string like `#788E87`, an `RGB` value like - `rgb(128,0,255)` or  it can be from a set of predefined HTML color names:`white`, `silver`, `gray`, `black`, `red`, `maroon`, `yellow`, `olive`, `lime`, `green`, `aqua`, `teal`, `blue`, `navy`, `fuchsia`, `purple`. |
| borderThickness | Line thickness to use to draw the border in pixels. Default value is `1`. |
| gridThickness | Line thickness to use to draw the border in pixels. If `null` then the same value as the `borderThickness` is used. |

