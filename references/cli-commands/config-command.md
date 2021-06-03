# Config Command

```text
$ konduit config  [-m] [-o <output-file>] -p <config> [-pr <value>]  [-y]
```

**OPTIONS**

<table>
  <thead>
    <tr>
      <th style="text-align:left">Command Flags</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">-m,--minified</td>
      <td style="text-align:left">If set, the output JSON will be printed in a single line, without indentations.
        (Ignored for YAML configuration output).</td>
    </tr>
    <tr>
      <td style="text-align:left">-o,--output</td>
      <td style="text-align:left">Optional: If set, the generated JSON/YAML will be saved here. Otherwise,
        it&apos;s printed on the console.</td>
    </tr>
    <tr>
      <td style="text-align:left">-p,--pipeline</td>
      <td style="text-align:left">
        <p>A comma-separated list of sequence/graph pipeline steps to create boilerplate
          configuration from.</p>
        <p></p>
        <p>For sequences, allowed values are: [<code>crop_grid</code>, <code>crop_fixed_grid</code>, <code>dl4j</code>, <code>keras</code>, <code>draw_bounding_box</code>, <code>draw_fixed_grid</code>, <code>draw_grid</code>, <code>draw_segmentation</code>, <code>extract_bounding_box</code>, <code>camera_frame_capture</code>, <code>video_frame_capture</code>, <code>image_to_ndarray</code>, <code>logging</code>, <code>ssd_to_bounding_box</code>, <code>samediff</code>, <code>show_image</code>, <code>tensorflow</code>, <code>nd4jtensorflow</code>, <code>python</code>, <code>onnx</code>, <code>classifier_output</code>].</p>
        <p></p>
        <p>For graphs, the list item should be in the format &apos;&lt;output&gt;=&lt;type&gt;(&lt;inputs&gt;)&apos;
          or &apos;[outputs]=switch(&lt;inputs&gt;)&apos; for switches. The pre-defined
          root input is named, &apos;input&apos;. Examples are:</p>
        <p>Pipeline step: &apos;a=tensorflow(input),b=dl4j(input)&apos;</p>
        <p>Merge Step: &apos;c=merge(a,b)&apos;</p>
        <p>Switch Step (int): &apos;[d1,d2,d3]=switch(int,select,input)&apos;</p>
        <p>Switch Step (string): &apos;[d1,d2,d3]=switch(string,select,x:1,y:2,z:3,input)
          &apos;</p>
        <p>Any Step: &apos;e=any(d1,d2,d3)&apos;.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">-pr,--protocol</td>
      <td style="text-align:left">Protocol to use with the server. Allowed values are [<code>http</code>, <code>grpc</code>, <code>mqtt</code>].</td>
    </tr>
    <tr>
      <td style="text-align:left">-y,--yaml</td>
      <td style="text-align:left">Set if you want the output to be a YAML configuration.</td>
    </tr>
  </tbody>
</table>

