# VIDEO\_CAPTURE

`VideoFrameCaptureStep` is a pipeline step that configures how to extracts a single frame from a video each time inference is called. The video path is hard coded, mainly used for testing/demo purposes.

<table>
  <thead>
    <tr>
      <th style="text-align:left">Configs</th>
      <th style="text-align:left">Descriptions</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">filePath</td>
      <td style="text-align:left">Location of the video file.</td>
    </tr>
    <tr>
      <td style="text-align:left">outputKey</td>
      <td style="text-align:left">Name of the output key where the image frame will be located. Default
        name is <code>image</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left">loop</td>
      <td style="text-align:left">Loop the video when it reaches the end. Default is set to <code>true</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left">skipFrames</td>
      <td style="text-align:left">
        <p>Optional - Number of frames to skip between returned frames. If not set:
          No frames are skipped. For example:</p>
        <p>Value 0: equivalent to no skipping.</p>
        <p>Value 1: skip 1 frame between returned frames (i.e., return every 2nd
          frame).</p>
        <p>Value 3: skip 2 frames between returned frames (i.e., return every 3rd
          frame) and so on.</p>
      </td>
    </tr>
  </tbody>
</table>

