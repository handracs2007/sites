## Handra

### [Home](/) | Blog | [Disclaimer](/disclaimer) | [Terms and conditions](/tnc)

[<<go back](..)

### Integrate OWASP Dependency Check with Maven
In this post, we are going to look at how to integrate OWASP dependency check with Maven. I will not go deep into what Maven is, assuming that all readers have some sort of familiarity with Maven.

---
#### Background
Anyone who works with Java, especially those who are building enterprise level applications, must know that there are plenty of libraries they will use to build the product. With more and more dependencies we add to our project, it becomes harder and harder to keep track of any security issue exists in our dependencies. Luckily, [OWASP (Open Web Application Security Project)](https://owasp.org/) provides us with a dependency check tool that can help us to check our dependencies for any reported vulnerability.

---
#### Why is it Important
As Java developers, we all know how much we rely on dependencies on our daily job. With the ease of Maven (or Gradle), we can easily add so many dependencies whenever we need it. This though, might bring bad impact in our code security if not managed well. Some libraries, albeit published, might have security issues that we as developers need to be aware of.

Keeping up-to-date with all the security fixes or vulnerabilities from plethora of libraries most likely is quite hard for us as a developer. Hence, the importance of the OWASP dependency check tool. Furthermore, we can integrate this with our CI/CD pipeline that can stop the build process when it found any security issue reported in one of our dependencies.

---
### Getting Started
To start, I have created a very simple Java project using Maven. The dependency check plugin for Maven itself can be easily configured inside the `plugin` section inside our `pom.xml` file. Below is the simplest element to add the dependency check into our `pom.xml`.

``` xml
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>6.1.5</version>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

As of this writing, the latest available version of the dependency check plugin is `6.1.5`, hence the version inside the `pom.xml`.

---
### Verification
Now that we have it configured inside our `pom.xml` file, let's do a quick test to ensure that it is working.

First of all, I'll add a new dependency to my project. Just for the sake of the demo purpose, I added older version of the `spring-boot` library.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot</artifactId>
    <version>2.2.9.RELEASE</version>
</dependency>
```

As can be seen, I added version `2.2.9.RELEASE` to my dependency. This version is known to have vulnerabilities.

Now, let's execute our maven dependency check using the command `mvn verify`. Below is the sample output of the command.

<pre>
handra@nebula  <b>mvn verify</b>
[INFO] Scanning for projects...
[INFO] 
[INFO] --------------------< com.handra.demo:mvndepcheck >---------------------
[INFO] Building mvndepcheck 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ mvndepcheck ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ mvndepcheck ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ mvndepcheck ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /media/handra/DATA/IdeaProjects/mvndepcheck/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ mvndepcheck ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ mvndepcheck ---
[INFO] No tests to run.
[INFO] 
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ mvndepcheck ---
[INFO] 
[INFO] --- dependency-check-maven:6.1.5:check (default) @ mvndepcheck ---
[INFO] Checking for updates
[INFO] Skipping NVD check since last check was within 4 hours.
[INFO] Skipping RetireJS update since last update was within 24 hours.
[INFO] Check for updates complete (98 ms)
[INFO] 

Dependency-Check is an open source tool performing a best effort analysis of 3rd party dependencies; false positives and false negatives may exist in the analysis performed by the tool. Use of the tool and the reporting provided constitutes acceptance for use in an AS IS condition, and there are NO warranties, implied or otherwise, with regard to the analysis or its use. Any use of the tool and the reporting provided is at the user’s risk. In no event shall the copyright holder or OWASP be held liable for any damages whatsoever arising out of or in connection with the use of this tool, the analysis performed, or the resulting report.


   About ODC: https://jeremylong.github.io/DependencyCheck/general/internals.html
   False Positives: https://jeremylong.github.io/DependencyCheck/general/suppression.html

💖 Sponsor: https://github.com/sponsors/jeremylong


[INFO] Analysis Started
[INFO] Finished Archive Analyzer (0 seconds)
[INFO] Finished File Name Analyzer (0 seconds)
[INFO] Finished Jar Analyzer (0 seconds)
[INFO] Finished Dependency Merging Analyzer (0 seconds)
[INFO] Finished Version Filter Analyzer (0 seconds)
[INFO] Finished Hint Analyzer (0 seconds)
[INFO] Created CPE Index (1 seconds)
[INFO] Finished CPE Analyzer (1 seconds)
[INFO] Finished False Positive Analyzer (0 seconds)
[INFO] Finished NVD CVE Analyzer (0 seconds)
[INFO] Finished Sonatype OSS Index Analyzer (0 seconds)
[INFO] Finished Vulnerability Suppression Analyzer (0 seconds)
[INFO] Finished Dependency Bundling Analyzer (0 seconds)
[INFO] Analysis Complete (1 seconds)
[INFO] Writing report to: /media/handra/DATA/IdeaProjects/mvndepcheck/target/dependency-check-report.html
<b>[WARNING] 

One or more dependencies were identified with known vulnerabilities in mvndepcheck:

spring-core-5.2.8.RELEASE.jar (pkg:maven/org.springframework/spring-core@5.2.8.RELEASE, cpe:2.3:a:pivotal_software:spring_framework:5.2.8:release:*:*:*:*:*:*, cpe:2.3:a:springsource:spring_framework:5.2.8:release:*:*:*:*:*:*, cpe:2.3:a:vmware:springsource_spring_framework:5.2.8:release:*:*:*:*:*:*) : CVE-2020-5421</b>


See the dependency-check report for more details.


[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  4.192 s
[INFO] Finished at: 2021-04-17T15:28:25+08:00
[INFO] ------------------------------------------------------------------------
</pre>

From the above output, you can see that there is a vulnerability reported.

```
spring-core-5.2.8.RELEASE.jar (pkg:maven/org.springframework/spring-core@5.2.8.RELEASE, cpe:2.3:a:pivotal_software:spring_framework:5.2.8:release:*:*:*:*:*:*, cpe:2.3:a:springsource:spring_framework:5.2.8:release:*:*:*:*:*:*, cpe:2.3:a:vmware:springsource_spring_framework:5.2.8:release:*:*:*:*:*:*) : CVE-2020-5421
```

This proves that with this simple configuration, it is enough to help us to verify that our dependencies are secure or not.

---
### Failing the Build
If you have noticed though, despite vulnerabilities were reported, maven still said that the build was successful. This is not something that we might want. We would prefer to have the build failed when vulnerabilities were reported.

To achieve this, we need to add another configuration into our dependency check plugin.

```xml
<failBuildOnCVSS>1</failBuildOnCVSS>
```

The number configured inside that tag is the minimum CVSS score reported for a vulnerability to determine whether or not to fail the build. The value `1` means that if any vulnerability found in our dependencies with the CVSS score of at least 1, the build shall fail. You may refer to [here](https://en.wikipedia.org/wiki/Common_Vulnerability_Scoring_System) to learn more about CVSS.

Now, let's retry our `mvn verify` command and see if the build fails or not.

<pre>
handra@nebula  <b>mvn verify</b>
[INFO] Scanning for projects...
[INFO] 
[INFO] --------------------< com.handra.demo:mvndepcheck >---------------------
[INFO] Building mvndepcheck 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ mvndepcheck ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ mvndepcheck ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ mvndepcheck ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /media/handra/DATA/IdeaProjects/mvndepcheck/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ mvndepcheck ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ mvndepcheck ---
[INFO] No tests to run.
[INFO] 
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ mvndepcheck ---
[INFO] Building jar: /media/handra/DATA/IdeaProjects/mvndepcheck/target/mvndepcheck-1.0-SNAPSHOT.jar
[INFO] 
[INFO] --- dependency-check-maven:6.1.5:check (default) @ mvndepcheck ---
[INFO] Checking for updates
[INFO] Skipping NVD check since last check was within 4 hours.
[INFO] Skipping RetireJS update since last update was within 24 hours.
[INFO] Check for updates complete (100 ms)
[INFO] 

Dependency-Check is an open source tool performing a best effort analysis of 3rd party dependencies; false positives and false negatives may exist in the analysis performed by the tool. Use of the tool and the reporting provided constitutes acceptance for use in an AS IS condition, and there are NO warranties, implied or otherwise, with regard to the analysis or its use. Any use of the tool and the reporting provided is at the user’s risk. In no event shall the copyright holder or OWASP be held liable for any damages whatsoever arising out of or in connection with the use of this tool, the analysis performed, or the resulting report.


   About ODC: https://jeremylong.github.io/DependencyCheck/general/internals.html
   False Positives: https://jeremylong.github.io/DependencyCheck/general/suppression.html

💖 Sponsor: https://github.com/sponsors/jeremylong


[INFO] Analysis Started
[INFO] Finished Archive Analyzer (0 seconds)
[INFO] Finished File Name Analyzer (0 seconds)
[INFO] Finished Jar Analyzer (0 seconds)
[INFO] Finished Dependency Merging Analyzer (0 seconds)
[INFO] Finished Version Filter Analyzer (0 seconds)
[INFO] Finished Hint Analyzer (0 seconds)
[INFO] Created CPE Index (1 seconds)
[INFO] Finished CPE Analyzer (1 seconds)
[INFO] Finished False Positive Analyzer (0 seconds)
[INFO] Finished NVD CVE Analyzer (0 seconds)
[INFO] Finished Sonatype OSS Index Analyzer (0 seconds)
[INFO] Finished Vulnerability Suppression Analyzer (0 seconds)
[INFO] Finished Dependency Bundling Analyzer (0 seconds)
[INFO] Analysis Complete (2 seconds)
[INFO] Writing report to: /media/handra/DATA/IdeaProjects/mvndepcheck/target/dependency-check-report.html
[WARNING] 

<b>One or more dependencies were identified with known vulnerabilities in mvndepcheck:

spring-core-5.2.8.RELEASE.jar (pkg:maven/org.springframework/spring-core@5.2.8.RELEASE, cpe:2.3:a:pivotal_software:spring_framework:5.2.8:release:*:*:*:*:*:*, cpe:2.3:a:springsource:spring_framework:5.2.8:release:*:*:*:*:*:*, cpe:2.3:a:vmware:springsource_spring_framework:5.2.8:release:*:*:*:*:*:*) : CVE-2020-5421</b>


See the dependency-check report for more details.


[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  4.352 s
[INFO] Finished at: 2021-04-17T15:29:55+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.owasp:dependency-check-maven:6.1.5:check (default) on project mvndepcheck: 
[ERROR] 
<b>[ERROR] One or more dependencies were identified with vulnerabilities that have a CVSS score greater than or equal to '1.0': 
[ERROR] 
[ERROR] spring-core-5.2.8.RELEASE.jar: CVE-2020-5421</b>
[ERROR] 
[ERROR] See the dependency-check report for more details.
[ERROR] 
[ERROR] 
[ERROR] -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
</pre>

As can be seen, the build is now failed. The below message indicates that our configuration was applied correctly.

```
[ERROR] One or more dependencies were identified with vulnerabilities that have a CVSS score greater than or equal to '1.0'
```

---
### Handling the False Alarm or Ignoring Vulnerability
As with any software, there would be cases where the reported vulnerabilities were actually false alarms. Sometimes, we would even ignore some vulnerabilities altogether as those are not relevant in our context (i.e.: our software does not have attack surface to trigger the attack).

In order to ignore a vulnerability, we can tell the plugin to read from a suppression file. The suppression file is actually an XML file. We need to configure what vulnerabilities to be suppressed / ignored from being reported and hence failing our build.

First, we need to add a new configuration into the plugin.

```xml
<suppressionFiles>
    <suppressionFile>suppression.xml</suppressionFile>
</suppressionFiles>
```

The configuration above tells the plugin to read the vulnerabilities that can be suppressed from a file named `suppression.xml`. As you might have noticed, you can repeat the element `suppressionFile` multiple times if you have more than one suppression files.

Now, let's say we want to ignore the vulnerabilities reported in our above build, here is what we have to put inside our suppression file.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<suppressions xmlns="https://jeremylong.github.io/DependencyCheck/dependency-suppression.1.3.xsd">
    <suppress>
        <notes><![CDATA[
      I do not care for now.
      ]]></notes>
        <packageUrl>pkg:maven/org.springframework/spring-core@5.2.8.RELEASE</packageUrl>
        <cve>CVE-2020-5421</cve>
    </suppress>

    <suppress>
        <notes><![CDATA[
      I do not care for now.
      ]]></notes>
        <packageUrl>pkg:maven/org.springframework/spring-aop@5.2.8.RELEASE</packageUrl>
        <cve>CVE-2020-5421</cve>
    </suppress>

    <suppress>
        <notes><![CDATA[
      I do not care for now.
      ]]></notes>
        <packageUrl>pkg:maven/org.springframework/spring-jcl@5.2.8.RELEASE</packageUrl>
        <cve>CVE-2020-5421</cve>
    </suppress>

    <suppress>
        <notes><![CDATA[
      I do not care for now.
      ]]></notes>
        <packageUrl>pkg:maven/org.springframework/spring-beans@5.2.8.RELEASE</packageUrl>
        <cve>CVE-2020-5421</cve>
    </suppress>

    <suppress>
        <notes><![CDATA[
      I do not care for now.
      ]]></notes>
        <packageUrl>pkg:maven/org.springframework/spring-context@5.2.8.RELEASE</packageUrl>
        <cve>CVE-2020-5421</cve>
    </suppress>

    <suppress>
        <notes><![CDATA[
      I do not care for now.
      ]]></notes>
        <packageUrl>pkg:maven/org.springframework/spring-expression@5.2.8.RELEASE</packageUrl>
        <cve>CVE-2020-5421</cve>
    </suppress>
</suppressions>
```

You can see there are several elements that I put inside the `suppressions` tag. The most important, you can refer to tag `packageUrl` and `cve`. The `packageUrl` is the URL of the package with vulnerability reported. The `cve` is the number of the CVE reported. You can easily find both information in the reported error by maven.

For example, for the below error:

<pre>
One or more dependencies were identified with known vulnerabilities in mvndepcheck:

spring-core-5.2.8.RELEASE.jar (<b>pkg:maven/org.springframework/spring-core@5.2.8.RELEASE</b>, cpe:2.3:a:pivotal_software:spring_framework:5.2.8:release:*:*:*:*:*:*, cpe:2.3:a:springsource:spring_framework:5.2.8:release:*:*:*:*:*:*, cpe:2.3:a:vmware:springsource_spring_framework:5.2.8:release:*:*:*:*:*:*) : <b>CVE-2020-5421</b>
</pre>

The package URL is `pkg:maven/org.springframework/spring-core@5.2.8.RELEASE` and the CVE number is `CVE-2020-5421`.

Now, let's perform the same command and check again the output.
<pre>
handra@nebula  <b>mvn verify</b>
[INFO] Scanning for projects...
[INFO] 
[INFO] --------------------< com.handra.demo:mvndepcheck >---------------------
[INFO] Building mvndepcheck 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ mvndepcheck ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ mvndepcheck ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ mvndepcheck ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /media/handra/DATA/IdeaProjects/mvndepcheck/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ mvndepcheck ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ mvndepcheck ---
[INFO] No tests to run.
[INFO] 
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ mvndepcheck ---
[INFO] 
[INFO] --- dependency-check-maven:6.1.5:check (default) @ mvndepcheck ---
[INFO] Checking for updates
[INFO] Skipping NVD check since last check was within 4 hours.
[INFO] Skipping RetireJS update since last update was within 24 hours.
[INFO] Check for updates complete (109 ms)
[INFO] 

Dependency-Check is an open source tool performing a best effort analysis of 3rd party dependencies; false positives and false negatives may exist in the analysis performed by the tool. Use of the tool and the reporting provided constitutes acceptance for use in an AS IS condition, and there are NO warranties, implied or otherwise, with regard to the analysis or its use. Any use of the tool and the reporting provided is at the user’s risk. In no event shall the copyright holder or OWASP be held liable for any damages whatsoever arising out of or in connection with the use of this tool, the analysis performed, or the resulting report.


   About ODC: https://jeremylong.github.io/DependencyCheck/general/internals.html
   False Positives: https://jeremylong.github.io/DependencyCheck/general/suppression.html

💖 Sponsor: https://github.com/sponsors/jeremylong


[INFO] Analysis Started
[INFO] Finished Archive Analyzer (0 seconds)
[INFO] Finished File Name Analyzer (0 seconds)
[INFO] Finished Jar Analyzer (0 seconds)
[INFO] Finished Dependency Merging Analyzer (0 seconds)
[INFO] Finished Version Filter Analyzer (0 seconds)
[INFO] Finished Hint Analyzer (0 seconds)
[INFO] Created CPE Index (1 seconds)
[INFO] Finished CPE Analyzer (1 seconds)
[INFO] Finished False Positive Analyzer (0 seconds)
[INFO] Finished NVD CVE Analyzer (0 seconds)
[INFO] Finished Sonatype OSS Index Analyzer (0 seconds)
[INFO] Finished Vulnerability Suppression Analyzer (0 seconds)
[INFO] Finished Dependency Bundling Analyzer (0 seconds)
[INFO] Analysis Complete (2 seconds)
[INFO] Writing report to: /media/handra/DATA/IdeaProjects/mvndepcheck/target/dependency-check-report.html
[INFO] ------------------------------------------------------------------------
<b>[INFO] BUILD SUCCESS</b>
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  4.307 s
[INFO] Finished at: 2021-04-17T16:34:48+08:00
[INFO] ------------------------------------------------------------------------
</pre>

As can be seen, by providing the suppression file, the build is now successful.

To learn more ways to suppress a vulnerability, you may go to [https://jeremylong.github.io/DependencyCheck/general/suppression.html](https://jeremylong.github.io/DependencyCheck/general/suppression.html){:target=_blank}

---
That's all. The dependency check plugin makes it very easy for us to check for any vulnerability in our dependencies. This then can be integrated into our CI/CD pipeline to help ensuring that the software that we are building and releasing is using more secure dependencies.

I hope this is helpful and the source code for this is available at Github [here](https://github.com/handracs2007/mvndepcheck){:target=_blank}.

Cheers

---

> Copyright &copy; 2020-2022 Handra. All Rights Reserved.
