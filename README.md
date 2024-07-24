# SpringBootCracRestoreCheckpoint
# Prepare a Spring Boot app with CRaC
For our tutorial, we will use a Spring Boot reference application, Spring Petclinic.

First of all, clone the Spring Petclinic's repository:
```
git clone https://github.com/spring-projects/spring-petclinic.git
```
To use CRaC with the new Spring Boot and Spring Framework, we need to add the dependency for the org.crac/crac package to pom.xml:

```
<dependency>
    <groupId>org.crac</groupId>
    <artifactId>crac</artifactId>
    <version>1.4.0</version>
</dependency>
```
Let's check the difference of our updated pom.xml:
```
git diff
diff --git a/pom.xml b/pom.xml
index 287a08a..f403155 100644
--- a/pom.xml
+++ b/pom.xml
@@ -36,6 +36,11 @@
   </properties>

   <dependencies>
+    <dependency>
+        <groupId>org.crac</groupId>
+        <artifactId>crac</artifactId>
+        <version>1.4.0</version>
+    </dependency>
     <!-- Spring and Spring Boot dependencies -->
     <dependency>
       <groupId>org.springframework.boot</groupId>
```
Now, build the application with

```
mvn clean package
```
# Prepare a work directory for the application dump

CRaC enables the developers to save the exact state of a running Java application (together with the information about the heap, JIT-compiled code, and so on). We need to create a work directory where the application dump will be stored after the checkpoint.

For this purpose, run

```
mkdir -p storage/checkpoint-spring-petclinic
```
We should also copy the Petclinic's jar file to the “storage” to use in a docker container:


```
cp target/spring-petclinic-3.2.0-SNAPSHOT.jar ./storage/
```
# Start the application in a Docker container

We will use the bellsoft/liberica-runtime-container:jdk-21-crac-slim-glibc image to start Petclinic and get the application dump for further restore. Note that BellSoft also provides images with musl libc.

Another important prerequsite is the kernel version of the underlying Linux distribution, which should be at least 5.9. Linux kernel 5.9 introduces the CAP_CHECKPOINT_RESTORE option, which separates the checkpoint/restore functionality from the CAP_SYS_ADMIN option and enables the developers to steer clear of running containers with elevated permissions.

The CAP_CHECKPOINT_RESTORE (and SYS_PTRACE, which is also required for checkpoint-restore) options are enabled with --cap-add flag added to newer Docker versions, so make sure you have the latest available Docker version.

Use the following command to start the application in a Docker container:

```
docker run --cap-add CHECKPOINT_RESTORE --cap-add SYS_PTRACE -d -v $(pwd)/storage:/storage -w /storage --name petclinic-app-container bellsoft/liberica-runtime-container:jdk-21-crac-slim-glibc java -Xmx512m -XX:CRaCCheckpointTo=/storage/checkpoint-spring-petclinic -jar spring-petclinic-3.3.0-SNAPSHOT.jar
```
If you use a Linux distribution with an older kernel version, you can you the --priviledged flag instead of CAP_CHECKPOINT_RESTORE and SYS_PTRACE (however, this is not the best practice to run containers with elevated permissions).

Now, let’s check the application output.

```
root@ip-172-31-39-178:/home/ubuntu/spring-petclinic# docker logs petclinic-app-container


              |\      _,,,--,,_
             /,`.-'`'   ._  \-;;,_
  _______ __|,4-  ) )_   .;.(__`'-'__     ___ __    _ ___ _______
 |       | '---''(_/._)-'(_\_)   |   |   |   |  |  | |   |       |
 |    _  |    ___|_     _|       |   |   |   |   |_| |   |       | __ _ _
 |   |_| |   |___  |   | |       |   |   |   |       |   |       | \ \ \ \
 |    ___|    ___| |   | |      _|   |___|   |  _    |   |      _|  \ \ \ \
 |   |   |   |___  |   | |     |_|       |   | | |   |   |     |_    ) ) ) )
 |___|   |_______| |___| |_______|_______|___|_|  |__|___|_______|  / / / /
 ==================================================================/_/_/_/

:: Built with Spring Boot :: 3.3.0


2024-07-24T11:40:39.729Z  INFO 129 --- [           main] o.s.s.petclinic.PetClinicApplication     : Starting PetClinicApplication v3.3.0-SNAPSHOT using Java 21.0.3 with PID 129 (/storage/spring-petclinic-3.3.0-SNAPSHOT.jar started by root in /storage)
2024-07-24T11:40:39.741Z  INFO 129 --- [           main] o.s.s.petclinic.PetClinicApplication     : No active profile set, falling back to 1 default profile: "default"
2024-07-24T11:40:43.242Z  INFO 129 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
2024-07-24T11:40:43.436Z  INFO 129 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 168 ms. Found 2 JPA repository interfaces.
2024-07-24T11:40:45.546Z  INFO 129 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port 8080 (http)
2024-07-24T11:40:45.572Z  INFO 129 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2024-07-24T11:40:45.573Z  INFO 129 --- [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.24]
2024-07-24T11:40:45.662Z  INFO 129 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2024-07-24T11:40:45.665Z  INFO 129 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 5681 ms
2024-07-24T11:40:46.471Z  INFO 129 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2024-07-24T11:40:46.841Z  INFO 129 --- [           main] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Added connection conn0: url=jdbc:h2:mem:e5883060-41c2-4343-90b3-099c842bd38f user=SA
2024-07-24T11:40:46.843Z  INFO 129 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2024-07-24T11:40:47.177Z  INFO 129 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
2024-07-24T11:40:47.303Z  INFO 129 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 6.5.2.Final
2024-07-24T11:40:47.365Z  INFO 129 --- [           main] o.h.c.internal.RegionFactoryInitiator    : HHH000026: Second-level cache disabled
2024-07-24T11:40:47.941Z  INFO 129 --- [           main] o.s.o.j.p.SpringPersistenceUnitInfo      : No LoadTimeWeaver setup: ignoring JPA class transformer
2024-07-24T11:40:50.666Z  INFO 129 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000489: No JTA platform available (set 'hibernate.transaction.jta.platform' to enable JTA platform integration)
2024-07-24T11:40:50.672Z  INFO 129 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2024-07-24T11:40:51.557Z  INFO 129 --- [           main] o.s.d.j.r.query.QueryEnhancerFactory     : Hibernate is in classpath; If applicable, HQL parser will be used.
2024-07-24T11:40:55.500Z  INFO 129 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 14 endpoints beneath base path '/actuator'
2024-07-24T11:40:55.698Z  INFO 129 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
2024-07-24T11:40:55.743Z  INFO 129 --- [           main] o.s.s.petclinic.PetClinicApplication     : Started PetClinicApplication in 17.305 seconds (process running for 18.93)

```
We can see that The PetClinicApplication started in 17.305 seconds.

# Perform the checkpoint to get the application dump
The jcmd command is used to send the command to VM to make the checkpoint and dump the application and VM state to storage:

```
docker exec petclinic-app-container jcmd spring JDK.checkpoint
```
As a result, the Java instance with the Petclinic app will be dumped and the container will be stopped. The docker ps will show that the petclinic-app-container was stopped.

You can check that the images for your application were dumped to the directory:

```
ls storage/checkpoint-spring-petclinic/

core-129.img  core-133.img  core-137.img  core-141.img  core-145.img  core-151.img  core-156.img  core-160.img  core-165.img  core-208.img  core-212.img  fs-129.img     pagemap-129.img  stats-dump
core-130.img  core-134.img  core-138.img  core-142.img  core-146.img  core-152.img  core-157.img  core-161.img  core-166.img  core-209.img  dump4.log     ids-129.img    pages-1.img      timens-0.img
core-131.img  core-135.img  core-139.img  core-143.img  core-147.img  core-153.img  core-158.img  core-163.img  core-167.img  core-210.img  fdinfo-2.img  inventory.img  pstree.img
core-132.img  core-136.img  core-140.img  core-144.img  core-150.img  core-154.img  core-159.img  core-164.img  core-207.img  core-211.img  files.img     mm-129.img     seccomp.img
```

# Use the prepared dump to start the application and the VM

Let’s start the application by restoring it from the dump. To do that, run


```
docker run --cap-add CHECKPOINT_RESTORE --cap-add SYS_PTRACE -it --rm -v $(pwd)/storage:/storage -w /storage -p 8080:8080 --name petclinic-app-container-from-checkpoint bellsoft/liberica-runtime-container:jdk-21-crac-slim-glibc java -XX:CRaCRestoreFrom=/storage/checkpoint-spring-petclinic
```

Check the application logs:


```
root@ip-172-31-39-178:/home/ubuntu/spring-petclinic# docker logs petclinic-app-container-f
rom-checkpoint
2024-07-24T11:41:57.920Z  INFO 129 --- [Attach Listener] o.s.c.support.DefaultLifecycleProcessor  : Restarting Spring-managed lifecycle beans after JVM restore
2024-07-24T11:41:57.938Z  INFO 129 --- [Attach Listener] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
2024-07-24T11:41:57.947Z  INFO 129 --- [Attach Listener] o.s.c.support.DefaultLifecycleProcessor  : Spring-managed lifecycle restart completed (restored JVM running for 102 ms)
```

We can see that restoration of checkpoint was very quick.

# Summary:

Checkpoint/Restore in Userspace  offers several benefits in terms of boot startup time for applications or services:

## Fast Application Startup:

Allows for the quick restoration of a previously checkpointed application or service.
Applications can resume from a saved state, significantly reducing the startup time compared to starting from scratch.

## State Preservation:

Captures and saves the complete state of an application, including memory, CPU state, and open file descriptors.
Enables applications to continue execution seamlessly from where they left off after restoration.

## Improved Resource Utilization:

Reduces the time required for scaling operations in cloud environments.
Facilitates faster scaling of applications and services based on demand, optimizing resource utilization.

## Enhanced Availability:

Enables rapid recovery from failures or system maintenance events.
Minimizes downtime by quickly restoring applications from checkpoints, ensuring high availability in production environments.

## Efficient Testing and Development:

Provides developers with a means to capture and restore application states for testing and debugging purposes.
Accelerates testing cycles by eliminating the need for repetitive initialization steps.
