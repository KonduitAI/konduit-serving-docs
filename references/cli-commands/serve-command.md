# Serve Command

```text
$ konduit serve [-ad <additional_dependencies>] [-b] [-cp <classpath>] -c
       <server-config>  [-h <host>] [-i <instances>] [-jo <value>]  [-p <port>]
       [-p <profile_name>] [-rwm] [-s <type>] [-id <value>]
```

**OPTIONS**

| Command Flags | Description |
| :--- | :--- |
| -ad,--addDep | Additional dependencies to include with the launch. |
| -b,--background | Runs the process in the background, if set. |
| -cp,--classpath | Provides an extra classpath to be used for the verticle deployment. |
| -c,--config | Specifies configuration that should be provided to the verticle. `<config>` should reference either a text file containing a valid JSON object which represents the configuration OR be a JSON string. |
| -h,--host | Specifies the host name of the Konduit server when the configuration provided is just a pipeline configuration instead of a whole inference configuration. Defaults is: `localhost`.  |
| -i,--instances | Specifies how many instances of the server will be deployed. Defaults is: `1`.  |
| -jo,--java-opts | Java Virtual Machine options to pass to the spawned process such as "`-Xmx1G` `-Xms256m -XX:MaxPermSize=256m`". If not set, the `JAVA_OPTS` environment variable is used.  |
| -p,--port | Specifies the port number of the konduit server when the configuration provided is just a pipeline configuration instead of a whole inference configuration. Defaults is: `0`.  |
| -p,--profileName | Name of the profile to be used with the server launch.  |
| -rwm,--runWithoutManifest  | Do not create the manifest jar file before launching the server.  |
| -s,--service | Service type that needs to be deployed. Defaults is: `inference`. |
| -id,--serving-id | Id of the serving process. This will be visible in the `list` command. This id can be used to call `predict` and `stop` commands on the running servers. If not given then an 8 character UUID is created automatically. |

