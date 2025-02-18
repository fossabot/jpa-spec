[![Build Status](https://travis-ci.org/wenhao/jpa-spec.svg?branch=master)](https://travis-ci.org/wenhao/jpa-spec) [![Coverage Status](https://coveralls.io/repos/github/wenhao/jpa-spec/badge.svg?branch=master)](https://coveralls.io/github/wenhao/jpa-spec?branch=master) [![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fwenhao%2Fjpa-spec.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fwenhao%2Fjpa-spec?ref=badge_shield)
[:cn:](./README_CN.md)

# jpa-spec

Inspired by [Legacy Hibernate Criteria Queries], while this should be considered deprecated vs JPA APIs,

but it still productive and easily understandable. Build on Spring Data JPA and simplify the dynamic query process.

### Features

* Compatible with Spring Data JPA and JPA 2.1 interface.
* Equal/NotEqual/Like/NotLike/In/NotIn support multiple values, Equal/NotEqual support **Null** value.
* Each specification support join query(left joiner).
* Support custom specification.
* Builder style specification creator.
* Support pagination and sort builder.

### Docs

| English Version | Chinese Version |
|-----------------|-----------------|
| [Latest]        | [最新]          |
| [3.2.5]         | [3.2.5_cn]      |
| [3.2.1]         | [3.2.1_cn]      |
| [3.1.0]         | [3.1.0_cn]      |
| [3.0.0]         | [3.0.0_cn]      |

### Gradle

```groovy
repositories {
    jcenter()
}

dependencies {
    implementation 'com.github.wenhao:jpa-spec:3.2.5'
}
```

### Maven

```xml
<dependency>
    <groupId>com.github.wenhao</groupId>
    <artifactId>jpa-spec</artifactId>
    <version>3.2.5</version>
</dependency>
```

### Build

```
./gradlew clean build
```

### Specification By Examples:

#### Each specification support three parameters:

1. **condition**: if true(default), apply this specification.
2. **property**: field name.
3. **values**: compare value with model, eq/ne/like support multiple values.

#### General Example

each Repository class should extends from two super class **JpaRepository** and **JpaSpecificationExecutor**.

```java
public interface PersonRepository extends JpaRepository<Person, Long>, JpaSpecificationExecutor<Person> {
}    
```

```java
public Page<Person> findAll(SearchRequest request) {
    Specification<Person> specification = Specifications.<Person>and()
            .eq(StringUtils.isNotBlank(request.getName()), "name", request.getName())
            .gt(Objects.nonNull(request.getAge()), "age", 18)
            .between("birthday", new Date(), new Date())
            .like("nickName", "%og%", "%me")
            .build();

    return personRepository.findAll(specification, new PageRequest(0, 15));
}
```

#### Equal/NotEqual Example

find any person nickName equals to "dog" and name equals to "Jack"/"Eric" or null value, and company is null.

**Test:** [EqualTest.java] and [NotEqualTest.java]

```java
public List<Person> findAll(SearchRequest request) {
    Specification<Person> specification = Specifications.<Person>and()
            .eq("nickName", "dog")
            .eq(StringUtils.isNotBlank(request.getName()), "name", "Jack", "Eric", null)
            .eq("company", null) //or eq("company", (Object) null)
            .build();

    return personRepository.findAll(specification);
}
```

#### In/NotIn Example

find any person name in "Jack" or "Eric" and company not in "ThoughtWorks" or "IBM".

**Test:** [InTest.java]

```java
public List<Person> findAll(SearchRequest request) {
    Specification<Person> specification = Specifications.<Person>and()
            .in("name", request.getNames())
            .notIn("company", Arrays.asList("ThoughtWorks", "IBM"))
            .build();

    return personRepository.findAll(specification);
}
```

#### Comparison Example

Support any comparison class which implements Comparable interface, find any people age bigger than 18.

**Test:** [GreatThanTest.java]

```java
public List<Person> findAll(SearchRequest request) {
    Specification<Person> specification = Specifications.<Person>and()
            .gt(Objects.nonNull(request.getAge()), "age", 18)
            .lt("birthday", new Date())
            .build();

    return personRepository.findAll(specification);
}
```

#### Between Example

find any person age between 18 and 25, birthday between someday and someday.

**Test:** [BetweenTest.java]

```java
public List<Person> findAll(SearchRequest request) {
    Specification<Person> specification = Specifications.<Person>and()
            .between(Objects.nonNull(request.getAge(), "age", 18, 25)
            .between("birthday", new Date(), new Date())
            .build();

    return personRepository.findAll(specification);
}  
```

#### Like/NotLike Example

find any person name like %ac% or %og%, company not like %ec%.

**Test:** [LikeTest.java] and [NotLikeTest.java]

```java
public Page<Person> findAll(SearchRequest request) {
    Specification<Person> specification = Specifications.<Person>and()
            .like("name", "ac", "%og%")
            .notLike("company", "ec")
            .build();

    return personRepository.findAll(specification);
}
```

#### Or

support or specifications.

**Test:** [OrTest.java]

```java
public List<Phone> findAll(SearchRequest request) {
    Specification<Person> specification = Specifications.<Person>or()
                    .like("name", "%ac%")
                    .gt("age", 19)
                    .build();

    return phoneRepository.findAll(specification);
}
```

#### Mixed `And` and `Or`

support mixed `and` and `or` specifications.

**Test:** [AndOrTest.java]

```java
public List<Person> findAll(SearchRequest request) {
    Specification<Person> specification = Specifications.<Person>and()
                    .like("name", "%ac%")
                    .predicate(Specifications.or()
                            .lt("age", 19)
                            .gt("age", 25)
                            .build())
                    .build();

    return personRepository.findAll(specification);
}
```

#### Join

each specification support association query as left join.

**Test:** [JoinTest.java]

@ManyToOne association query, find person name equals to "Jack" and phone brand equals to "HuaWei".

```java
public List<Phone> findAll(SearchRequest request) {
    Specification<Phone> specification = Specifications.<Phone>and()
        .eq(StringUtils.isNotBlank(request.getBrand()), "brand", "HuaWei")
        .eq(StringUtils.isNotBlank(request.getPersonName()), "person.name", "Jack")
        .build();

    return phoneRepository.findAll(specification);
}
```

@ManyToMany association query, find person age between 10 and 35, live in "Chengdu" street.

```java
public List<Phone> findAll(SearchRequest request) {
    Specification<Person> specification = Specifications.<Person>and()
        .between("age", 10, 35)
        .eq(StringUtils.isNotBlank(jack.getName()), "addresses.street", "Chengdu")
        .build();

    return phoneRepository.findAll(specification);
}
```

#### Custom Specification

You can custom specification to do the @ManyToOne and @ManyToMany as well.

@ManyToOne association query, find person name equals to "Jack" and phone brand equals to "HuaWei".

**Test:** [PredicateTest.java]

```java
public List<Phone> findAll(SearchRequest request) {
    Specification<Phone> specification = Specifications.<Phone>and()
        .eq(StringUtils.isNotBlank(request.getBrand()), "brand", "HuaWei")
        .predicate(StringUtils.isNotBlank(request.getPersonName()), (root, query, cb) -> {
            Path<Person> person = root.get("person");
            return cb.equal(person.get("name"), "Jack");
        })
        .build();

    return phoneRepository.findAll(specification);
}
```

@ManyToMany association query, find person age between 10 and 35, live in "Chengdu" street.

**Test:** [PredicateTest.java]

```java
public List<Phone> findAll(SearchRequest request) {
    Specification<Person> specification = Specifications.<Person>and()
        .between("age", 10, 35)
        .predicate(StringUtils.isNotBlank(jack.getName()), ((root, query, cb) -> {
            Join address = root.join("addresses", JoinType.LEFT);
            return cb.equal(address.get("street"), "Chengdu");
        }))
        .build();

    return phoneRepository.findAll(specification);
}
```

#### Sort

**Test:** [SortTest.java]

```java
public List<Person> findAll(SearchRequest request) {
    Specification<Person> specification = Specifications.<Person>and()
            .eq(StringUtils.isNotBlank(request.getName()), "name", request.getName())
            .gt("age", 18)
            .between("birthday", new Date(), new Date())
            .like("nickName", "%og%")
            .build();

    Sort sort = Sorts.builder()
        .desc(StringUtils.isNotBlank(request.getName()), "name")
        .asc("birthday")
        .build();

    return personRepository.findAll(specification, sort);
}
```

#### Pagination

find person by pagination and sort by name desc and birthday asc.

```java
public Page<Person> findAll(SearchRequest request) {
    Specification<Person> specification = Specifications.<Person>and()
            .eq(StringUtils.isNotBlank(request.getName()), "name", request.getName())
            .gt("age", 18)
            .between("birthday", new Date(), new Date())
            .like("nickName", "%og%")
            .build();

    Sort sort = Sorts.builder()
        .desc(StringUtils.isNotBlank(request.getName()), "name")
        .asc("birthday")
        .build();

    return personRepository.findAll(specification, new PageRequest(0, 15, sort));
}
```

#### Virtual View

Using **@org.hibernate.annotations.Subselect** to define a virtual view if you don't want a database table view.

There is no difference between a view and a database table for a Hibernate mapping.

**Test:** [VirtualViewTest.java]

```java
@Entity
@Immutable
@Subselect("SELECT p.id, p.name, p.age, ic.number " +
           "FROM person p " +
           "LEFT JOIN id_card ic " +
           "ON p.id_card_id=ic.id")
public class PersonIdCard {
    @Id
    private Long id;
    private String name;
    private Integer age;
    private String number;

    // Getters and setters are omitted for brevity
}    
```

```java
public List<PersonIdCard> findAll(SearchRequest request) {
    Specification<PersonIdCard> specification = Specifications.<PersonIdCard>and()
            .gt(Objects.nonNull(request.getAge()), "age", 18)
            .build();

    return personIdCardRepository.findAll(specification);
}
```

#### Projection, GroupBy, Aggregation

Spring Data JPA doesn't support **Projection**(a little but trick), **GroupBy** and **Aggregation**,

furthermore, Projection/GroupBy/Aggregation are often used for complex statistics report, it might seem like overkill to use Hibernate/JPA ORM to solve it.

Alternatively, using virtual view and give a readable/significant class name to against your problem domain may be a better option.

### Copyright and license

Copyright © 2016-2019 Wen Hao

Licensed under [Apache License]


[Latest]: ./docs/3.2.5.md
[3.2.5]: ./docs/3.2.5.md
[3.2.1]: ./docs/3.2.1.md
[3.1.0]: ./docs/3.1.0.md
[3.0.0]: ./docs/3.0.0.md
[最新]: ./docs/3.2.5_cn.md
[3.2.5_cn]: ./docs/3.2.5_cn.md
[3.2.1_cn]: ./docs/3.2.1_cn.md
[3.1.0_cn]: ./docs/3.1.0_cn.md
[3.0.0_cn]: ./docs/3.0.0_cn.md
[Legacy Hibernate Criteria Queries]: https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#appendix-legacy-criteria
[EqualTest.java]: ./src/test/java/com/github/wenhao/jpa/integration/EqualTest.java
[NotEqualTest.java]: ./src/test/java/com/github/wenhao/jpa/integration/NotEqualTest.java
[InTest.java]: ./src/test/java/com/github/wenhao/jpa/integration/InTest.java
[GreatThanTest.java]: ./src/test/java/com/github/wenhao/jpa/integration/GreatThanTest.java
[BetweenTest.java]: ./src/test/java/com/github/wenhao/jpa/integration/BetweenTest.java
[LikeTest.java]: ./src/test/java/com/github/wenhao/jpa/integration/LikeTest.java
[NotLikeTest.java]: ./src/test/java/com/github/wenhao/jpa/integration/NotLikeTest.java
[OrTest.java]: ./src/test/java/com/github/wenhao/jpa/integration/OrTest.java
[AndOrTest.java]: ./src/test/java/com/github/wenhao/jpa/integration/AndOrTest.java
[PredicateTest.java]: ./src/test/java/com/github/wenhao/jpa/integration/PredicateTest.java
[JoinTest.java]: ./src/test/java/com/github/wenhao/jpa/integration/JoinTest.java
[SortTest.java]: ./src/test/java/com/github/wenhao/jpa/integration/SortsTest.java
[VirtualViewTest.java]: ./src/test/java/com/github/wenhao/jpa/integration/VirtualViewTest.java
[中文]: ./README_CN.md
[Apache License]: ./LICENSE


[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fwenhao%2Fjpa-spec.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fwenhao%2Fjpa-spec?ref=badge_large)