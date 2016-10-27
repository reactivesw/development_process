# Application Logging
This document describes the application logging configuration that enables dynamic remote logging level change using JMX. 

We use logback for all application logging. To enable JMX dynamic configuraiton, follow the instructions in step 1 and 2 to config logging in application and enable JMX in docker image.  

## 1. Logging configuration
Create a `src/main/resources/logback-spring.xml` that has the following content: 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <jmxConfigurator />

    <include resource="org/springframework/boot/logging/logback/base.xml"/>

    <logger name="root" level="INFO" additivity="false">
        <appender-ref ref="CONSOLE" />
    </logger>
</configuration>
```

It configures three things: 

1. Enables JMX remote management
2. Uses default spring logback configuration
3. Sets root trace level to `INFO`


## 2. Enable JMX in Docker Image
To enable remote JMX management, we need to define the following JVM options. 

```sh
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.local.only=false
-Dcom.sun.management.jmxremote.port=<port>
-Dcom.sun.management.jmxremote.rmi.port=<port>
-Djava.rmi.server.hostname=127.0.0.1 
```

The `port` and `rmi.port` should be the same port number.  the following build script can be used to configure docker image.  Here we user a port number of 8081 -- it should be read from an enviornment variable in production. 

```groovy
//the entryPoint
List<String> list = new ArrayList<String>()
list.add("java")
list.add("-Djava.security.egd=file:/dev/./urandom")
list.add("-Dcom.sun.management.jmxremote")
list.add("-Dcom.sun.management.jmxremote.authenticate=false")
list.add("-Dcom.sun.management.jmxremote.ssl=false")
list.add("-Dcom.sun.management.jmxremote.port=8081")
list.add("-Dcom.sun.management.jmxremote.rmi.port=8081")
list.add("-Djava.rmi.server.hostname=127.0.0.1")
list.add("-jar")
list.add("/app.jar")
entryPoint(list)
``` 

## 3. Remote Config Logging Level

First, forward a pod's JMX port ot local port `kubectl port-forward <your-app-pod> 8081`
Then, connect to it to config log level `jconsole 127.0.0.1:8081`
