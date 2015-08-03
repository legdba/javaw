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
process of PID 1. This is basically achieved by running a 'sh -c "java $*"' entrypoint. But there is a bit more to the javaw script to
allow TERM(15) and Ctrl-C to be propagated from PID 1 to the java process so that to allow docker stop and Ctrl-C to propagate a TERM(15)
signal to the java and git it the opportunity to perform a gracefull stop.

# How to use this template?
Use it as any other java template except that you call "javaw" instead of java command.
You should as well set the '-XX:OnOutOfMemoryError="kill -9 %p"' java option.

Example:
```
FROM legdba/javaw:latest
MAINTAINER Vincent Bourdaraud <vincent@bourdaraud.com>

# Copy your java files, resources, etc.

# Define CMD and ENTRYPOINT. This is obviously not mandatory
# but this is IMO the better way of using Dockerized application.
CMD ["-someoption", "somevalue"]
# Example with -jar option; any other way of running your application
# would work. I find fatJars to be an easier way of running Dockerized
# applications though.
ENTRYPOINT ["./javaw",\
            "-jar",\
            "-XX:OnOutOfMemoryError='kill -9 %p'",\
            "-Xms50m",\
            "-Xmx50m",\
            "/opt/myapp/myapp-all.jar"]
```
