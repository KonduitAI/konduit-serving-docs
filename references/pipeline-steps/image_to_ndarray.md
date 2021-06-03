# IMAGE\_TO\_NDARRAY

`ImageToNDArrayStep` is a `PipelineStep` for converting images to n-dimensional arrays. The exact way that images are converted is highly configurable \(formats, channels, output sizes, normalization, etc\).

| Configs | Descriptions |
| :--- | :--- |
| config | Configuration for how conversion should be performed. |
| keys | May be null. If non-null, these are the names of images in the Data instance to convert. |
| outputNames | May be null. If non-null, the input images are renamed to this in the output Data instance after conversion to n-dimensional array. |
| keepOtherValues | True by default. If true, copy all the other \(non-converted/non-image\) entries in the input data to the output data. |
| metadata | False by default. If true, include metadata about the images in the output data. For example, if/how it was cropped, and the original input size. |
| metadataKey | Sets the key that the metadata will be stored under. Not relevant if metadata == `false`. Default is `@ImageToNDArrayStepMetadata` |

`ImageToNDArrayConfig` is configuration for converting an image into n-dimensional array. This configuration is used in `config` from `ImageToNDArrayStep`, for example:

```text
inferenceConfiguration.pipeline(SequencePipeline.builder()
                .add(new ImageToNDArrayStep() //add ImageToNDArrayStep() into pipeline to set image to NDArray for input
                        .config(new ImageToNDArrayConfig() //image configuration
                                .width(28)
                                .height(28)
                                .dataType(NDArrayType.FLOAT)
                                .aspectRatioHandling(AspectRatioHandling.CENTER_CROP)
                                .includeMinibatchDim(true)
                                .channelLayout(NDChannelLayout.GRAYSCALE)
                                .format(NDFormat.CHANNELS_FIRST)
                                .normalization(ImageNormalization.builder().type(ImageNormalization.Type.SCALE).build())
                        )
                        .keys("image")
                        .outputNames("input_layer")
                        .keepOtherValues(true)
                        .metadata(false)
                        .metadataKey(ImageToNDArrayStep.DEFAULT_METADATA_KEY))
                .build()
```

<table>
  <thead>
    <tr>
      <th style="text-align:left">Configs</th>
      <th style="text-align:left">Descriptions</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">height</td>
      <td style="text-align:left">Output array image height. Leave null to convert to the same size as the
        image height.</td>
    </tr>
    <tr>
      <td style="text-align:left">width</td>
      <td style="text-align:left">Output array image width. Leave null to convert to the same size as the
        image width.</td>
    </tr>
    <tr>
      <td style="text-align:left">dataType</td>
      <td style="text-align:left">Data type of the n-dimensional array. Default value is <code>FLOAT</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left">includeMinibatchDim</td>
      <td style="text-align:left">If true, the output array will contain an extra dimension for the minibatch
        number. This will look like (1, Channels, Height, Width) instead of (Channels,
        Height, Width) for format == <code>CHANNELS_FIRST</code> or (1, Height, Width,
        Channels) instead of (Height, Width, Channels) for format == <code>CHANNELS_LAST</code>.
        Default is <code>true</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left">aspectRatioHandling</td>
      <td style="text-align:left">
        <p>An enum to Handle the situation where the input image and output NDArray
          have different aspect ratios.</p>
        <p><code>CENTER_CROP</code> (crop larger dimension then resize if necessary), <code>PAD</code> (pad
          smaller dimension then resize if necessary), <code>STRETCH</code> (simply
          resize, distorting if necessary). Default is <code>CENTER_CROP</code>.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">format</td>
      <td style="text-align:left">The format to be used when converting an Image to an NDArray. Default
        is <code>CHANNEL_FIRST</code>. Another option is <code>CHANNEL_LAST</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left">channelLayout</td>
      <td style="text-align:left">An enum that represents the type (and order) of the color channels for
        an image after it has been converted to an NDArray. For example, <code>RGB</code> vs. <code>BGR</code> etc,
        default value is <code>RGB</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left">normalization</td>
      <td style="text-align:left">Configuration that specifies the normalization type of an image array
        values.</td>
    </tr>
    <tr>
      <td style="text-align:left">listHandling</td>
      <td style="text-align:left">An enum to specify how to handle a list of input images. Default is <code>NONE</code>.</td>
    </tr>
  </tbody>
</table>

