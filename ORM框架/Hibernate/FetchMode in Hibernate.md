# FetchMode in Hibernate

## 1. Introduction

In this short tutorial, we’ll take a look at different *FetchMode* values we can use in the *@**org.hibernate.annotations.Fetch* annotation.

## 2. Setting up the Example

As an example, we'll use the following *Customer* entity with just two properties – an id and a set of orders:

```java
@Entity
public class Customer {

    @Id
    @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "customer")
    @Fetch(value = FetchMode.SELECT)
    private Set<Order> orders = new HashSet<>();

    // getters and setters
}
```

Also, we'll create an *Order* entity consisting of an id, a name and a reference to the *Customer*.

```java
@Entity
public class Order {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;

    // getters and setters
}
```

In each of the next sections, we'll fetch the customer from the database and get all of its orders:

```java
Customer customer = customerRepository.findById(id).get();
Set<Order> orders = customer.getOrders();
```



## 3. *FetchMode.SELECT*

On our *Customer* entity, we've annotated the *orders* property with a *@Fetch* annotation:

```java
@OneToMany
@Fetch(FetchMode.SELECT)
private Set<Orders> orders;
```

We use *@Fetch* to describe how Hibernate should retrieve the property when we lookup a *Customer.*

Using *SELECT* indicates that the property should be loaded lazily.

This means that for the first line:

```java
Customer customer = customerRepository.findById(id).get();
```

We won't see a join with the orders table:

```sql
Hibernate: 
    select ...from customer
    where customer0_.id=?

```

And that for the next line:

```java
Set<Order> orders = customer.getOrders();
```

We'll see subsequent queries for the related orders:

```java
Hibernate: 
    select ...from order
    where order0_.customer_id=?
```

The *Hibernate FetchMode.SELECT* generates a separate query for each *Order* that needs to be loaded.

In our example, that gives one query to load the Customers and five additional queries to load the orders collection.

**This is known as the n + 1 select problem.** Executing one query will trigger *n* additional queries.



### 3.1. @*BatchSize*

*FetchMode.SELECT* has an optional configuration annotation using the *@BatchSize* annotation:

```java
@OneToMany
@Fetch(FetchMode.SELECT)
@BatchSize(size=10)
private Set<Orders> orders;
```

*Hibernate* will try to load the orders collection in batches defined by the *size* parameter.

In our example, we have just five orders so one query is enough.

We'll still use the same query:

```java
Hibernate:
    select ...from order
    where order0_.customer_id=?
```

**But it will only be run once.** Now we have just two queries: One to load the *Customer* and one to load the orders collection.



## 4. *FetchMode.JOIN*

While *FetchMode.SELECT* loads relations lazily, *FetchMode.JOIN* loads them eagerly, say via a join:

```java
@OneToMany
@Fetch(FetchMode.JOIN)
private Set<Orders> orders;
```

This results in just one query for both the *Customer* and their *Order*s:

```sql
Hibernate: 
    select ...
    from
        customer customer0_ 
    left outer join
        order order1 
            on customer.id=order.customer_id 
    where
        customer.id=?
```



## 5. *FetchMode.SUBSELECT*

Because the *orders* property is a collection, we could also use *FetchMode.SUBSELECT*:

```java
@OneToMany
@Fetch(FetchMode.SUBSELECT)
private Set<Orders> orders;
```

We can only use *SUBSELECT* with collections.

With this setup, we go back to one query for the *Customer:*

```sql
Hibernate: 
    select ...
    from customer customer0_

```

And one query for the *Order*s, using a sub-select this time:

```sql
Hibernate: 
    select ...
    from
        order order0_ 
    where
        order0_.customer_id in (
            select
                customer0_.id 
            from
                customer customer0_
        )
```



## 6. *FetchMode* vs. *FetchType*

In general, *FetchMode* defines how *Hibernate* will fetch the data (by select, join or subselect). *FetchType*, on the other hand, defines whether Hibernate will load data eagerly or lazily.

The exact rules between these two are as follows:

- if the code doesn't set *FetchMode*, the default one is *JOIN* and *FetchType* works as defined
- with *FetchMode.SELECT* or *FetchMode.SUBSELECT* set, *FetchType* also works as defined
- with *FetchMode.JOIN* set, *FetchType* is ignored and a query is always eager

For further information please refer to [Eager/Lazy Loading In Hibernate](https://www.baeldung.com/hibernate-lazy-eager-loading).



## 7. Conclusion

In this tutorial, we’ve learned about *FetchMode*‘s different values and also how they are related to *FetchType*.

As always all source code is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-mapping).