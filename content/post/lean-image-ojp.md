---
author: cunhazera
title: "Reducing OJP Java docker image with jlink and jdeps"
date: 2026-06-12
description: How we managed to reduce OJP Server Docker image by 40%
---

Java has a reputation for being slow, resource-consuming, and also having large artifacts. From the dependencies used in the project to generated jar files and docker images, everything looks too big compared to the binaries produced by other modern languages. Part of the complaint is fair: if optimizing your application is not on your radar, you will certainly get what the default offers, which is not the best result you can get out of your Java project. Narrowing it down to Docker, Java developers must just go ahead and pick the first available image with the JDK version they need. Again, the default or the most obvious choice is not always the best. Picking the first option you find just because it makes your application run will impact your development, and sometimes you don't even consider the consequences for your cycle to build, push, and deploy your artifacts.
That's exactly the scenario we had on OJP (if you don't know what it is, here's a good article to learn more). Back in the day, when the OJP server was still using Java 21/22, `eclipse-temurin:24-jre` was the official image being used, which has ~300mb uncompressed, and would produce a final artifact of 146MB. OJP image used to be generated using a plugin called `jib`. It means that the image was produced by a gradle plugin and the content of the image would be determined in the plugin itself as part of the project build lifecycle. It's worth noting that OJP was already on `jre` and not `jdk` Java image. Therefore, the #1 docker image optimization hint wouldn't help, since the project was relying on a smaller image and that still wasn't enough.

## Old image generation with Google Jib

The image build step was using the jib maven plugin. The `pom.xml` file had the `jib-maven-plugin` pointing to the Temurin JRE image. Every docker setting was defined inside the plugin configuration: main class, JVM args, base image, and the final image that would be generated during the build. Jib is definitely a great tool, convenient for Java projects, allowing Java devs to set up everything they need in their docker artifacts from a maven plugin. If you're familiar with Java build configuration you will feel at home dealing with Jib.
The docker image build step in the pipeline would compile the `ojp-server` module, trigger the plugin via `mvn compile jib:build`, and then the OJP image would be generated and pushed to Docker Hub. Nothing wrong here. On the contrary, the project was doing just fine with this process, and jib was a good fit for a project that needs to handle docker artifacts.
However, jib could not provide the flexibility we needed to reduce the image size. And given that moving from JDK to JRE-based images didn't get the job done, a different approach was required.

## Introducing jlink and jdeps

After reading part of the discussion and the previous work done in attempts to make the image smaller, I remembered watching a talk about jlink and jdeps and how they are useful for creating artifacts with the things you need, without having to bring an entire universe into your artifacts.
jlink (https://openjdk.org/jeps/282) was introduced in JDK 9, as part of Project Jigsaw, which focused on Java modularization. It allows you to create a custom JRE containing only the Java modules your code needs, without including other JDK modules that your code never calls (directly or indirectly). That's the first change of approach: instead of using a traditional JRE available from different vendors, we could have our own JRE.
Here's where a Dockerfile comes in (yes, using jib removes the need for a Dockerfile). The very first line: `FROM eclipse-temurin:24-jdk-alpine AS jre-build`. Yes, a JDK image is being downloaded, but with one purpose: to provide the Java commands required, jlink and jdeps. There's no way around it, you need a traditional JDK/JRE in order to run Java commands. But that does not mean this image will be present in the final result. The chosen approach was to rely on a multi-stage docker build, in order to pull a JRE during the build, but only to run a few commands, not to use it in the final image. Here's how the Dockerfile starts:
```
FROM eclipse-temurin:24-jdk-alpine AS jre-build

RUN jlink \
    --add-modules java.base,java.compiler,java.desktop,java.management,java.naming,java.rmi,java.scripting,java.security.jgss,java.sql,jdk.httpserver,jdk.unsupported \
    --strip-debug \
    --no-man-pages \
    --no-header-files \
    --compress=zip-6 \
    --output /custom-jre
```

Notice that a list of modules (`--add-modules`) is passed as a parameter to the jlink command that will produce the final JRE (`--output /custom-jre`). And that's basically all the Java-related changes that resulted in a smaller image. At the end of the day, it was not about finding a smaller image, but about creating a custom JRE with only the required Java modules, leaving other modules out.
The approach for this case was using a multi-stage docker build. Usually, Java containerized applications just have a single `FROM` in their Dockerfiles, bringing in a standard image like a JDK or JRE to serve common OS requirements and be ready to run the jar. With multi-stage builds, we include multiple `FROM` entries, but only the last one becomes the final image (unless you copy something else into it).
The OJP Server Dockerfile starts by using a JDK image, which is needed to run both `jlink` and `jdeps`, but that won't be packed into the final artifact. So the full JDK here only serves build-related work.
That's why there's a second `FROM` that points to a minimal alpine image, which gets a custom JRE instead of the whole JDK or full JRE. As said before, this will be the final image, containing the `custom-jre` that was built with only the required modules.
Speaking of the required modules of the OJP server, the first version of the custom image had a hard-coded set of values, as shown in the code block above. How did we know exactly which modules to pass to the jlink command? That's where jdeps comes in. When you run `jdeps` pointing to a jar file, it analyzes your bytecode and lists all the modules present in that jar. So this particular part of the Dockerfile was changed to find the required modules at build time:

```
COPY target/ojp-server-*-shaded.jar /tmp/ojp-server.jar

RUN MODULES=$(jdeps --ignore-missing-deps --print-module-deps /tmp/ojp-server.jar) && \
    jlink \
        --add-modules ${MODULES} \
        --strip-debug \
        --no-man-pages \
        --no-header-files \
        --compress=zip-6 \
        --output /custom-jre
```

We flipped the order: now we first copy in the jar and then grab the modules from the compiled binary. Keeping a hard-coded list of modules would increase the chances of throwing `ClassNotFoundException` or `NoClassDefFoundError`, as we could add something to the server project that requires a module not present in that first list. Running jdeps during the image build against the jar guarantees that we get an up-to-date list of modules in case something new is added to the project.
We also added a simple integration test to check that the server running with the custom image starts without missing a class:

```
@Test
void localImageStartsSuccessfully() {
    String version = System.getProperty("ojp.image.version");
    assertNotNull(version, "ojp.image.version must be set — run this test via 'mvn verify', not directly from the IDE");
    DockerImageName image = DockerImageName.parse("rrobetti/ojp:" + version)
            .asCompatibleSubstituteFor("rrobetti/ojp:0.4.2-beta");

    try (OjpContainer container = new OjpContainer(image)) {
        container.start();
        assertTrue(container.isRunning(), "OJP container should be running");
    }
}
```

That alone does not guarantee that nothing is missing, especially because we build this image ignoring possible missing module errors and warnings by passing the `--ignore-missing-deps` parameter to `jlink`. However, 70% of all integration tests run against the lean docker image, which means that more paths are being covered and if there were something missing, these other tests would catch it.

## The result

Considering that the main goal was just to reduce the image size, if you compare version 0.4.1-beta (the last version using JRE) with 0.4.22-beta or even 1.0.0, you can see that the image size went from 146MB to 92MB, which represents a nearly 37% decrease. A significant result with little effort, since we're building a custom gRPC server with a custom JRE that saves a few MBs by only including what is necessary to run the app.
Now, those who run OJP Server with Docker will notice an image significantly smaller, which puts Java applications back in the game, and having resources like jlink and jdeps is really useful for those who want more control over their generated images.

## Lessons learned

Shifting from a stable image to a custom one has brought a few interesting things to be aware of:
 - Java defaults _almost_ never help: default JVM configs, de facto Java images, none of these will deliver the most optimized result. If you're aiming to keep a lean image or small artifacts, you have to deliberately work on these topics. Picking what you want to optimize and setting a goal helps you avoid the feeling that it's never good enough.
 - The obvious choice is not always the best: going after existing images, looking for a smaller image was the first and most obvious choice, but moving to a different approach was the key point to achieve great results. It completely changed how we build the image, of course, but it paid off. Java modularization with `jlink` and `jdeps` weas not the original plan, what taught us to go beyond the obvious.

## Conclusion

Depending on your needs, a third-party image may not serve exactly what you need, and moving on with a custom image might be the way to go. Jib offers easy image generation for Java apps, working very well from gradle config, but that costs flexibility in terms of how you want your image to look like.
