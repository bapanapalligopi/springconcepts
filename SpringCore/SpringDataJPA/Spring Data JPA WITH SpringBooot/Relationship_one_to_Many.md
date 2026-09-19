## 8. `@OneToMany` — One-to-Many Relationship

Let's understand this **from the database perspective first**, then Java/JPA.

### 1. What does One-to-Many mean?

It means:

> **One record in Table A can be associated with many records in Table B.**

A common example:

```text
Customer
   |
   | has many
   ↓
Orders
```

One customer can place multiple orders.

For example:

```text
Customer
----------------
id | name
----------------
1  | Rahul
2  | Amit
```

```text
Order
-------------------------
id | amount | customer_id
-------------------------
101| 500    | 1
102| 800    | 1
103| 300    | 1
104| 900    | 2
```

Here:

```text
Rahul (customer 1)
    ↓
Order 101
Order 102
Order 103

Amit (customer 2)
    ↓
Order 104
```

So:

```text
Customer 1 ─────────── * Orders
```

That's **One-to-Many**.

---

# 2. How do we represent this in Java?

We have two entities.

### Customer

```java
@Entity
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany
    private List<Order> orders;
}
```

### Order

```java
@Entity
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private BigDecimal amount;
}
```

Conceptually:

```text
Customer
   |
   | @OneToMany
   ↓
List<Order>
```

The important part is:

```java
@OneToMany
private List<Order> orders;
```

It tells JPA:

> "One Customer can be associated with multiple Orders."

---

# 3. But where is the relationship stored in the database?

This is where things get interesting.

The database doesn't have a Java `List`.

Instead, the relationship is usually represented using a **foreign key**.

```text
Customer
----------------
id | name
----------------
1  | Rahul
2  | Amit
```

```text
Order
-------------------------
id | amount | customer_id
-------------------------
101| 500    | 1
102| 800    | 1
103| 300    | 1
104| 900    | 2
```

The important column is:

```text
customer_id
```

That's a **foreign key** pointing to:

```text
customer.id
```

So:

```text
Order.customer_id → Customer.id
```

---

# 4. Why is `@ManyToOne` usually involved?

This is one of the most important things to understand.

We said:

```text
Customer 1 ─────── * Order
```

From the **Customer's perspective**:

```text
One Customer → Many Orders
```

Therefore:

```java
@OneToMany
```

From the **Order's perspective**:

```text
Many Orders → One Customer
```

Therefore:

```java
@ManyToOne
```

So in a real application, you will often have **both**:

```java
Customer
   ↓
@OneToMany
List<Order>

Order
   ↓
@ManyToOne
Customer
```

This is called a **bidirectional relationship**.

---

# 5. The proper mapping

### Customer

```java
@Entity
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "customer")
    private List<Order> orders = new ArrayList<>();
}
```

### Order

```java
@Entity
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private BigDecimal amount;

    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;
}
```

Now look carefully at:

```java
@OneToMany(mappedBy = "customer")
```

and:

```java
@ManyToOne
@JoinColumn(name = "customer_id")
```

These are extremely important.

---

# 6. What does `mappedBy = "customer"` mean?

This confuses almost everyone initially.

```java
@OneToMany(mappedBy = "customer")
private List<Order> orders;
```

`mappedBy = "customer"` means:

> "The `customer` field inside the `Order` entity owns the relationship."

Look at Order:

```java
@ManyToOne
@JoinColumn(name = "customer_id")
private Customer customer;
```

The field is:

```text
customer
```

Therefore:

```java
mappedBy = "customer"
```

So you can think:

```text
Customer
@OneToMany(mappedBy = "customer")
        ↓
Look inside Order
        ↓
Order.customer
```

---

# 7. What does `@JoinColumn` do?

Here:

```java
@ManyToOne
@JoinColumn(name = "customer_id")
private Customer customer;
```

we're telling JPA:

> "The foreign key column for this relationship is `customer_id`."

Database:

```text
orders
--------------------------------
id | amount | customer_id
--------------------------------
101| 500    | 1
102| 800    | 1
```

Java:

```text
Order.customer
      ↓
Customer
```

Database:

```text
Order.customer_id
      ↓
Customer.id
```

So:

```text
@ManyToOne
@JoinColumn(name = "customer_id")
```

is basically describing:

```text
orders.customer_id → customers.id
```

---

# 8. Which side owns the relationship?

This is **very important**.

In this example:

```java
Customer
@OneToMany(mappedBy = "customer")
List<Order> orders;
```

and:

```java
Order
@ManyToOne
@JoinColumn(name = "customer_id")
Customer customer;
```

The **Order side owns the relationship**.

Why?

Because `Order` contains the foreign key:

```text
orders.customer_id
```

So:

```text
Order → owning side
Customer → inverse side
```

A good rule:

> The side containing the foreign key is generally the owning side.

---

# 9. Why doesn't `Customer` own it?

Because the database relationship is actually stored here:

```text
orders.customer_id
```

not:

```text
customers.order_id
```

One customer can have many orders.

If you put `order_id` in `customers`, you'd only be able to naturally point to one order, which doesn't represent the relationship correctly.

Instead:

```text
orders
----------------
order_id
customer_id
```

Each order points to its customer.

---

# 10. What does `mappedBy` actually prevent?

Suppose you write:

```java
@OneToMany
private List<Order> orders;
```

and also:

```java
@ManyToOne
private Customer customer;
```

without `mappedBy`.

JPA may treat them as separate relationship mappings and potentially create an unwanted **join table** or additional mapping.

But:

```java
@OneToMany(mappedBy = "customer")
```

says:

> "Don't create another relationship mapping here. The `Order.customer` field already manages it."

So:

```text
Customer.orders
       ↓
mappedBy = "customer"
       ↓
Order.customer manages the relationship
```

---

# 11. How do we create an order?

Suppose:

```java
Customer customer = customerRepository.findById(1L)
        .orElseThrow();

Order order = new Order();
order.setAmount(new BigDecimal("500"));
order.setCustomer(customer);

orderRepository.save(order);
```

The important line is:

```java
order.setCustomer(customer);
```

This establishes:

```text
Order → Customer
```

Hibernate can then insert something like:

```sql
INSERT INTO orders (amount, customer_id)
VALUES (500, 1);
```

Notice:

```text
customer_id = 1
```

That's how the relationship is actually stored.

---

# 12. What about `customer.getOrders()`?

Because we have:

```java
@OneToMany(mappedBy = "customer")
private List<Order> orders;
```

we can conceptually do:

```java
Customer customer = customerRepository.findById(1L)
        .orElseThrow();

List<Order> orders = customer.getOrders();
```

Hibernate can fetch the orders belonging to that customer.

Something conceptually like:

```sql
SELECT *
FROM orders
WHERE customer_id = 1;
```

---

# 13. Very important: `LAZY`

By default, you generally want collections such as `@OneToMany` to be loaded lazily.

You can explicitly write:

```java
@OneToMany(
    mappedBy = "customer",
    fetch = FetchType.LAZY
)
private List<Order> orders;
```

This means:

```text
Load Customer
     ↓
Customer loaded
     ↓
Orders NOT immediately loaded
```

Then when you access:

```java
customer.getOrders();
```

Hibernate may execute another query.

This is useful because imagine a customer has:

```text
50,000 orders
```

You usually don't want every order loaded every time you retrieve the customer.

---

# 14. One common mistake

Beginners sometimes think this:

```java
@OneToMany
private List<Order> orders;
```

means the database has something like:

```text
Customer
------------------------
id | order1 | order2 ...
```

That's **not how a relational database works**.

Instead, the relationship is usually:

```text
customers
------------
id

orders
----------------
id
customer_id
```

The foreign key is on the **many side**.

Remember:

> **The "many" side usually contains the foreign key.**

So:

```text
Customer 1 ───────── * Order
                         ↑
                    foreign key
```

---

# 15. The most important mental model

Don't memorize annotations separately.

Think about the database first:

```text
customers
    |
    | 1
    |
    |--------- *
              |
            orders
```

Then:

```text
Customer
@OneToMany
List<Order>
```

and:

```text
Order
@ManyToOne
Customer
```

and finally:

```text
orders.customer_id
        ↓
customers.id
```

So the complete picture is:

```text
                    Java
                     │
        ┌────────────┴────────────┐
        ↓                         ↓
    Customer                    Order
        │                         │
 @OneToMany                  @ManyToOne
        │                         │
 List<Order>              Customer customer
        │                         │
        └──── mappedBy ───────────┘
                                  │
                             @JoinColumn
                                  │
                                  ↓
                           customer_id
                                  │
                                  ↓
                              Database
```

### For your 1.5-year level, make sure you can explain these 5 things without memorizing:

1. **What does `@OneToMany` mean?**
2. **Why is `@ManyToOne` normally present on the other side?**
3. **What does `mappedBy` mean?**
4. **What does `@JoinColumn` mean?**
5. **Which side owns the relationship and why?**

Once these are clear, the next concept—**`@ManyToOne`**—becomes much easier because you've already seen it as the other side of this relationship.
