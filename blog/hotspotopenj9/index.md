## Handra

### [Home](/) | Blog | [Disclaimer](/disclaimer) | [Terms and conditions](/tnc)

[<<go back](..)

### Comparing Hotspot and OpenJ9
In this post, we are going to do a comparison between Hotspot and OpenJ9. For those of you who are already familiar with Java, you must already be familiar with the term Hotspot as well. Hotspot is the [Java Virtual Machine (JVM)](https://en.wikipedia.org/wiki/Java_virtual_machine){:target=_blank} implementation developed by Oracle (before this was Sun Microsystem). However, OpenJ9 might be less known JVM implementation among the community, especially newcomers to the Java ecosystem. Nowadays, it gains more traction due to the cloud native movement.

---

#### A brief history of OpenJ9
OpenJ9 is yet another JVM implementation, originally owned and developed by IBM, which then contributed to Eclipse Foundation on 2017, hence the naming Eclipse OpenJ9. Built to run on mobile devices with very limited memory, OpenJ9 is a suitable solution to be used on the cloud. With high optimisation for fast startup, lower memory footprint, quick ramp-up, and excellent throughput performance, applications deployed using OpenJ9 claimed to be more cost-effective, especially when deployed in the cloud. Some people has claimed that by just replacing their JVM from Hotspot to OpenJ9, their memory consumption has been reduced quite significantly.

OpenJ9 is an open-source project that anyone can contribute. You may go to their Github page <https://github.com/eclipse/openj9> for more details on the OpenJ9, including the source code. You can even start contributing to the project from there :).

---

#### Sample SpringBoot Application
For the sake of this testing, I developed a simple application using SpringBoot and Kotlin. What the application does is just to expose ONE (1) REST API endpoint. When the endpoint is called, internally it will call another REST API and return it to the API caller.

Below is the snipped of the controller.
````kotlin
@RestController
class DemoResource {

    private val restTemplate = RestTemplate()

    @GetMapping("/call")
    fun callMe(): String {
        // Just simply call the web service and get the response
        val response = this.restTemplate.getForObject("https://covid19-api.org/api/status", String::class.java)
        return response!!
    }
}
````

As can be seen from the sample code above, the API does something super simple. It will call the covid19 API that is freely available and accessible. Again, for the sake of simplicity, it will just return the response to the API caller without doing any processing. This is enough for the testing that we are going to conduct.

The SpringBoot application source code is freely accessible from <https://github.com/handracs2007/simplespringbootapp>.

---

#### Docker containers
The application will be deployed in a Docker container. I am using images from the adoptopenjdk that can easily be pulled from the Docker hub.

Below is the Dockerfile for my application based on Hotspot JVM.

```
FROM adoptopenjdk:11-jre-hotspot
RUN mkdir /opt/app
COPY simplespringbootapp-0.0.1-SNAPSHOT.jar /opt/app/myapp.jar
CMD ["java", "-jar", "/opt/app/myapp.jar"]
```

Below is the Dockerfile for my application based on OpenJ9 JVM.
```
FROM adoptopenjdk:11-jre-openj9
RUN mkdir /opt/app
COPY simplespringbootapp-0.0.1-SNAPSHOT.jar /opt/app/myapp.jar
CMD ["java", "-jar", "/opt/app/myapp.jar"]
```

As you can see, the only difference is for the Hotspot, I'm using ```adoptopenjdk:11-jre-hotspot``` whereas for the OpenJ9, I'm using ```adoptopenjdk:11-jre-openj9```.

```bash
handra@nebula  ~  docker images
REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
myapp-openj9                      latest              2538d7c30891        4 hours ago         241MB
myapp-hotspot                     latest              bfc58fc19964        4 hours ago         249MB
```

---

Now, let's startup the containers and see what is the memory usage after first startup.

I'm using the following command to startup the container for Hotspot. The published port to be used to hit the server is **8080**.
```bash
docker run -p 8080:8080 --name myapphotspot --rm -d myapp-hotspot
```

The below command is to startup the container for OpenJ9. The published port to be used to hit the server is **9090**.
```bash
docker run -p 9090:8080 --name myappopenj9 --rm -d myapp-openj9
```

By using the ```docker stats``` command, we will be able to monitor the docker statistics. Below is the output I received from the command.
```bash
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
320cf8c860c7        myapphotspot        0.19%               243.6MiB / 15.25GiB   1.56%               5.95kB / 0B         35.2MB / 0B         39
da54c85579d1        myappopenj9         0.67%               84.14MiB / 15.25GiB   0.54%               4.31kB / 0B         38MB / 4.1kB        46
```

Simply from the result above, we can see that ```Hotspot``` is using **243.6MiB** whereby OpenJ9 is using only **84.14Mib**. This is **65.5%** less memory usage. So far we have seen quite a good result.

---

#### Load testing
Now, we will see whether or not this lower memory footprint can really be retained after we are sending plenty of requests. I will be doing load testing to both the containers and see how the overall result is.

For the load testing, I'm using the **ab** command, that is provided by the Apache. This tool is used mainly for benchmarking (hence the name **ab**).

For this testing, I will be sending **1000000** requests together to both containers, with the number of concurrent users set to **50** (I don't want to break down my laptop as well with super high load :)).

This is the command I used to hit the Hotspot container (port 8080).
```bash
ab -c 50 -n 1000000 -r http://localhost:8080/call
```

This is the command I used to hit the OpenJ9 container (port 9090).
```bash
ab -c 50 -n 1000000 -r http://localhost:9090/call
```

From the conducted testing, below is the result.
```
	TPS                                 RAM		
	Hotspot     OpenJ9      Reduction   Hotspot OpenJ9  Reduction
#1	12512.54    11885.73    5.01        380.8   145.4   61.82
#2	13443.84    12608.04    6.22        381.4   151.8   60.20
#3	13096.41    12419.7     5.17        382.2   151     60.49
```

As can be seen from the result above, the memory usage of the OpenJ9 is still significantly lower than what the Hotspot is using. However, as you might have noticed as well, this memory reduction is paid by the decrease of throughput by around 5-6%. Depending on your application use case, this might or might not be a huge number.

During the testing phase, the CPU usage of both containers are as well equal.

---

#### Conclusion
Based on the testing result, it is proven than OpenJ9 provides significantly lower resource usage for Java applications, which can be proven useful for the cloud deployment. However, it does come with a reasonable drawback.

So, which one to choose for your next project? The selection of technology must be aligned with the nature of your application. As the saying goes, it depends. For now, I go with OpenJ9 due to my needs. The choice is yours.

Cheers

---

> Copyright &copy; 2020 Handra. All Rights Reserved.