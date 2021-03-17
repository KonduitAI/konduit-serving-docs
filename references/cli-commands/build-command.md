# Build Command

```text
$ konduit build [-ad <value>] [-a <value>] [-c <value>]  [-dt <value>] [-d
       <value>] [-m <value>] [-o <value>] [-p <value>] [-s <value>]
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
      <td style="text-align:left">-ad,--addDep</td>
      <td style="text-align:left">Additional dependencies to include, in GAV(C) format: &quot;group_id:artifact_id:version&quot;
        / &quot;group_id:artifact_id:version:classifier&quot;.</td>
    </tr>
    <tr>
      <td style="text-align:left">-a,--arch</td>
      <td style="text-align:left">The target CPU architecture. Must be one of {<code>x86</code>, <code>x86_avx2</code>, <code>x86_avx512</code>, <code>armhf</code>, <code>arm64</code>, <code>ppc64le</code>}.
        Note that most modern desktops can be built with <code>x86_avx2</code>,
        which is the default.</td>
    </tr>
    <tr>
      <td style="text-align:left">-c,--config</td>
      <td style="text-align:left">
        <p>Configuration for the deployment types specified via -dt/--deploymentType.</p>
        <p>For example, &quot;-c jar.outputdir=/some/dir jar.name=my.jar&quot; etc.</p>
        <p>Configuration keys:</p>
        <p>JAR deployment config keys: jar.outputdir, jar.name,jar.groupid, jar.artifactid,
          jar.version</p>
        <p>ClassPathDeployment config keys: classpath.outputFile, classpath.type</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">-dt,--deploymentType</td>
      <td style="text-align:left">The deployment types to use: <code>JAR</code>, <code>DOCKER</code>, <code>EXE</code>, <code>WAR</code>, <code>RPM</code>, <code>DEB</code> or <code>TAR</code> (case
        insensitive).</td>
    </tr>
    <tr>
      <td style="text-align:left">-d,--device</td>
      <td style="text-align:left">Compute device to be used. If not set: artifacts are build for <code>CPU</code> only.
        Valid values: <code>CPU</code>, <code>CUDA_10.0</code>, <code>CUDA_10.1</code>, <code>CUDA_10.2</code> (case
        insensitive).</td>
    </tr>
    <tr>
      <td style="text-align:left">-m,--modules</td>
      <td style="text-align:left">Names of the Konduit-Serving modules to include, as a comma-separated
        list of values. Note that this is not necessary when a pipeline is included
        (via <code>-p/--pipeline</code>), as the modules will be inferred automatically
        based on the pipeline contents.</td>
    </tr>
    <tr>
      <td style="text-align:left">-o,--os</td>
      <td style="text-align:left">Operating systems to build for. Valid values: {<code>linux</code>, <code>windows</code>, <code>mac</code>}
        (case insensitive). If not set, the current system OS will be used.</td>
    </tr>
    <tr>
      <td style="text-align:left">-p,--pipeline</td>
      <td style="text-align:left">Path to a pipeline JSON file.</td>
    </tr>
    <tr>
      <td style="text-align:left">-s,--serverType</td>
      <td style="text-align:left">Type of server - <code>HTTP</code> or <code>GRPC</code> (case insensitive).</td>
    </tr>
  </tbody>
</table>

