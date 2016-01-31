[![License Apache](https://www.brimarx.com/pub/apache2.svg)](http://www.apache.org/licenses/LICENSE-2.0)
# javaw
Minimal Java image with a small wrapper to allow JVM to properly stop on Out of Memory (OOM) error.

# Intent
Java and Docker are great technologies but they do not play well when it comes to the Out Of Memory (OOM)
error situation. This container intent is to solve this.

Upon an OOM a JVM is left in an inconsistent state and should really be stopped. Unfortunataly the JVM
keeps running after an OOM. To solve this the most common solution is to add the '-XX:OnOutOfMemoryError="kill -9 %p"'
option to fhe Java options (see http://stackoverflow.com/questions/3871278/how-do-i-make-the-jvm-exit-on-any-outofmemoryexception-even-when-bad-people-try).

Unfortunately this no longer works with Docker when running java as the ENTRYPOINT. This is because the entry point runs as PID 1
and Linux does not allow PID 1 to get killed with a KILL(9) signal. It would be possible to use other signals instead, such as TERM(15)
but since the JVM is no longer in a stable state that would not be reliable.

To allow a KILL(9) to work java shall run as a child process of PID 1. This is exactly what this container is doing: running java as a child
process of PID 1 if the current process is PID 1 (if another PID is found, the java command is just execed). This is basically achieved by running a 'sh -c "java $*"' entrypoint. But there is a bit more to the javaw script to
allow TERM(15) and Ctrl-C to be propagated from PID 1 to the java process so that to allow docker stop and Ctrl-C to propagate a TERM(15)
signal to the java and git it the opportunity to perform a gracefull stop.

# How to use this template?
Use it as any other java template except that you call "javaw" instead of java command.
The '-XX:OnOutOfMemoryError=kill -9 %p' java option is automatically set (hard-coded in the script).

Example:
```
FROM legdba/javaw:latest
MAINTAINER Vincent Bourdaraud <vincent@bourdaraud.com>

# Copy your java files, resources, etc.

# Define CMD and ENTRYPOINT. This is obviously not mandatory
# but this is IMO the better way of using Dockerized application.
CMD ["-someoption", "somevalue"]
# See the example below (any other java syntax would work)
# NOTE the lack of quotes around the kill command!!!
ENV JAVA_OPTS="-Xms50m -Xmx50m"
ENTRYPOINT ["./javaw",\
            "-jar",\
            "/opt/myapp/myapp-all.jar"]
```

# Credits
* Jean Blanchard minimal java template based on busybox (https://registry.hub.docker.com/u/jeanblanchard/java/)
* Alpine docker image

# Source code
See https://github.com/legdba/javaw

# License
This software is under Apache 2.0 license.
```
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
```

