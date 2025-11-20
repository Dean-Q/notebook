---
description: Specification is a dynamic query tool as Domain-Driven Design pattern
---

# Specification

impIn the past, we write SQL by @Query, well, @Query is a good static way to query DB, but now we have Specification

some background, we will use this repo as a example to demo how to use specification

<pre><code>@Entity
class User {
    @Id Long id;
    String name;
    Integer age;

    @ManyToOne(fetch = FetchType.LAZY)
    Department department;
}

@Entity
class Department {
    @Id Long id;
    String name;
}


public interface UserRepository 
        extends JpaRepository&#x3C;User, Long>, <a data-footnote-ref href="#user-content-fn-1">JpaSpecificationExecutor&#x3C;User> </a>{}
</code></pre>

### Single field query

```
public static Specification<User> nameEquals(String name) {
    return (root, query, cb) ->
            cb.equal(root.get("name"), name);
}

//how to call in another class
var users = userRepository.findAll(nameEquals("Alice"));

//compose of sql will be
select u.*
from user u
where u.name = ?


```

### Multi And query

```
//use record to simply DTO
public record UserFilter(String name, Integer minAge, Integer maxAge) {}

//specification
public static Specification<User> buildSpec(UserFilter filter) {
    return (root, query, cb) -> {
        var predicates = new ArrayList<Predicate>();

        if (filter.name() != null) {
            predicates.add(cb.like(root.get("name"), "%" + filter.name() + "%"));
        }

        if (filter.minAge() != null) {
            predicates.add(cb.greaterThanOrEqualTo(root.get("age"), filter.minAge()));
        }

        if (filter.maxAge() != null) {
            predicates.add(cb.lessThanOrEqualTo(root.get("age"), filter.maxAge()));
        }

        return cb.and(predicates.toArray(Predicate[]::new));
    };
}
//how to call 
var filter = new UserFilter("A", 18, 30);
var users = userRepository.findAll(buildSpec(filter));

//compose of sql will be
select u.*
from user u
where u.name like ?
  and u.age >= ?
  and u.age <= ?


```

### Joint table query

```
public static Specification<User> inDepartment(String deptName) {
    return (root, query, cb) -> {
        var dept = root.join("department", JoinType.LEFT);
        return cb.equal(dept.get("name"), deptName);
    };
}

//how to call
var result = userRepository.findAll(inDepartment("IT"));

//sql will be
select u.*
from user u
left join department d on u.department_id = d.id
where d.name = ?

```

### Count aggregate query

```
public static Specification<User> countByAge(Integer age) {
    return (root, query, cb) -> {
        query.select(cb.count(root)); // 切换返回类型
        return cb.equal(root.get("age"), age);
    };
}

//how to call
var result = userRepository.findBy(countByAge(18), Long.class);

//sql will be
select count(u.id)
from user u
where u.age = ?

```

### Sum aggregate query

```
public static Specification<User> sumAgeInDepartment(String deptName) {
    return (root, query, cb) -> {
        var dept = root.join("department");
        query.select(cb.sum(root.get("age")));
        return cb.equal(dept.get("name"), deptName);
    };
}


//how to call
var totalAge = userRepository.findBy(sumAgeInDepartment("IT"), Integer.class);

//sql
select sum(u.age)
from user u
join department d on u.department_id = d.id
where d.name = ?

```

### Dynamic Condition and pagination

```
public record UserQuery(
        String nameContains,
        Integer minAge,
        String deptName
) {}

public static Specification<User> buildComplex(UserQuery q) {
    return (root, query, cb) -> {
        List<Predicate> list = new ArrayList<>();

        if (q.nameContains() != null) {
            list.add(cb.like(root.get("name"), "%" + q.nameContains() + "%"));
        }

        if (q.minAge() != null) {
            list.add(cb.greaterThanOrEqualTo(root.get("age"), q.minAge()));
        }

        if (q.deptName() != null) {
            var dept = root.join("department");
            list.add(cb.equal(dept.get("name"), q.deptName()));
        }

        return cb.and(list.toArray(Predicate[]::new));
    };
}

//how to call
var q = new UserQuery("A", 20, "IT");

var page = userRepository.findAll(
    buildComplex(q),
    PageRequest.of(0, 10, Sort.by("age").descending())
);


//sql will be
select u.*
from user u
join department d on u.department_id = d.id
where u.name like ?
  and u.age >= ?
  and d.name = ?
order by u.age desc
limit ? offset ?

```

### Conjunction（1=1）

```
public static Specification<User> ageAndDept(Integer minAge, String deptName) {
    return (root, query, cb) -> {
        var dept = root.join("department", JoinType.LEFT);

        // 使用 cb.conjunction() 创建 AND 条件容器
        Predicate conjunction = cb.conjunction(); 
        conjunction.getExpressions().add(cb.greaterThanOrEqualTo(root.get("age"), minAge));
        conjunction.getExpressions().add(cb.equal(dept.get("name"), deptName));

        return conjunction;
    };
}

//how to call
var users = userRepository.findAll(ageAndDept(18, "IT"));

//sql
select u.*
from user u
left join department d on u.department_id = d.id
where u.age >= ?
  and d.name = ?


```

### Disjunction (1=0)

```
public static Specification<User> nameOrAge(String name, Integer age) {
    return (root, query, cb) -> {
        // 使用 cb.disjunction() 创建 OR 条件容器
        Predicate disjunction = cb.disjunction();
        disjunction.getExpressions().add(cb.equal(root.get("name"), name));
        disjunction.getExpressions().add(cb.lessThanOrEqualTo(root.get("age"), age));
        return disjunction;
    };
}


//how to call
var users = userRepository.findAll(nameOrAge("Alice", 20));

//sql will be
select u.*
from user u
where u.name = ?
   or u.age <= ?

```



[^1]: important class
