## Project Lombok and MapStruct

### Project Lombok History

- Started by Reinier Zwitserloot - @surial on Twitter and Roel Spilker before 2009
- Why “Lombok”? Java is also an island in Indonesia. Lombok is the second island
  east of the Island Java.
- Lombok is also Indonesian for chilli.
- Hence tag line - “Spicing up your Java”

### How Lombok Works

- Hooks in via the Annotation processor API
- The AST (raw source code) is passed to Lombok for code generation before the Java
  complier continues
- Thus, produces properly compiled Java code in conjunction with the Java compiler
- Under the ‘target/classes’ you can view the complied class files
- IntelliJ will decompile to show you the source code

### Project Lombok Configuration

- Add dependency in `POM.xml`

```
<dependency>
    <groupId>org.project.lombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

- Enable Annotation Processor in IDE

  - Preference -> Build -> Compiler -> Annotaion Processor -> Enable Annotation Processing

- Add Class annotation in Entity/DTO whereever we want to auto generate

### Overview of MapStruct

- Removing the need to add mapping b/w DTOs and ENTITY (and vice versa)

- MapStruct started in 2014, with the first release in 2017
- Founded by Gunnar Morling, who is the spec lead for Hibernate on Bean Validation 2.0
- Current release 1.3.0.Final - February, 2019
- MapStruct is a code generator for mapping between Java bean types
- ie BeerDto -> Beer (Entity)
- Like Project Lombok, MapStruct hooks into annotation processing to generate code
- Unlike Project Lombok, MapStruct will generate source code

- MapStruct follows a convention over configuration approach
- MapStruct works by you declaring an interface, MapStruct generates the implementation
- Properties of the same name / same type will automatically get mapped
  - Huge time saver!
- Different property names are configured via annotation properties
- Different types can be referenced with additional mapper implementations
- If present, MapStruct will use builders

- Mappers can reference other mappers
  - ie an Order Mapper can use the Order Line Mapper
- Can be configured to annotate generated mappers as Spring Components
  - Useful for Dependency Injection with Spring
- Can use default methods on interfaces
- Complex mappings can be done via abstract classes

### MapStruct Configuration

- Add dependency in `POM.xml`

```
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.3.0.final</version>
</dependency>
```

- Add Maven Compiler configuration

```
//Add version
<properties>
		<java.version>11</java.version>
		<mapstruct.version>1.4.2.Final</mapstruct.version>
		<org.lombok.version>1.18.18</org.lombok.version>
	</properties>

.
.
.
//Add plugin config
<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.8.0</version>
				<configuration>
					<annotationProcessorPaths>
						<path>
							<groupId>org.mapstruct</groupId>
							<artifactId>mapstruct-processor</artifactId>
							<version>${mapstruct.version}</version>
						</path>
						<path>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok</artifactId>
							<version>${org.lombok.version}</version>
						</path>
						<path>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok-mapstruct-binding</artifactId>
							<version>0.2.0</version>
						</path>
					</annotationProcessorPaths>
					<compilerArgs>
						<compilerArg>-Amapstruct.defaultComponentModel=spring</compilerArg>
					</compilerArgs>
				</configuration>
			</plugin>
```

- Add Mapper Interface

```
@Mapper(uses = {DateMapper.class})
public interface BeerMapper {
    BeerDto beerToBeerDto(Beer beer);
    Beer beerDtoToBeer(BeerDto dto);
}

```

### Adding Circle CI to Beer Service Project

```
# Java Maven CircleCI 2.0 configuration file
#
# Check https://circleci.com/docs/2.0/language-java/ for more details
#
version: 2
jobs:
    build:
        docker:
            # specify the version you desire here
            - image: circleci/openjdk:11-browsers-legacy

            # Specify service dependencies here if necessary
            # CircleCI maintains a library of pre-built images
            # documented at https://circleci.com/docs/2.0/circleci-images/
            # - image: circleci/postgres:9.4

        working_directory: ~/repo

        environment:
            # Customize the JVM maximum heap limit
            MAVEN_OPTS: -Xmx3200m

        steps:
            - checkout

            # Download and cache dependencies
            - restore_cache:
                  keys:
                      - v1-dependencies-{{ checksum "pom.xml" }}
                      # fallback to using the latest cache if no exact match is found
                      - v1-dependencies-

            - run: mvn dependency:go-offline

            - save_cache:
                  paths:
                      - ~/.m2
                  key: v1-dependencies-{{ checksum "pom.xml" }}

            # run tests!
            - run: mvn integration-test
```

### KEY TERMS

- Like Project Lombok, MapStruct hooks into annotation processing to generate code
- Unlike Project Lombok, MapStruct will generate source code
