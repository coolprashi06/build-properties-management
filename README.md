#build-number-management

Often in scenario of continuous delivery and continuous deployment we really need to track what artifact version of build does contain which commit.
go to jenkins build find out the commit info??

NOO....there should be a way to refer to specific version commit from the server build. 
won't it be good for a QA to identify which commit was included in the deployed build. 

well there exists certain ways to do it...

Before we proceed for that I will explain below first how to modify your project number with the jenkins build number to differentiate it when deploying the artifact to artifactory.

Insert below mentioned Maven command in your Jenkins job:

```
build-helper:parse-version versions:set -DnewVersion=${parsedVersion.majorVersion}.${parsedVersion.minorVersion}.${parsedVersion.incrementalVersion}+$BUILD_NUMBER versions:commit
```

Here I'm taking advantage of Build Helper maven plugin to play with project version.

#### get build information from Jenkins
 
When a Jenkins job executes, it sets some environment variables that you may use in your shell script, batch command, Ant script or Maven POM. Some of these environment variables are listed below:

```
BUILD_NUMBER : Jenkins build number
GIT_COMMIT : sha-1 of git commit
GIT_BRANCH : Git branch that was checked out for the build
GIT_URL : Git url (like git@github.com:user/repo.git or [https://github.com/user/repo.git])
JOB_NAME : name of jenkins job used to build
```

Utilize these variables in your pom.xml or properties file as below:
```
<myElement>${BUILD_NUMBER}</myElement>
<myElement1>${GIT_COMMIT}</myElement1>
```

Remember to enable resource filtering in Maven if you need to consume these jenkins environment variables in your properties file:

```
e.g.

<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

In my case I had buildInfo.xml in src/main/resources as:

```
<?xml version="1.0" encoding="UTF-8" ?>
<buildinfo>
        <version>${project.version}</version>
        <git_commit_url>${env.GIT_COMMIT}</git_commit_url>
        <jenkins_build_number>${env.BUILD_NUMBER}</jenkins_build_number>
</buildinfo>

```
after jenkins build the project this xml was populated with the variables as:
```
<?xml version="1.0" encoding="UTF-8" ?>
<buildinfo>
        <version>1.17.5+16</version>
        <git_commit_url>fb6cf246aa776f86e7114370b9aa9a530d36c44a</git_commit_url>
        <jenkins_build_number>16</jenkins_build_number>
</buildinfo>

```

Now you can consume above xml elements in your Java classes.

References: 
https://wiki.jenkins-ci.org/display/JENKINS/Building+a+software+project
https://blog.codecentric.de/en/2015/04/increment-versions-maven-build-helper-versions-plugin/


### Use buildnumber-maven-plugin 
 
create goal of this plugin is used to create a build number.
create-metadata goal is used to generate build metadata properties file.

sample usage shown below:

    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>buildnumber-maven-plugin</artifactId>
        <version>1.4</version>
        <configuration>
          <attach>true</attach>
          <!--make it available for jar/war classpath resource -->
          <addOutputDirectoryToResources>true</addOutputDirectoryToResources>
        </configuration>
        <executions>
          <execution>
            <phase>validate</phase>
            <goals>
              <goal>create-metadata</goal>
            </goals>
          </execution>
        </executions>
      </plugin>

if using this plugin then don't forget to add scm element to define your git repo as below:

  <scm>
    <url>https://github.com/coolprashi06/build-number-management.git</url>
    <connection>scm:git:git://github.com:coolprashi06/build-number-management.git</connection>
    <developerConnection>scm:git:ssh://git@github.com:coolprashi06/build-number-management.git</developerConnection>
  </scm>

After Jenkins build your project, build.properties file can be found in your project WEB-INF directory. content would be like as:

```
version=1.17.5+16
revision=fb6cf246aa776f86e7114370b9aa9a530d36c44a
name=build-number-management
timestamp=1486207610076
```

now you can consume this properties file in your java classes like any other properties file.

References:
https://blog.jayway.com/2012/04/07/continuous-deployment-versioning-and-git/
http://www.mojohaus.org/buildnumber-maven-plugin/


 
Alternatively you can also use git-commit-id-plugin:
https://www.petrikainulainen.net/programming/spring-framework/spring-from-the-trenches-returning-git-commit-information-as-json/
https://github.com/ktoso/maven-git-commit-id-plugin