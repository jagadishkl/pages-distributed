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
- Commit the code in local git repository
-----------
Distributed App Start
--------------
## Instruction to start the Distributed Application from scratch
The applications have Test files. Follow the below instructions to reach to solution for the Test classes.
### Code Changes for Module component/category
- Replace the content of build.gradle with below
```groovy
plugins {
    id 'org.springframework.boot'
    id 'io.spring.dependency-management'
    id 'java'
}

description("Category Component")
bootJar {
    enabled = false
}

jar {
    enabled = true
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
}
```
- Create two packages *org.dell.edu.kube.category.data*  and *org.dell.edu.kube.category* under src/main/java 
- Create **Category.java** in *org.dell.edu.kube.category.data* package
```java
package org.dell.edu.kube.category.data;

import com.fasterxml.jackson.annotation.JsonInclude;

import javax.persistence.*;
import java.util.*;

@Entity
@Table(name = "category")
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Category  {
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String type;
    private String description;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }


    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Category category = (Category) o;
        return Objects.equals(id, category.id) &&
                Objects.equals(name, category.name) &&
                Objects.equals(type, category.type) &&
                Objects.equals(description, category.description) ;
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name, type, description);
    }

    @Override
    public String toString() {
        return "Category{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", type='" + type + '\'' +
                ", description='" + description +
                '}';
    }
}
```
- Create **CategoryRepository.java** interface in *org.dell.edu.kube.category.data* package
```java
package org.dell.edu.kube.category.data;

import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.CrudRepository;

import java.util.List;

public interface CategoryRepository extends CrudRepository<Category,Long> {

    @Query("select c from Category c where c.type = ?1")
    List<Category> findByType(String type);
}
```
- Create a RestController in the name **CategoryController.java** in *org.dell.edu.kube.category* package
```java
package org.dell.edu.kube.category;

import org.dell.edu.kube.category.data.Category;
import org.dell.edu.kube.category.data.CategoryRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping("/category")
public class CategoryController {
    Logger logger = LoggerFactory.getLogger(CategoryController.class);
    @Autowired
    CategoryRepository repository;

    @PostMapping
    public ResponseEntity add(@RequestBody Category category){
        repository.save(category);
        logger.debug("Category created "+category);
        return new ResponseEntity(category, HttpStatus.CREATED);
    }

    @GetMapping
    public ResponseEntity getAll(){
        return new ResponseEntity(repository.findAll(),HttpStatus.OK);
    }

    @GetMapping("/{id}")
    public ResponseEntity find(@PathVariable Long id){
        Optional<Category> category = repository.findById(id);
        if(category.isPresent()){
            return new ResponseEntity(category.get(),HttpStatus.OK);
        }else {
            return new ResponseEntity("No Category Available",HttpStatus.NOT_FOUND);
        }
    }

    @GetMapping("type/{type}")
    public ResponseEntity findByType(@PathVariable String type){
        List<Category> category = repository.findByType(type);
        if(category != null && !category.isEmpty()){
            return new ResponseEntity(category,HttpStatus.OK);
        }else{
            return new ResponseEntity("No Business Category available for the type",HttpStatus.NOT_FOUND);
        }
    }
    @PutMapping("/{id}")
    public ResponseEntity update(@PathVariable Long id,@RequestBody Category category){
        if(repository.existsById(id)){
            category.setId(id);
            repository.save(category);
            return new ResponseEntity(category,HttpStatus.OK);
        }else {
            return new ResponseEntity("Category Not Available",HttpStatus.NOT_FOUND);
        }
    }

    @DeleteMapping("/{id}")
    public String delete(@PathVariable Long id){
        repository.deleteById(id);
        return "Category Deleted";
    }
}
```
### Code changes for Module component/business
- Replace the build.gradle with below content
```groovy
plugins {
    id 'org.springframework.boot'
    id 'io.spring.dependency-management'
    id 'java'
}

description("Business Component")
bootJar {
    enabled = false
}

jar {
    enabled = true
}

repositories {
    mavenCentral()
}

dependencies {
    implementation project(":components:category")
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
}
```
- Create two packages *org.dell.edu.kube.business.data* and *org.dell.edu.kube.business* under src/main/java
- Create **Business.java** in *org.dell.edu.kube.business.data* package.
```java
package org.dell.edu.kube.business.data;

import com.fasterxml.jackson.annotation.JsonInclude;

import javax.persistence.*;
import java.util.Objects;

@Entity
@Table(name="business")
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Business  {
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String address;
    private String owner;
    @Column(name = "category_id")
    private Long  category;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getOwner() {
        return owner;
    }

    public void setOwner(String owner) {
        this.owner = owner;
    }

    public Long getCategory() {
        return category;
    }

    public void setCategory(Long category) {
        this.category = category;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Business business = (Business) o;
        return Objects.equals(id, business.id) &&
                Objects.equals(name, business.name) &&
                Objects.equals(address, business.address) &&
                Objects.equals(owner, business.owner) &&
                Objects.equals(category, business.category);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name, address, owner, category);
    }

    @Override
    public String toString() {
        return "Business{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", address='" + address + '\'' +
                ", owner='" + owner + '\'' +
                ", category=" + category +
                '}';
    }
}
```
- Create **BusinessVO.java** in *org.dell.edu.kube.business.data* package.
```java
package org.dell.edu.kube.business.data;

import com.fasterxml.jackson.annotation.JsonInclude;
import org.dell.edu.kube.category.data.Category;

import java.io.Serializable;
import java.util.Objects;

@JsonInclude(JsonInclude.Include.NON_NULL)
public class BusinessVO implements Serializable {
    private Long id;

    private String name;
    private String address;
    private String owner;
    private Category category;
    private Long categoryId;

    public BusinessVO() {
    }

    public BusinessVO(Business business) {
        this.id = business.getId();
        this.name = business.getName();
        this.address = business.getAddress();
        this.owner = business.getOwner();
        //this.category = category;
        this.categoryId = business.getCategory();
    }


    @Override
    public String toString() {
        return "BusinessVO{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", address='" + address + '\'' +
                ", owner='" + owner + '\'' +
                ", category=" + category +
                ", categoryId=" + categoryId +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        BusinessVO that = (BusinessVO) o;
        return Objects.equals(id, that.id) &&
                Objects.equals(name, that.name) &&
                Objects.equals(address, that.address) &&
                Objects.equals(owner, that.owner) &&
                Objects.equals(category, that.category)&&
                Objects.equals(categoryId, that.categoryId);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name, address, owner, category,categoryId);
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getOwner() {
        return owner;
    }

    public void setOwner(String owner) {
        this.owner = owner;
    }

    public Category getCategory() {
        return category;
    }

    public void setCategory(Category category) {
        this.category = category;
    }

    public Long getCategoryId() {
        return categoryId;
    }

    public void setCategoryId(Long categoryId) {
        this.categoryId = categoryId;
    }
}
```
- Create **BusinessRepository.java** interface in *org.dell.edu.kube.business.data* package. 
```java
package org.dell.edu.kube.business.data;

import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.CrudRepository;

import java.util.List;

public interface BusinessRepository extends CrudRepository<Business,Long> {
    @Query("select b from Business b where b.category = ?1")
    List<Business> findByCategory(Long category);

    @Query("select b from Business b where b.owner = ?1")
    List<Business> findByOwner(String owner);


}
```
- Create a RestController **BusinessController.java** in *org.dell.edu.kube.business* package.
```java
package org.dell.edu.kube.business;

import org.dell.edu.kube.business.data.BusinessRepository;
import org.dell.edu.kube.business.data.BusinessVO;
import org.dell.edu.kube.business.data.Business;
import org.dell.edu.kube.category.data.Category;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;

import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping(path="/business")
public class BusinessController {
    Logger logger = LoggerFactory.getLogger(BusinessController.class);
    @Autowired
    BusinessRepository repository;
    @Autowired
    RestTemplate restTemplate;
    @Value("${category.url:http://localhost:8082/category}")
    private String categoryUrl;

    @PostMapping
    public ResponseEntity add( @RequestBody Business business){

        repository.save(business);
        BusinessVO vo = new BusinessVO(business);
        if(business.getCategory() != null ){
            Category category = getCategory(business.getCategory());
            if(category != null){
                vo.setCategory(category);
            }
        }
        logger.debug("**************************Business Entity Created"+vo+"*****************************");
        return new ResponseEntity(vo, HttpStatus.CREATED);
    }

    @GetMapping
    public ResponseEntity all(){
        return new ResponseEntity(repository.findAll(),HttpStatus.OK);

    }

    @GetMapping("/{id}")
    public ResponseEntity get(@PathVariable Long id){
        Optional<Business> business = repository.findById(id);
        if(business.isPresent()){
            BusinessVO vo = new BusinessVO(business.get());
            if(vo.getCategoryId() != null){
                vo.setCategory(getCategory(vo.getCategoryId()));
            }
            return new ResponseEntity(vo,HttpStatus.OK);
        }else{
            return new ResponseEntity("Business not available",HttpStatus.NOT_FOUND);
        }

    }
    @PutMapping("/{id}")
    public ResponseEntity update(@PathVariable Long id, @RequestBody Business business){
        if(repository.existsById(id)){
            business.setId(id);
            repository.save(business);
            return  new ResponseEntity(business,HttpStatus.OK);
        }else{
            return new ResponseEntity("Business not available",HttpStatus.NOT_FOUND);
        }
    }

    @DeleteMapping("/{id}")
    public ResponseEntity delete(@PathVariable Long id){
        if(repository.existsById(id)){
            repository.deleteById(id);
        }
        return new ResponseEntity("Deleted",HttpStatus.OK);

    }

    @GetMapping("category/{categoryId}")
    public ResponseEntity getByCategory(@PathVariable Long categoryId){
        Category category = getCategory(categoryId);
        if(category != null){
            List<Business> businesses = repository.findByCategory(categoryId);
            return new ResponseEntity(businesses,HttpStatus.OK);
        }else {
            return new ResponseEntity("Wrong or Invalid Category ID",HttpStatus.NOT_FOUND);
        }
    }

    @GetMapping("owner/{owner}")
    public ResponseEntity getByOwner(@PathVariable String owner){
        List<Business> business = repository.findByOwner(owner);
        if(business != null && !business.isEmpty()){
            return new ResponseEntity(business,HttpStatus.OK);
        }else{
            return new ResponseEntity("No Businesses owned by the owner",HttpStatus.NOT_FOUND);
        }

    }

    private Category getCategory(Long categoryId){
        ResponseEntity<Category> entity = null;
        try{
            entity =  restTemplate.getForEntity(categoryUrl+"/{id}",Category.class,categoryId);
        }catch (Exception e){
            logger.error("No Category Available for ID"+categoryId);
        }
        if(entity != null){
            logger.debug("*************************Category Available :"+"*****************************");
            return entity.getBody();
        }else {

            return null;
        }
    }
}
```
### Code Changes in application
- Replace the content of server.gradle with below content
```groovy
apply plugin: "org.springframework.boot"
apply plugin: "io.spring.dependency-management"
apply plugin: "java"


repositories {
    mavenCentral()
}
dependencies {
    implementation "org.springframework.boot:spring-boot-starter-web"
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation "org.springframework.boot:spring-boot-starter-actuator"
    implementation group: 'io.springfox', name: 'springfox-core', version: '2.9.2'
    implementation group: 'io.swagger', name: 'swagger-annotations', version: '1.6.2'
    implementation 'io.springfox:springfox-swagger2:2.9.2'
    implementation  'io.springfox:springfox-swagger-ui:2.9.2'
    runtimeOnly 'mysql:mysql-connector-java:8.0.12'

    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
}

test {
    useJUnitPlatform()
}
```
### Code Changes in application/category-server
- Replace the *build.gradle* with below content
```groovy
apply from: "$projectDir/../server.gradle"

group = 'org.dell.edu.kube'
version = '0.0.1-SNAPSHOT'
description("Category Server")

dependencies {
    implementation project(":components:category")
}
```
- Create package *org.dell.edu.kube.category* under src/main/java
- Create Application class named **CategoryApplication.java** in *org.dell.edu.kube.category* package
```java
package org.dell.edu.kube.category;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@EnableSwagger2
@SpringBootApplication
public class CategoryApplication {
    @Bean
    public Docket productApi() {
        return new Docket(DocumentationType.SWAGGER_2).select()
                .apis(RequestHandlerSelectors.basePackage("org.dell.edu.kube.category")).build();
    }


    public static void main(String[] args) {
        SpringApplication.run(CategoryApplication.class, args);
    }

}

```
- Create Application class named **WelcomeCategoryController.java** in *org.dell.edu.kube.category* package
```java
package org.dell.edu.kube.category;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/")
public class WelcomeCategoryController {
    Logger loger = LoggerFactory.getLogger(WelcomeCategoryController.class);
    @Value("${welcome.message:Welcome to Kubernetes Category Application}")
    private String message;
    @GetMapping
    public String index(){
        loger.debug("Welcome to Kubernetes Category Application Message Generated");
        loger.info("Welcome to Kubernetes Category Application Message Generated");
        loger.trace("Welcome to Kubernetes Category Application Message Generated");
        loger.warn("Welcome to Kubernetes Category Application Message Generated");
        loger.error("Welcome to Kubernetes Category Application Message Generated");
        return message;
    }
}
```
- Add the following content in the application.properties in src/main/resources folder. Create a new file of not present.
```properties
spring.application.name=category
server.port=8082
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect

#For Deployment in Kubernetes
#spring.datasource.url=jdbc:mysql://mysql/category?createDatabaseIfNotExist=true&allowPublicKeyRetrieval=true&useSSL=false&user=root
#MySQL Root user password in kubernetes deployment is password
#spring.datasource.password=password
#spring.datasource.username=root

#For Testing locally
spring.datasource.url=jdbc:mysql://localhost:3306/category?createDatabaseIfNotExist=true&allowPublicKeyRetrieval=true&useSSL=false&user=root
#For Deployment locally provide the appropriate root user password
#[Root User Password @Localhost MySQL Deployment]
spring.datasource.password=
spring.datasource.username=root

logging.file.name=/var/tmp/category.log
debug=true
logging.level.org.springframework.web=debug
logging.level.root=debug
logging.level.org.hibernate=error
welcome.message="<html><head><title>Welcome to Dell Kubernetes Category Microservices</title></head><body><center><h1>Welcome to the Dell Kubernetes Microservices Workshop<h1><br><h2>Please click <a href='/swagger-ui.html'>here </a> to access the API Documentation</h2><br><h2>Please click <a href='/actuator'>here </a> to access the actuator endpoints</h2></center></body></html>"
```
### Code Changes in application/business-server
- Chnage the content of build.gradle with the below
```groovy
apply from: "$projectDir/../server.gradle"

group = 'org.edu.dell.kube'
version = '0.0.1-SNAPSHOT'
description("Business Server")

dependencies {
    implementation project(":components:category")
    implementation project(":components:business")

}
```
- Create a package in the name **org.dell.edu.kube.business** under src/main/java
- Create class **BusinessApplication.java**
```java
package org.dell.edu.kube.business;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.domain.EntityScan;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@EnableSwagger2
@EntityScan(basePackages="org.dell.edu.kube.business")
@SpringBootApplication
public class BusinessApplication {
    @Bean
    public Docket productApi() {
        return new Docket(DocumentationType.SWAGGER_2).select()
                .apis(RequestHandlerSelectors.basePackage("org.dell.edu.kube.business"))
                .build();
    }


    public static void main(String[] args) {
        SpringApplication.run(BusinessApplication.class, args);
    }

    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder.build();
    }

}
```
- Create RestController **WelcomeBusinessController.java**
```java
package org.dell.edu.kube.business;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/")
public class WelcomeBusinessController {
    Logger loger = LoggerFactory.getLogger(WelcomeBusinessController.class);
    @Value("${welcome.message:Welcome to Kubernetes Business Application}")
    private String message;
    @GetMapping
    public String index(){
        loger.debug("Welcome to Kubernetes Business Application Message Generated");
        loger.info("Welcome to Kubernetes Business Application Message Generated");
        loger.warn("Welcome to Kubernetes Business Application Message Generated");
        loger.trace("Welcome to Kubernetes Business Application Message Generated");
        loger.error("Welcome to Kubernetes Business Application Message Generated");
        return message;
    }
}
```
- Add the following content in the application.properties in src/main/resources folder. Create a new file of not present.
```properties
spring.application.name=business
server.port=8081
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect

#For Deployment in Kubernetes
#spring.datasource.url=jdbc:mysql://mysql/business?createDatabaseIfNotExist=true&allowPublicKeyRetrieval=true&useSSL=false&user=root
#MySQL Root user password in kubernetes deployment is password
#spring.datasource.password=password
#spring.datasource.username=root

#For Testing locally
spring.datasource.url=jdbc:mysql://localhost:3306/business?createDatabaseIfNotExist=true&allowPublicKeyRetrieval=true&useSSL=false&user=root
#For Deployment locally provide the appropriate root user password
#[Root User Password @Localhost MySQL Deployment]
spring.datasource.password=
spring.datasource.username=root

logging.file.name=/var/tmp/business.log
debug=true
logging.level.org.springframework.web=debug
logging.level.root=debug
logging.level.org.hibernate=error
welcome.message="<html><head><title>Welcome to Dell Kubernetes Business Microservices</title></head><body><center><h1>Welcome to the Dell Kubernetes Microservices Workshop<h1><br><h2>Please click <a href='/swagger-ui.html'>here </a> to access the API Documentation</h2><br><h2>Please click <a href='/actuator'>here </a> to access the actuator endpoints</h2></center></body></html>"
```
### Local Testing of the application
- *business-server* is dependent on the *category-server*. We need to run *category-server* followed by *business-server*
```shell script
./gradlew clean
./gradlew :application:category-server:build
./gradlew :application:business-server:build
./gradlew :application:category-server:bootRun
```
- Open another terminal and run
```shell script
./gradlew :application:business-server:bootRun
```

### Dockerizing both the applications
- Execute the below commands to build it afresh.
```shell script
./gradlew clean
./gradlew :application:category-server:build 
./gradlew :application:business-server:build 
```
- Create a directory **dockerfiles** under the project root.
- Create a file **Dockerfile-business** in *dockerfiles* with below content
```shell script
FROM openjdk:11-jdk
ARG JAR_FILE=application/business-server/build/libs/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```
- Create a file **Dockerfile-category** in *dockerfiles* with below content
```shell script
FROM openjdk:11-jdk
ARG JAR_FILE=application/category-server/build/libs/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```
- To create docker images use the below commands. Please replace <docker-user-name> with your own docker hub user name'
```shell script
docker build -f dockerfiles/Dockerfile-category -t <docker-user-name>/category:distributed .
docker build -f dockerfiles/Dockerfile-business -t <docker-user-name>/business:distributed .
```
- Test the docker images locally by running the below commands
```shell script
docker run -p 8082:8082 -t <docker-user-name>/category:distributed
docker run -p 8081:8081 -t <docker-user-name>/business:distributed
```
- To push the images to docker hub use below commands
```shell script
docker push <docker-user-name>/category:distributed
docker push <docker-user-name>/business:distributed
```

### Kubernetizing the application
- In both business-server and category-server, in the src/main/resources/application.properties comment the  all the properties present under *For Testing locally* section and uncomment all the properties present under *For Deployment in Kubernetes* section

- Create a **deployments** folder under the root folder. We need to create the following Kubernetes Deployment files under *deployments* folder
- dist-namespace.yaml
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: <your-name>
```
 - app-log-pvc.yaml
 ```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: log-persistent-claim
  namespace: <your-name>
spec:
  volumeMode: Filesystem
  storageClassName: pv-<your-name>
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```
 - app-log-pv.yaml
 ```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: log-persistent-volume
  namespace: <your-name>
  labels:
    type: local
spec:
  volumeMode: Filesystem
  storageClassName: pv-<your-name>
  capacity:
    storage: 500Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/logs"
```
 - business-deployment.yaml
 ```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: business
  name: business
  namespace: <your-name>
spec:
  replicas: 2
  selector:
    matchLabels:
      app: business
  strategy: {}
  template:
    metadata:
      labels:
        app: business
    spec:
      volumes:
      - name: log-volume
        persistentVolumeClaim:
          claimName: log-persistent-claim
      containers:
      - image: <docker-user-name>/business:distributed
        imagePullPolicy: Always
        name: business
        env:
        - name : CATEGORY_URL
          value: "http://category:8082/category"
        volumeMounts:
        - name: log-volume
          mountPath: "/var/tmp/"
        ports:
        - containerPort: 8081
        resources: {}
status: {}

```
 - business-pod-deployment.yaml
 ```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: business
  name: business
  namespace: <your-name>
spec:
  volumes:
  - name: log-volume
    persistentVolumeClaim:
      claimName: log-persistent-claim
  containers:
  - image: <docker-user-name>/business:distributed
    imagePullPolicy: Always
    name: business
    env:
    - name : CATEGORY_URL
      value: "http://category:8082/category"
    volumeMounts:
      - name: log-volume
        mountPath: "/var/tmp/"
    ports:
    - containerPort: 8081
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```
 - business-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: business
  name: business
  namespace: <your-name>
spec:
  ports:
  - port: 8081
    protocol: TCP
    targetPort: 8081
  selector:
    app: business
  type: LoadBalancer
status:
  loadBalancer: {}

```
 - category-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: category
  name: category
  namespace: <your-name>
spec:
  replicas: 2
  selector:
    matchLabels:
      app: category
  strategy: {}
  template:
    metadata:
      labels:
        app: category
    spec:
      volumes:
      - name: log-volume
        persistentVolumeClaim:
          claimName: log-persistent-claim
      containers:
      - image: <docker-user-name>/category:distributed
        imagePullPolicy: Always
        name: category
        ports:
        - containerPort: 8082
        volumeMounts:
        - name: log-volume
          mountPath: "/var/tmp/"
        resources: {}
status: {}
```
 - category-pod-deployment.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: category
  name: category
  namespace: <your-name>
spec:
  volumes:
  - name: log-volume
    persistentVolumeClaim:
        claimName: log-persistent-claim
  containers:
  - image: <docker-user-name>/category:distributed
    imagePullPolicy: Always
    name: category
    volumeMounts:
    - name: log-volume
      mountPath: "/var/tmp/"
    ports:
    - containerPort: 8082
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```
 - category-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: category
  name: category
  namespace: <your-name>
spec:
  ports:
  - port: 8082
    protocol: TCP
    targetPort: 8082
  selector:
    app: category
  type: LoadBalancer
status:
  loadBalancer: {}

```
- mysql-secret.yaml
```yaml
apiVersion: v1
data:
  mysql-pass: cGFzc3dvcmQ=
kind: Secret
metadata:
  name: mysql-secret
  namespace: <your-name>
```
 - mysql-client.sh
```shell script
kubectl run -it --rm --image=mysql:8.0 --restart=Never mysql-client -- mysql -h mysql -ppassword
```
 - mysql-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: <your-name>
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:8.0
        name: mysql
        env:
          # Instead of using value directly we could also use secrets
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: mysql-pass
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim

```
 - mysql-pvc.yaml
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  namespace: <your-name>
spec:
  storageClassName: mysql-<your-name>
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

```
 - mysql-pv.yaml
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: mysql-persistent-volume
  namespace: <your-name>
  labels:
    type: local
spec:
  storageClassName: mysql-<your-name>
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
 - mysql-service.yaml
```yaml
kind: Service
apiVersion: v1
metadata:
  name: mysql
  namespace: <your-name>
  labels:
    app: mysql
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
  clusterIP: None
```
- Replace the value of  \<docker-user-name> with proper docker-user-name in all *-deployment.yaml files. Also imagePullPolicy could be removed. Please add executable permission on the mysql-client.sh file.
### Code Changes for Pipeline
- Add the folowing in application/business-server/build.gradle
```groovy
test.environment([
        "SPRING_DATASOURCE_USERNAME": "root",
        "SPRING_DATASOURCE_PASSWORD": "root",
		"SPRING_DATASOURCE_URL": "jdbc:mysql://localhost:3306/business?createDatabaseIfNotExist=true&useSSL=false&user=root",
])
```
- Add the folowing in application/category-server/build.gradle
```groovy
test.environment([
        "SPRING_DATASOURCE_USERNAME": "root",
        "SPRING_DATASOURCE_PASSWORD": "root",
		"SPRING_DATASOURCE_URL": "jdbc:mysql://localhost:3306/category?createDatabaseIfNotExist=true&useSSL=false&user=root",
])
```
- Create the following secrets in your github repository
  * DOCKER_USERNAME [Your own docker account user name]
  * DOCKER_PASSWORD [Your own docker account login password ]
  * PKS_API [Get it from the instructor]
  * PKS_USERNAME [Get it from the instructor]
  * PKS_PASSWORD [Get it from the instructor]
  * PKS_CLUSTER [Get it from the instructor]
  * PKS_TOKEN [Follow the below steps to create it]
```text
1. Create an account using the link https://account.run.pivotal.io/z/uaa/sign-up
2. Check your inbox and verify email, so that you can sign in successfully.
3. Access https://network.pivotal.io/users/dashboard/edit-profile
4. Create an API token and copy it.
5. Use it as the value for PKS_TOKEN
```
- Create **.github/workflows** directory under root project and create *pipeline.yaml* file with below content
```yaml
name: PeloPages Distributed Pipeline

on:
  push:
    branches:
    - master

jobs:
  build-artifact:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - name: Set up JDK 11
        uses: actions/setup-java@v1
        with:
          java-version: 11
      - name: Start Ubuntu MySQL
        run: sudo systemctl start mysql.service
      - name: Build with Gradle
        uses: eskatos/gradle-command-action@v1
        with:
          arguments: build
          gradle-version: 6.4
      - name: Verify Build
        run: |
           ls -l application/business-server/build/libs
           ls -l application/category-server/build/libs
           echo "Build Fine"
      - name: Upload Artifact Business
        uses: actions/upload-artifact@v2
        with:
          name: artifact
          path: application/business-server/build/libs/business-1.0-SNAPSHOT.jar
      - name: Upload Artifact Category
        uses: actions/upload-artifact@v2
        with:
          name: artifact
          path: application/category-server/build/libs/category-1.0-SNAPSHOT.jar
      - name: build-docker-image-business
        uses: docker/build-push-action@v1
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
          repository: <docker-your-name>/business
          tags: distributed
          dockerfile: dockerfiles/Dockerfile-business
      - name: build-docker-image-category
        uses: docker/build-push-action@v1
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
          repository: <docker-user-name>/category
          tags: distributed
          dockerfile: dockerfiles/Dockerfile-category
  deploy-image-to-pks:
    needs: build-artifact
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - name: Install Pivnet & PKS
        run: |
          sudo apt-get update
          wget -O pivnet github.com/pivotal-cf/pivnet-cli/releases/download/v0.0.55/pivnet-linux-amd64-0.0.55 && chmod +x pivnet && sudo mv pivnet /usr/local/bin
          pivnet login --api-token=${{ secrets.PKS_TOKEN }}
          pivnet download-product-files --product-slug='pivotal-container-service' --release-version='1.7.0' --product-file-id=646536
          sudo mv pks-linux-amd64-1.7.0-build.483 pks
          chmod +x pks
          sudo mv pks /usr/local/bin/
      - name: Install Kubectl
        run: |
          pivnet download-product-files --product-slug='pivotal-container-service' --release-version='1.7.0' --product-file-id=633728
          sudo mv  kubectl-linux-amd64-1.16.7 kubectl
          sudo mv kubectl /usr/local/bin/
      - name: PKS Login
        run: |
          pks login -a ${{ secrets.PKS_API }}   -u ${{ secrets.PKS_USERNAME }} -k -p ${{ secrets.PKS_PASSWORD }}
          pks get-credentials ${{ secrets.PKS_CLUSTER }}

          kubectl apply -f deployments/dist-namespace.yaml
          kubectl apply -f deployments/app-log-pvc.yaml
          kubectl apply -f deployments/app-log-pv.yaml
          kubectl apply -f deployments/mysql-pv.yaml
          kubectl apply -f deployments/mysql-pvc.yaml
          kubectl apply -f deployments/mysql-service.yaml
          kubectl apply -f deployments/mysql-secret.yaml
          kubectl apply -f deployments/mysql-deployment.yaml
          kubectl apply -f deployments/category-service.yaml
          kubectl apply -f deployments/category-deployment.yaml
          kubectl apply -f deployments/business-service.yaml
          kubectl apply -f deployments/business-deployment.yaml
```
- In all the files replace \<docker-user-name> with your own docker-user-name and replace \<user-name> with your own first name
- Push your code to github repository so that Github Actions would build, test, dockerize the application and finally deploy it on kubernetes cluster.
- The pipeline would create a namespace with \<your-name> and create all the objects inside it.
- All the files would be executed in following order in kubernetes cluster if pipeline is not used.
```shell script
kubectl apply -f deployments/dist-namespace.yaml
kubectl apply -f deployments/app-log-pvc.yaml
kubectl apply -f deployments/app-log-pv.yaml
kubectl apply -f deployments/mysql-pv.yaml
kubectl apply -f deployments/mysql-pvc.yaml
kubectl apply -f deployments/mysql-service.yaml
kubectl apply -f deployments/mysql-secret.yaml
kubectl apply -f deployments/mysql-deployment.yaml
kubectl apply -f deployments/category-service.yaml
kubectl apply -f deployments/category-deployment.yaml
kubectl apply -f deployments/business-service.yaml
kubectl apply -f deployments/business-deployment.yaml
```
- If business and category deployments are already available in the cluster and your are trying to re-deploy. always delete the deployments using below commands.
```shell script
kubectl delete -f deployments/category-deployment.yaml
kubectl delete -f deployments/business-deployment.yaml
```
- Execute the below command in kubernetes cluster to set your default namespace
```shell script
kubectl config set-context --current --namespace=<your-name>
```
- All the deployments could be verified by using the following commands one by one
```shell script
kubectl get pv
kubectl get pvc
kubectl get secret
kubectl get deployment
kubectl get pod
kubectl get service
```
- From the output of the last command, get the url of business and category services. Then access the services using "http//\<external-ip>:Port" on browser.
- If the service is of type *NodePort*, use the following command to port forward the services to access it. 
```shell script
kubectl port-forward service/business 8081:8081
```
- You can execute the above command in a terminal and access the application using url http://localhost:8081. To stop port forward use CTRL+C
--------------
Distributed App Ends
---------------