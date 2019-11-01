[![codecov.io](https://codecov.io/github/derjust/spring-data-dynamodb/coverage.svg?branch=master)](https://codecov.io/github/derjust/spring-data-dynamodb?branch=master) [![Build Status](https://travis-ci.org/derjust/spring-data-dynamodb.svg?branch=master)](https://travis-ci.org/derjust/spring-data-dynamodb) 
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.derjust/spring-data-dynamodb/badge.svg)](https://search.maven.org/search?q=g:com.github.derjust)
[![Developer Workspace](https://codenvy.io/factory/resources/codenvy-contribute.svg)](https://codenvy.io/f?user=userrzonfqnofgrmfxjx&amp;name=spring-data-dynamodb)
[![Donation badge](https://img.shields.io/badge/Donate-%F0%9F%92%B8-DAA520.svg)](DONATION.md)



# Spring  Data DynamoDB #

<img align="left" src="https://derjust.github.io/spring-data-dynamodb/banner/spring-data-dynamodb.png" />

The primary goal of the [Spring® Data](https://projects.spring.io/spring-data/) project is to make it easier to build Spring-powered applications that use data access technologies.

This module deals with enhanced support for a data access layer built on [AWS DynamoDB](https://aws.amazon.com/dynamodb/).

Technical infos can be found on the [project page](https://derjust.github.io/spring-data-dynamodb/).

## Supported Features ##

* Implementation of [CRUD methods for](https://docs.spring.io/spring-data/commons/docs/current/reference/html/#repositories.definition) DynamoDB Entities
* Dynamic query generation from [query method names](https://docs.spring.io/spring-data/commons/docs/current/reference/html/#repositories.query-methods.query-creation) ([Supported keywords and comparison operators](https://github.com/derjust/spring-data-dynamodb/wiki/Supported-Spring-Data-Comparison-Operators))
* [Projections](https://github.com/derjust/spring-data-dynamodb/wiki/Projections)
* Possibility to integrate [custom repository code](https://github.com/derjust/spring-data-dynamodb/wiki/Custom-repository-implementations)
* Easy Spring annotation based integration
* [REST support](https://github.com/derjust/spring-data-dynamodb-examples/blob/master/README-rest.md) via [spring-data-rest](https://projects.spring.io/spring-data-rest/)

## Demo application ##

For a demo of spring-data-dynamodb, using spring-data-rest to showcase DynamoDB repositories exposed with REST,
please see [spring-data-dynamodb-examples](https://github.com/derjust/spring-data-dynamodb-examples).

## Quick Start ##

Download the JAR though [Maven Central](http://mvnrepository.com/artifact/com.github.derjust/spring-data-dynamodb) ([`SNAPSHOT` builds](https://oss.sonatype.org/content/repositories/snapshots/com/github/derjust/spring-data-dynamodb/) are available via the [OSSRH snapshot repository](https://github.com/derjust/spring-data-dynamodb/wiki/Access-to-releases) ):

```xml
<dependency>
  <groupId>com.github.derjust</groupId>
  <artifactId>spring-data-dynamodb</artifactId>
  <version>5.1.0</version>
</dependency>
```

Setup DynamoDB configuration as well as enabling Spring-Data DynamoDB repository support via Annotation ([XML-based configuration](wiki/Quick-Start---XML-based-configuration))

Create a DynamoDB entity [User](https://github.com/derjust/spring-data-dynamodb-examples/blob/master/src/main/java/com/github/derjust/spring_data_dynamodb_examples/simple/User.java) for this table:

```java
@DynamoDBTable(tableName = "User")
public class User {

	private String id;
	private String firstName;
	private String lastName;

	public User() {
		// Default constructor is required by AWS DynamoDB SDK
	}

	public User(String firstName, String lastName) {
		this.firstName = firstName;
		this.lastName = lastName;
	}

	@DynamoDBHashKey
	@DynamoDBAutoGeneratedKey
	public String getId() {
		return id;
	}

	@DynamoDBAttribute
	public String getFirstName() {
		return firstName;
	}

	@DynamoDBAttribute
	public String getLastName() {
		return lastName;
	}

	//setter & hashCode & equals
}
```

Create a CRUD repository interface [UserRepository](https://github.com/derjust/spring-data-dynamodb-examples/blob/master/src/main/java/com/github/derjust/spring_data_dynamodb_examples/simple/UserRepository.java):

```java
@EnableScan
public interface UserRepository extends CrudRepository<User, String> {
  List<User> findByLastName(String lastName);
  List<User> findByFirstName(String firstName);
}
```

or for paging and sorting...

```java
public interface PagingUserRepository extends PagingAndSortingRepository<User, String> {
	Page<User> findByLastName(String lastName, Pageable pageable);
	Page<User> findByFirstName(String firstName, Pageable pageable);

	@EnableScan
	@EnableScanCount
	Page<User> findAll(Pageable pageable);
}
```

Create the configuration class [DynamoDBConfig](https://github.com/derjust/spring-data-dynamodb-examples/blob/master/src/test/java/com/github/derjust/spring_data_dynamodb_examples/simple/UserRepositoryIT.java#L61):
```java
@Configuration
@EnableDynamoDBRepositories(basePackageClasses = UserRepository.class)
public static class DynamoDBConfig {

	@Value("${amazon.aws.accesskey}")
	private String amazonAWSAccessKey;

	@Value("${amazon.aws.secretkey}")
	private String amazonAWSSecretKey;

	public AWSCredentialsProvider amazonAWSCredentialsProvider() {
		return new AWSStaticCredentialsProvider(amazonAWSCredentials());
	}

	@Bean
	public AWSCredentials amazonAWSCredentials() {
		return new BasicAWSCredentials(amazonAWSAccessKey, amazonAWSSecretKey);
	}

	@Bean
	public DynamoDBMapperConfig dynamoDBMapperConfig() {
		return DynamoDBMapperConfig.DEFAULT;
	}

	@Bean
	public DynamoDBMapper dynamoDBMapper(AmazonDynamoDB amazonDynamoDB, DynamoDBMapperConfig config) {
		return new DynamoDBMapper(amazonDynamoDB, config);
	}

	@Bean
	public AmazonDynamoDB amazonDynamoDB() {
		return AmazonDynamoDBClientBuilder.standard().withCredentials(amazonAWSCredentialsProvider())
				.withRegion(Regions.US_EAST_1).build();
	}
}
```

And finally write a test client [UserRepositoryIT](https://github.com/derjust/spring-data-dynamodb-examples/blob/master/src/test/java/com/github/derjust/spring_data_dynamodb_examples/simple/UserRepositoryIT.java) or start calling it from your existing Spring code.


The full source code is available at [spring-data-dynamodb-examples' simple example](https://github.com/derjust/spring-data-dynamodb-examples/blob/master/README-simple.md)

## More
More sample code can be found in the [spring-data-dynamodb-examples](https://github.com/derjust/spring-data-dynamodb-examples) project.

Advanced topics can be found in the [wiki](https://github.com/derjust/spring-data-dynamodb/wiki).


## Version & Spring Framework compatibility ##

The major and minor number of this library refers to the compatible Spring framework version. The build number is used as specified by SEMVER.

API changes will follow SEMVER and loosly the Spring Framework releases.

| `spring-data-dynamodb` version  | Spring Boot compatibility      |Spring Framework compatibility  | Spring Data compatibility |
| ------------------------------- | ------------------------------ | ------------------------------ | ------------------------- |
| 1.0.x                           |                                | >= 3.1 && < 4.2                |                           |
| 4.2.x                           | >= 1.3.0 && < 1.4.0            | >= 4.2 && < 4.3                | Gosling-SR1               |
| 4.3.x                           | >= 1.4.0 && < 2.0              | >= 4.3 && < 5.0                | Gosling-SR1               |
| 4.4.x                           | >= 1.4.0 && < 2.0              | >= 4.3 && < 5.0                | Hopper-SR2                |
| 4.5.x                           | >= 1.4.0 && < 2.0              | >= 4.3 && < 5.0                | Ingalls                   |
| 5.0.x                           | >= 2.0 && < 2.1                | >= 5.0 && < 5.1                | Kay-SR1                   |
| 5.1.x                           | == 2.1                         | >= 5.1                         | Lovelace-SR1              |
| 5.2.x                           | >= 2.2                         | >= 5.2                         | Moore-RELASE              |


`spring-data-dynamodb` depends directly on `spring-data` as also `spring-context`, `spring-data` and `spring-tx`.

`compile` and `runtime` dependencies are kept to a minimum to allow easy integartion, for example into 
Spring-Boot projects.

## History
The code base has some history already in it - let's clarify it a bit:
* The code base was established under [github.com/michaellavelle/spring-data-dynamodb)](https://github.com/michaellavelle/spring-data-dynamodb)
* It was forked and further maintained under [github.com/derjust/spring-data-dynamodb)](https://github.com/derjust/spring-data-dynamodb) 
    * Available in Maven Central under [`com.github.derjust:spring-data-dynamodb`](http://central.maven.org/maven2/com/github/derjust/spring-data-dynamodb/)

The Java package name/XSD namespace never changed from `org.socialsignin.spring.data.dynamodb`.
But the XSD is now also available at [`https://derjust.github.io/spring-data-dynamodb/spring-dynamodb-1.0.xsd`](https://derjust.github.io/spring-data-dynamodb/spring-dynamodb-1.0.xsd).
