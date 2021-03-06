Spring Data Elasticsearch
=========================

Spring Data implementation for ElasticSearch

Spring Data makes it easier to build Spring-powered applications that use new data access technologies such as non-relational databases, map-reduce frameworks, and cloud based data services as well as provide improved support for relational database technologies.

The Spring Data Elasticsearch project provides integration with the [elasticsearch](http://www.elasticsearch.org/) search engine.

Guide
------------

* [Reference Documentation](http://docs.spring.io/spring-data/elasticsearch/docs/1.0.0.M2/reference/html/)
* [PDF Documentation](http://docs.spring.io/spring-data/elasticsearch/docs/1.0.0.M2/reference/pdf/spring-data-elasticsearch-reference.pdf)
* [API Documentation](http://docs.spring.io/spring-data/elasticsearch/docs/1.0.0.M2/api/)
* [Spring Data Project](http://project.spring.io/spring-data)
* [Sample Test Application](https://github.com/BioMedCentralLtd/spring-data-elasticsearch-sample-application)
* [Spring Data Elasticsearch Google Group] (https://groups.google.com/d/forum/spring-data-elasticsearch-devs)
* For more detailed questions, use the [forum](http://forum.springsource.org/forumdisplay.php?f=80).


Quick Start
-----------
Wiki page for [Getting Started] (https://github.com/spring-projects/spring-data-elasticsearch/wiki/How-to-start-with-spring-data-elasticsearch)

### Maven configuration

Add the Maven dependency:

```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-elasticsearch</artifactId>
    <version>1.0.0.RC1</version>
</dependency>

<repository>
  <id>spring-libs-milestone</id>
  <name>Spring Milestone Repository</name>
  <url>http://repo.spring.io/libs-milestone</url>
</repository>
```

If you'd rather like the latest snapshots of the upcoming major version, use our Maven snapshot repository and declare
the appropriate dependency version.

```xml
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-elasticsearch</artifactId>
  <version>1.0.0.BUILD-SNAPSHOT</version>
</dependency>

<repository>
  <id>spring-libs-snapshot</id>
  <name>Spring Snapshot Repository</name>
  <url>http://repo.spring.io/libs-snapshot</url>
</repository>
```

### ElasticsearchRepository
A default implementation of ElasticsearchRepository, aligning to the generic Repository Interfaces, is provided. Spring can do the Repository implementation for you depending on method names in the interface definition.

The ElasticsearchCrudRepository extends PagingAndSortingRepository

```java
    public interface ElasticsearchCrudRepository<T, ID extends Serializable> extends ElasticsearchRepository<T, ID>, PagingAndSortingRepository<T, ID> {
    }
```

Extending ElasticsearchRepository for custom methods

```java
    public interface BookRepository extends Repository<Book, String> {

        List<Book> findByNameAndPrice(String name, Integer price);

        List<Book> findByNameOrPrice(String name, Integer price);
        
        Page<Book> findByName(String name,Pageable page);

        Page<Book> findByNameNot(String name,Pageable page);

        Page<Book> findByPriceBetween(int price,Pageable page);

        Page<Book> findByNameLike(String name,Pageable page);

        @Query("{\"bool\" : {\"must\" : {\"term\" : {\"message\" : \"?0\"}}}}")
        Page<Book> findByMessage(String message, Pageable pageable);
    }
```
Indexing a single document with Repository

```java
        @Autowired
        private SampleElasticsearchRepository repository;

        String documentId = "123456";
        SampleEntity sampleEntity = new SampleEntity();
        sampleEntity.setId(documentId);
        sampleEntity.setMessage("some message");

        repository.save(sampleEntity);
```

Indexing multiple Document(bulk index) using Repository

```java
        @Autowired
        private SampleElasticsearchRepository repository;

        String documentId = "123456";
        SampleEntity sampleEntity1 = new SampleEntity();
        sampleEntity1.setId(documentId);
        sampleEntity1.setMessage("some message");

        String documentId2 = "123457"
        SampleEntity sampleEntity2 = new SampleEntity();
        sampleEntity2.setId(documentId2);
        sampleEntity2.setMessage("test message");

        List<SampleEntity> sampleEntities = Arrays.asList(sampleEntity1, sampleEntity2);

        //bulk index
        repository.save(sampleEntities);
```


### ElasticsearchTemplate

ElasticsearchTemplate is the central support class for elasticsearch operations.

Indexing a single document using Elasticsearch Template

```java
        String documentId = "123456";
        SampleEntity sampleEntity = new SampleEntity();
        sampleEntity.setId(documentId);
        sampleEntity.setMessage("some message");
        IndexQuery indexQuery = new IndexQueryBuilder().withId(sampleEntity.getId()).withObject(sampleEntity).build();
        elasticsearchTemplate.index(indexQuery);
```

Indexing multiple Document(bulk index) using Elasticsearch Template

```java
        @Autowired
        private ElasticsearchTemplate elasticsearchTemplate;

        List<IndexQuery> indexQueries = new ArrayList<IndexQuery>();
        //first document
        String documentId = "123456";
        SampleEntity sampleEntity1 = new SampleEntity();
        sampleEntity1.setId(documentId);
        sampleEntity1.setMessage("some message");

        IndexQuery indexQuery1 = new IndexQueryBuilder().withId(sampleEntity1.getId()).withObject(sampleEntity1).build();
        indexQueries.add(indexQuery1);

        //second document
        String documentId2 = "123457";
        SampleEntity sampleEntity2 = new SampleEntity();
        sampleEntity2.setId(documentId2);
        sampleEntity2.setMessage("some message");

        IndexQuery indexQuery2 = new IndexQueryBuilder().withId(sampleEntity2.getId()).withObject(sampleEntity2).build()
        indexQueries.add(indexQuery2);

        //bulk index
        elasticsearchTemplate.bulkIndex(indexQueries);
```

Searching entities using Elasticsearch Template

```java
        @Autowired
        private ElasticsearchTemplate elasticsearchTemplate;

        SearchQuery searchQuery = new NativeSearchQueryBuilder()
        .withQuery(queryString(documentId).field("id"))
        .build();
        Page<SampleEntity> sampleEntities = elasticsearchTemplate.queryForPage(searchQuery,SampleEntity.class);
```

### XML Namespace

You can set up repository scanning via xml configuration, which will happily create your repositories.

Using Node Client

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
       xsi:schemaLocation="http://www.springframework.org/schema/data/elasticsearch http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch-1.0.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd">

    <elasticsearch:node-client id="client" local="true"/>

    <bean name="elasticsearchTemplate" class="org.springframework.data.elasticsearch.core.ElasticsearchTemplate">
        <constructor-arg name="client" ref="client"/>
    </bean>

</beans>
```

Using Transport Client

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
       xsi:schemaLocation="http://www.springframework.org/schema/data/elasticsearch http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch-1.0.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd">

    <elasticsearch:repositories base-package="com.xyz.acme"/>

    <elasticsearch:transport-client id="client" cluster-nodes="ip:9300,ip:9300" cluster-name="elasticsearch" />

    <bean name="elasticsearchTemplate" class="org.springframework.data.elasticsearch.core.ElasticsearchTemplate">
        <constructor-arg name="client" ref="client"/>
    </bean>

</beans>
```

## Help Pages

* [Geo distance and location search](https://github.com/spring-projects/spring-data-elasticsearch/wiki/Geo-indexing-and-request)
* [Custom object mapper](https://github.com/spring-projects/spring-data-elasticsearch/wiki/Custom-ObjectMapper)

## Contributing to Spring Data

Here are some ways for you to get involved in the community:

* Get involved with the Spring community on the Spring Community Forums.  Please help out on the [forum](http://forum.springsource.org/forumdisplay.php?f=80) by responding to questions and joining the debate.
* Create [JIRA](https://jira.springframework.org/browse/DATAES) tickets for bugs and new features and comment and vote on the ones that you are interested in.  
* Github is for social coding: if you want to write code, we encourage contributions through pull requests from [forks of this repository](http://help.github.com/forking/). If you want to contribute code this way, please reference a JIRA ticket as well covering the specific issue you are addressing.
* Watch for upcoming articles on Spring by [subscribing](http://www.springsource.org/node/feed) to springframework.org

Before we accept a non-trivial patch or pull request we will need you to sign the [contributor's agreement](https://support.springsource.com/spring_committer_signup).  Signing the contributor's agreement does not grant anyone commit rights to the main repository, but it does mean that we can accept your contributions, and you will get an author credit if we do.  Active contributors might be asked to join the core team, and given the ability to merge pull requests.


Code formatting for [Eclipse and Intellij](https://github.com/spring-projects/spring-data-build/tree/master/etc/ide)

[More information about contributing to Spring Data](https://github.com/spring-projects/spring-data-build/blob/master/CONTRIBUTING.md)