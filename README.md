## Instruction to start the Distributed Application from scratch
- Download the project zip file from [here](http://dell-edu-lab-store.s3.ap-south-1.amazonaws.com/repository/pages-distributed.zip)  and extract it inside workspace folder
- Create a repository in github with the name *pages-distributed*. Keep everything default, while creating the repository, don't change anything other than default.
- Copy the git remote add origin <path> command and execute it in the directory
- Create a build.gradle file with following content

```groovy
plugins {
	id 'org.springframework.boot' version '2.3.1.RELEASE'
	id 'io.spring.dependency-management' version '1.0.9.RELEASE'
	id 'java'
}

group = 'com.example'
sourceCompatibility = '11'

repositories {
	mavenCentral()
}
bootJar{
	enabled=false
}
```
- Create the gradle ecosystem by using the following commands
```sh
    gradle wrapper --gradle-version 6.4.1 --distribution-type all
```
- Create a .gitignore file with following content
```text
HELP.md
.gradle
build/
!gradle/wrapper/gradle-wrapper.jar
!**/src/main/**
!**/src/test/**

### STS ###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

### IntelliJ IDEA ###
.idea
*.iws
*.iml
*.ipr
out/

### NetBeans ###
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/

### VS Code ###
.vscode/
```
- Open the project in Intellij Idea, select the import gradle project option in low right corner and set project SDK to JDK 11
- We need create a multi-module gradle project. The names of modules are given below. To create the a module *Right Click* on the root project ---> New ---> Module ---> Select Module as Gradle and Library as Java ---> Enter the full module name by copying it from below.
  It would also create the default build.gradle file in each module, which we would change later.
  * components/business
  * components/category
  * application/business-server
  * application/category-server
  
- Create a *settings.gradle* file in the project directory if not present.Add below content to the settings.gradle and remove everything if present.
```groovy
rootProject.name = 'pages-distributed'
include 'components:business'
include 'components:category'
include 'application:business-server'
include 'application:category-server'
```
- Create src/main/java and src/main/resources folder under all the modules. Ensure that both these folders are marked as *Sources Root* and *Resources Root* respectively in all the four modules.
- Create a server.gradle file under application
