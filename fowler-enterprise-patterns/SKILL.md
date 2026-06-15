---
name: fowler-enterprise-patterns
description: Use when designing data access patterns — choosing between Active Record, Data Mapper, Repository, Table Module, and Service Layer; understanding transaction scripts vs domain models; mapping objects to databases; or reviewing code where persistence and domain logic are tangled. From Fowler's Patterns of Enterprise Application Architecture (PoEAA).
---

# Fowler — Enterprise Application Patterns (PoEAA)

## The Core Question PoEAA Answers

How do you connect objects (which have behavior and references) to databases (which have tables and foreign keys)? This impedance mismatch is unavoidable; the patterns are different ways of managing it.

> "The central question is how much you want your domain objects to know about the database." — Fowler, PoEAA

The answer shapes everything downstream: how testable the code is, how the domain evolves, and how the persistence layer can change.

## The Spectrum from Simple to Complex

```
Transaction Script ──── Table Module ──── Active Record ──── Domain Model + Data Mapper
   (simplest)                                                        (most powerful)
   database-first                                                    domain-first
   logic in procedures                                               logic in objects
   good for simple CRUD                                              good for complex domains
```

### Transaction Script

Organize logic as a set of procedures, each handling one request. Each procedure directly queries the database. No domain objects — just inputs → database → output.

```python
def place_order(customer_id, items):
    total = sum(item.price * item.qty for item in items)
    customer = db.query("SELECT credit_limit FROM customers WHERE id=?", customer_id)
    if total > customer.credit_limit:
        raise ValueError("Credit limit exceeded")
    order_id = db.execute("INSERT INTO orders ...")
    for item in items: db.execute("INSERT INTO order_lines ...")
    return order_id
```

**When to use:** simple applications with few business rules, CRUD-heavy systems, scripts that will be replaced. **Problem:** doesn't scale as business logic grows — rules duplicate across procedures, logic becomes unsearchable.

### Active Record

An object that wraps a database row: knows how to load, save, delete itself. Rails popularized this; most ORMs default to it.

```python
class Order(ActiveRecord):
    def place(self):
        if self.total > self.customer.credit_limit:
            raise DomainError("Credit limit exceeded")
        self.status = "PLACED"
        self.save()   # knows how to persist itself
```

**When to use:** moderate complexity; when the domain and database shape are closely aligned; when team moves fast with convention over configuration. **Problem:** domain object knows about the database; changing the schema means changing the domain; hard to test without the DB.

### Data Mapper

A separate mapper handles all communication between domain objects and the database. Domain objects know nothing about persistence.

```python
class Order:
    # No save(), no load() — pure domain logic
    def place(self, customer):
        if self.total > customer.available_credit():
            raise DomainError("Credit limit exceeded")
        self.status = "PLACED"
        self.record(OrderPlaced(self.id))

class OrderMapper:
    def find(self, id) -> Order: ...        # DB → Order
    def save(self, order: Order): ...       # Order → DB
    def find_by_customer(self, cid): ...
```

**When to use:** complex domains where the object model and database schema differ; when testability matters (domain tests need no DB); when domain and database evolve independently. **Problem:** more upfront code; a separate mapper class for each domain object.

### Repository (on top of Data Mapper)

The repository is the application-facing interface over a Data Mapper. It presents a collection-like illusion:

```python
# The interface the application uses (collection-like, domain vocabulary)
class OrderRepository:
    def find_by_id(self, id: OrderId) -> Optional[Order]: ...
    def find_pending_for_customer(self, cid: CustomerId) -> List[Order]: ...
    def save(self, order: Order): ...

# The implementation that knows about the DB
class SQLOrderRepository(OrderRepository):
    def find_by_id(self, id): ...   # SQL + OrderMapper.to_domain()
```

This is the pattern in clean-architecture and persistence-repository — the domain defines the interface, infrastructure implements it. Full testability via an in-memory implementation.

### Table Module

One object per database table, handling all rows of that table. Somewhere between Transaction Script and Domain Model.

```python
class Orders:           # one instance handles all orders
    def place(self, customer_id, items): ...
    def cancel(self, order_id): ...
    def find_by_customer(self, customer_id): ...
```

**When to use:** when you need some organization beyond Transaction Script but don't want full domain object complexity. Suits simple tabular data well. **Problem:** harder to represent complex object graphs.

## Service Layer — Defining the Application Boundary

The Service Layer sits above domain objects and below the API layer. It defines the application's use cases:

```python
class OrderService:
    def place_order(self, cmd: PlaceOrderCommand) -> Order:
        # Coordinates domain objects + repositories
        customer = self.customers.find(cmd.customer_id)
        order = Order.place(customer, cmd.items)
        self.orders.save(order)
        self.events.publish(OrderPlaced(order.id))
        return order
```

The Service Layer:
- Has no domain logic (that's in domain objects)
- Has no UI or HTTP concerns (that's in the controller)
- Defines the transaction boundary
- Maps to use-case concepts, not database operations

**Thin vs Rich Service Layer:** Thin = domain objects do the work, service just orchestrates. Rich = service contains logic that should be in domain objects (anemic domain model). Prefer thin.

## Choosing the Pattern

```
Simple CRUD, low complexity     → Transaction Script or Active Record
Moderate complexity, fast team  → Active Record (Rails/Django style)
Complex domain, testability     → Domain Model + Data Mapper + Repository
Reporting-heavy, tabular data   → Table Module
```

The patterns coexist in one codebase. The core business domain uses Data Mapper + Repository for testability. Reporting uses SQL directly. Configuration uses Active Record for simplicity. Choosing a single pattern for the whole application is over-constraint.

**REQUIRED COMPANION:** persistence-repository covers the Repository pattern in depth with the dependency inversion principle. domain-model covers the domain objects that the Data Mapper serves.
