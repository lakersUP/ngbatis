
<!--
Copyright (c) 2022 All project authors and nebula-contrib. All rights reserved.

This source code is licensed under Apache 2.0 License.
-->

# NgBatis

<p align="center">
  <br> English | <a href="README-CN.md">中文</a>
</p>

- [NgBatis Docs](https://nebula-contrib.github.io/ngbatis/)
- [NgBatis 文档](https://graph-cn.github.io/ngbatis-docs/)

## What is NgBatis

**NgBatis** is a database ORM framework base [NebulaGraph](https://github.com/vesoft-inc/nebula) + spring-boot, which takes advantage of the [mybatis’](https://github.com/mybatis/mybatis-3) fashion development, including some de-factor operations in single table and vertex-edge, like [mybatis-plus](https://github.com/baomidou/mybatis-plus).

If you prefer JPA, [graph-ocean](https://github.com/nebula-contrib/graph-ocean) is a good choice.

## How it works

See [EXECUTION-PROCESS.md](./EXECUTION-PROCESS.md)

## Requirements

- Springboot
- Maven
- Java 8+

## Version matching

  NgBatis | nebula-java | JDK | Springboot | Beetl
  ---|-------------|---|------------|---
  2.0.1 | 3.8.3       | 8 | 2.7.0 | 3.15.10.RELEASE
  2.0.0-beta | 3.8.3       | 8 | 2.7.0 | 3.15.10.RELEASE
  1.3.0 | 3.8.3       | 8 | 2.7.0 | 3.15.10.RELEASE
  1.3.0-jdk17 | 3.8.3       | 17 | 3.0.7 | 3.15.10.RELEASE
  1.2.2 | 3.6.0       | 8 | 2.7.0 | 3.15.10.RELEASE
  1.2.2-jdk17 | 3.6.0       | 17 | 3.0.7 | 3.15.10.RELEASE
  1.2.1 | 3.6.0       | 8 | 2.7.0 | 3.15.10.RELEASE
  1.2.0-jdk17 | 3.6.0       | 17 | 3.0.7 | 3.15.10.RELEASE
  1.2.0 | 3.6.0       | 8 | 2.7.0 | 3.15.10.RELEASE
  1.1.5 | 3.5.0       | 8 | 2.7.0 | 3.1.8.RELEASE
  1.1.4 | 3.5.0       | 8 | 2.7.0 | 3.1.8.RELEASE

### SNAPSHOT

  NgBatis | nebula-java | JDK | Springboot | Beetl
  ---|-------------|---|------------|---
  1.2.2-jdk17-SNAPSHOT | 3.6.0       | 17 | 3.0.7 | 3.15.10.RELEASE
  1.2.2-SNAPSHOT | 3.6.0       | 8 | 2.7.0 | 3.15.10.RELEASE

> The third-party dependencies may differ within the same snapshot version.

## How to use

> You could refer to ngbatis-demo in this repo.

- Include in your `pom.xml`
  - Maven

    ```xml
        <dependency>
          <groupId>org.nebula-contrib</groupId>
          <artifactId>ngbatis</artifactId>
          <version>2.01</version>
        </dependency>
    ```

  - Gradle

    ```groovy
    implementation 'org.nebula-contrib:ngbatis:2.0.1'
    ```

- Referring to [ngbatis-demo](./ngbatis-demo), which was smoothly integrated with spring-boot. The API examples could be found under the test of it for all features of ngbatis.

- Configure the NebulaGraph Database

  Configure `application.yml` with the host and credential to enable access to the NebulaGraph Cluster.

```yml
nebula:
  ngbatis:
    session-life-length: 300000 # since v1.1.2
    check-fixed-rate: 300000 # since v1.1.2
    # space name needs to be informed through annotations(@Space) or xml(space="test")
    # default false(false: Session pool map will not be initialized)
    use-session-pool: false # since v1.1.2
  hosts: 127.0.0.1:19669, 127.0.0.1:9669
  username: root
  password: nebula
  space: test
  pool-config:
    min-conns-size: 0
    max-conns-size: 10
    timeout: 0
    idle-time: 0
    interval-idle: -1
    wait-time: 0
    min-cluster-health-rate: 1.0
    enable-ssl: false
```

- Dynamically register beans

```java
@SpringBootApplication(scanBasePackages = { "org.nebula", "your.domain"})
public class YourApplication {
    public static void main(String[] args) {
        new SpringApplication(YourApplication.class).run(args);
    }
}
```

> If SpringCloud is used in your project,
> please use `@ComponentScan( basePackages = {"org.nebula.contrib", "your.domain"} )` instead.

## Examples

### a. The MyBatis fashion(compose nGQL queries)

#### a.1 Declare the data access interface

```java
package ye.weicheng.ngbatis.demo.repository;

import ye.weicheng.ngbatis.demo.pojo.Person;
import java.util.List;
import java.util.Map;
import java.util.Set;

public interface TestRepository {
    // new features from v1.2.0
    Integer returnAge(@Param("person")Person person);

    Person selectPerson();
    Person selectByPerson(Person person);
    List<Person> selectAgeGt(Integer age);
    List<String> selectListString();
    List<Map> selectPersonsMap();
    Map<String, Object> selectTriple();
}

```

#### a.2 The query statments

`resource/mapper/TestRepository.xml`

```xml
<mapper
    namespace=
    "ye.weicheng.ngbatis.demo.repository.TestRepository"
>
    <!-- new features from v1.2.0 start -->
    <nGQL id="include-test-value">
        ${myInt}
    </nGQL>

    <nGQL id="ngql-return-age">
        RETURN @ng.include('include-test-value',{'myInt':age});
    </nGQL>

    <!--
    The same as: 
        RETURN ${person.age};
    You can try extracting more common and meaningful scripts.
    -->
    <select id="returnAge" resultType="java.lang.Integer">
        @ng.include('ngql-return-age',person);
    </select>
    <!-- new features from v1.2.0 end -->

    <select id="selectPerson" resultType="ye.weicheng.ngbatis.demo.pojo.Person">
        match (v:person) return v.person.name as name, v.person.age as age limit 1
    </select>

    <select id="selectAgeGt" resultType="ye.weicheng.ngbatis.demo.pojo.Person">
        MATCH (n: person)
        WHERE n.person.age > $p0
        RETURN n
        LIMIT 100
    </select>


    <select id="selectByPerson" resultType="ye.weicheng.ngbatis.demo.pojo.Person">
        MATCH (n: person)
        WHERE n.person.name == $p0.name
        RETURN n
        LIMIT 100
    </select>

    <select id="selectListString" resultType="java.lang.String">
        match (v:person) return v.person.name as name limit 100
    </select>

    <select id="selectPersonsMap" resultType="java.util.Map">
        match (v:person) return v.person.name as name, v.person.age  as age limit 100
    </select>

    <select id="selectTriple" resultType="java.util.Map">
        MATCH (n: person)-[r: like]->(n2: person)
        RETURN n, r, n2
        LIMIT 100
    </select>

    <!-- 
        More complex `nGQL` may need to be fully tested.
        The two-layer object data structure of the project I am currently using is also satisfying.
        `Path` is not yet supported because it can basically be handled by the `n, r, n2` structure in development.
    -->

</mapper>
```

### b. The MyBatis-plus fashion

#### b.1 model-vertex

```java
package com.example.model.vertex.Person;

import lombok.Data;
import javax.persistence.Id;
import javax.persistence.Table;

@Data
@Table(name = "person")
public class Person {
    @Id
    private String name;
    private Integer age;
}
```

#### b.2 model-edge

```java
package com.example.model.edge.Like;

import lombok.AllArgsConstructor;
import lombok.NoArgsConstructor;
import lombok.Data;
import javax.persistence.Table;

@Data
@Table(name = "like")
@NoArgsConstructor
@AllArgsConstructor
public class Like {
    private Double likeness;
}
```

#### b.3 dao

```java
package com.example.dao;

import org.nebula.contrib.ngbatis.proxy.NebulaDaoBasic;
import com.example.model.vertex.Person;

public interface PersonDao extends NebulaDaoBasic<Person, String>{}
```

#### b.4 xml

> Note, this is a mandatory requirement.

```xml
<mapper
    namespace=
    "com.example.dao.PersonDao"
>
</mapper>
```

#### b.5 service

```java
package com.example.service;

import org.nebula.contrib.ngbatis.utils.Page;
import com.example.dao.PersonDao;
import com.example.model.vertex.Person;
import com.example.model.edge.Like;

@Service
public class PersonServiceImpl {

    @Autowired private PersonDao dao;

    public void demos() {
        // Implement two node insertions
        Person tom = new Person();
        tom.setName("Tom");
        dao.insert( tom ); 
        
        Person jerry = new Person();
        jerry.setName( "Jerry" );
        dao.insert( jerry );

        // Establishing the relationship between two nodes
        Like like = new Like( 0.99999 );
        dao.insertEdge( tom, like, jerry );

        // Find people who like jerry
        String jerryId = jerry.getName();
        List<Person> whoLikeJerry = dao.listStartNodes( Like.class, jerryId );

        // Find the only people who like jerry, Non-Unique Times Error。（Limited to scenarios where there is only one upstream for a given relationship）
        Person tom = dao.startNode( Like.class, jerryId );

        // See the Like relationship between Tom and Jerry
        String tomId = tom.getName();
        Boolean tomLikeJerry = dao.existsEdge( tomId, Like.class, jerryId ); // true
        Boolean jerryLikeTom = dao.existsEdge( jerryId, Like.class, tomId ); // false
        // Poor Tom

        // Find all information by Tom's name
        Person tomDb = dao.selectById( "Tom" );

        // Search by page
        Page<Person> page = new Page<>();
        List<Person> personPage = dao.selectPage( page );
        page.getTotal(); // 2 rows， Tom and Jerry
        Boolean theyAreFamily = page.getRows() == personPage; // true

        // The story always wants to have a good ending
        dao.insertEdge( jerry, like, tom );

        // More base class operations are still under development；Expectations
    }


}

```

### c. Entity Direct Search

#### c.1 Entity class

##### c.1.1 Vertex Entity

- Extends the `GraphBaseVertex` class identifier as a vertex entity
- The name attribute of `@Tag` indicates the Tag of the vertex entity

```java

@Tag(name = "player")
public class Player extends GraphBaseVertex {

  @Id
  private String id;

  private String name;

  private Integer age;
    
  ...

}

```

Specific reference `ye.weicheng.ngbatis.demo.pojo.vertex` vertex entities under the package sample.

##### c.1.2 Edge Entity

- Extends the `GraphBaseEdge` class to identify edge entities
- The name attribute of `@EdgeType` indicates the type of the edge entity
- `@Id` (Optional, if the uniqueness of an edge of the same type between two nodes is determined by the source node id and the destination node id, the current attribute can be omitted)
- `@SrcId` (optional, if you do not need to obtain the source node id of the relationship, you can omit the current attribute)
- `@DstId` (Optional, if you do not need to get the target node id of the relationship, you can omit the current attribute)

```java

@EdgeType(name = "serve")
public class Serve extends GraphBaseEdge {

  @Id 
  private Long rank;

  @SrcId 
  private String srcId;

  @DstId 
  private String dstId;

  @Column(name = "start_year")
  private Integer startYear;
  @Column(name = "end_year")
  private Integer endYear;

  ...
  
}

```

Specific reference `ye.weicheng.ngbatis.demo.pojo.edge` edge entities under the package sample.

#### c.2 The method is now provided

##### c.2.1 About vertex entity

API | 用法说明
--|--
queryIdsByProperties()                              | Query a collection of vertex ids for a particular Tag or attribute
queryVertexById()                                   | Query a single vertex for a specific vertex Id
queryVertexByTag()                                  | Query a collection of vertices  for a specific Tag
queryVertexByProperties()                           | Query a collection of vertexes for a specific property
queryAllAdjacentVertex(Class<?>... edgeClass)       | Query a collection of all neighboring vertexes of a particular vertex, specifying one or more edge types that connect the two vertexes
queryIncomingAdjacentVertex(Class<?>... edgeClass)  | Query the set of adjacent vertexes in the direction of the incoming edge of a particular vertex, specifying one or more edge types that connect two vertexes
queryOutgoingAdjacentVertex(Class<?>... edgeClass)  | Query the set of adjacent vertexes in the direction of the edge of a particular vertex, specifying one or more edge types that connect two vertexes
queryNeighborIdsWithHopById(int m, int n, Class<?>... edgeClass) | Query a collection of vertex ids within a specified number of hops for a particular vertex, specifying one or more edge types that connect two vertexes
queryConnectedEdgesById(Direction direction)        | Query the set of all edges associated with a particular vertex, specifying the direction and type of the edge
queryPathFromVertex(Direction direction)            | Query the collection of all paths associated with a particular vertex, specifying the direction of the edge
queryFixedLengthPathFromVertex(Integer maxHop, Direction direction, Class<?>... edgeClass) | Query a set of fixed-length paths from a specific vertex, specifying the maximum number of steps, the direction of the edge, and the type of the edge
queryVariableLengthPathFromVertex(Integer minHop, Integer maxHop,   Direction direction, Class<?>... edgeClass) | Query a set of variable-length paths from a specific vertex, specifying the minimum number of steps, the maximum number of steps, the direction of the edge, and the type of the edge
queryShortestPathFromSrcAndDst(Integer maxHop,   Direction direction, T v2) | Query any shortest path from a specific vertex, specifying the number of steps, the direction of the edge, and the end vertex entity
queryAllShortestPathsFromSrcAndDst(Integer maxHop,   Direction direction, T v2) | Query the set of all shortest paths from this vertex, specifying the number of steps, the direction of the edge, and the end vertex entity
queryVertexCountByTag()                             | Query the number of vertexes for a specific Tag

For specific implementation, see the point entity base class `GraphBaseVertex` under the `org.nebula.contrib.ngbatis.base` package.

##### c.2.2 About edge entity

API | 用法说明
--|--
queryEdgeByType(Direction direction)                       | Query a set of edges of a specific type and direction
queryEdgeWithSrcAndDstByProperties(T srcVertex, Direction direction, T dstVertex) | Query a set of edges for a particular property
queryEdgePropertiesBySrcAndDstId()                         | Query a set of edges for a specific always vertex id
queryEdgeCountByType()                                     | Query the number of edges for a specific Type

For specific implementation, see the point entity base class `GraphBaseEdge` under the `org.nebula.contrib.ngbatis.base` package.

#### c.3 test

```java

@Test
public void testVertex(){
    Player srcPlayer = new Player();
    //Query all Player vertices that meet the condition name = "Vince Carter"
    srcPlayer.setName("Vince Carter");
    List<Player> vertices = player.queryVertexByProperties();
}

@Test
public void testEdge(){
    Serve serve = new Serve();
    //Query the Server edge whose starting point ID is player100 and the end point ID is team204.
    serve.setSrcId("player100");
    serve.setDstId("team204");
    Serve edge = serve.queryEdgeWithSrcAndDstByProperties();
    //Query the edges of Serve type and direction "->"
    List<Serve> edges = serve.queryEdgeByType(Direction.NULL);
}

```

For specific usage examples of each direct inspection method, please refer to the `NebulaGraphBasicTests` test class in ngbatis-demo.

## Upstream projects

- [beetl](https://gitee.com/xiandafu/beetl), BSD-3, we proudly use the beetl template language as our template engine, which is consumed in binary package(as is).

## Community

- English: [![Slack](https://img.shields.io/badge/Slack-9F2B68?style=for-the-badge&logo=slack&logoColor=white)](https://join.slack.com/t/nebulagraph/shared_invite/zt-7ybejuqa-NCZBroh~PCh66d9kOQj45g)
- Chinese: [![WeChat](https://img.shields.io/badge/WeChat-7BB32E?style=for-the-badge&logo=wechat&logoColor=white)](https://github.com/nebula-contrib/ngbatis/issues/270)

## License

NGBATIS is under the [Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0).
