---
name: martin-clean-arch
description: Use when designing the high-level structure of a system — applying Uncle Bob's Clean Architecture (the concentric rings), understanding the Dependency Rule, designing use cases as the architectural center, separating policy from mechanism, or when reviewing whether a codebase's architecture protects the business rules from framework/database/UI changes.
---

# Martin — Clean Architecture

## The Central Claim

> "The architecture of a software system is the shape given to that system by those who build it. The purpose of that shape is to facilitate the development, deployment, operation, and maintenance of the software system contained within it." — Robert C. Martin, Clean Architecture

Clean Architecture (2017) is Robert C. Martin's synthesis of hexagonal architecture (Alistair Cockburn), onion architecture (Jeffrey Palermo), and BCE (Ivar Jacobson). The core insight: **the web is a delivery mechanism. The database is a detail. The UI is a detail. The business rules are the system.**

Most architectures get this backwards: the database or framework is at the center, and business rules are expressed in terms of it.

## The Concentric Rings

```
          ┌────────────────────────────────────────────┐
          │  Frameworks & Drivers                       │
          │  (Web, DB, UI, External APIs)               │
          │  ┌──────────────────────────────────────┐  │
          │  │  Interface Adapters                   │  │
          │  │  (Controllers, Gateways, Presenters)  │  │
          │  │  ┌────────────────────────────────┐  │  │
          │  │  │  Application Business Rules     │  │  │
          │  │  │  (Use Cases / Interactors)      │  │  │
          │  │  │  ┌──────────────────────────┐  │  │  │
          │  │  │  │  Enterprise Business      │  │  │  │
          │  │  │  │  Rules (Entities)         │  │  │  │
          │  │  │  └──────────────────────────┘  │  │  │
          │  │  └────────────────────────────────┘  │  │
          │  └──────────────────────────────────────┘  │
          └────────────────────────────────────────────┘
                    Dependencies point INWARD
```

**The Dependency Rule:** source code dependencies must point inward. Nothing in an inner ring can know about something in an outer ring. The name of a framework, database, or UI element must not appear in the inner rings.

## The Four Rings

### 1. Entities (innermost)

Enterprise business rules. Objects or data structures that encapsulate the most general, high-level rules. They change least frequently — when they do, it's because the fundamental business policy changed.

```python
class Order:  # Entity — no imports from outer rings
    def place(self, customer, lines):
        if self._total(lines) > customer.credit_limit:
            raise CreditLimitExceeded()
        return Order(lines=lines, status=OrderStatus.PLACED)
```

### 2. Use Cases

Application-specific business rules. Orchestrate the flow of data to and from Entities. They encode the *application's* intent — what the system does. One use case per user operation.

```python
class PlaceOrderUseCase:  # Use Case — knows Entities, not frameworks
    def __init__(self, customer_repo: CustomerRepository, order_repo: OrderRepository):
        ...
    def execute(self, cmd: PlaceOrderCommand) -> OrderId:
        customer = self.customer_repo.find(cmd.customer_id)
        order = Order.place(customer, cmd.lines)
        self.order_repo.save(order)
        return order.id
```

The use case takes a **Request Model** (plain data structure) and returns a **Response Model** (plain data structure). No HTTP objects, no ORM entities, no framework types.

### 3. Interface Adapters

Convert data between the format convenient for use cases/entities and the format convenient for the outer ring (web, DB, UI).

- **Controllers**: convert HTTP request → Request Model, call use case, convert Response Model → HTTP response
- **Presenters**: convert Response Model → ViewModel for display
- **Gateways**: implement repository interfaces using the actual database/ORM

### 4. Frameworks & Drivers (outermost)

The messy details: web frameworks, databases, UI frameworks, external APIs. These change most frequently; that's why they're outermost. They implement the interfaces defined by the inner rings — they are plugins.

## Policy vs Mechanism — the Core Separation

**Policy**: the business rules — what the system should do. Stable. Valuable.  
**Mechanism**: how it does it — the web framework, the SQL dialect, the ORM. Volatile. Replaceable.

A well-structured system makes it trivial to swap mechanisms without touching policy. You should be able to replace MySQL with PostgreSQL, or REST with GraphQL, without touching your use cases or entities.

The test: **can you test your use cases without starting the web server or connecting to a database?** If yes, you've separated policy from mechanism. If no, the mechanism has leaked into the policy.

```python
# This test proves the architecture is clean:
def test_place_order_enforces_credit_limit():
    # No web server. No database. No framework.
    customer = Customer(credit_limit=Money(100))
    repo = InMemoryOrderRepository()  # test double, not real DB
    use_case = PlaceOrderUseCase(customer_repo=FakeCustomerRepo(customer), order_repo=repo)
    
    with pytest.raises(CreditLimitExceeded):
        use_case.execute(PlaceOrderCommand(customer_id=customer.id, total=Money(200)))
```

## The Humble Object Pattern

At every boundary between rings, introduce a "humble object" — a thin translator with no logic that converts data between representations. The humble object is too simple to have bugs; all the logic is in the inner rings where it's testable.

```
HTTP Request (outer)
   → Controller (humble object — translates HTTP to Request Model)
   → Use Case (the logic — fully testable)
   → Presenter (humble object — translates Response Model to ViewModel)
   → View (humble object — renders the ViewModel)
```

Testing the use case tests all the logic. The humble objects are so thin they're tested through integration tests only.

## What Clean Architecture Is NOT

- **Not a prescription for microservices.** A single monolith with clean rings is cleaner than distributed mud.
- **Not a requirement for a DDD domain model.** Transaction scripts can live in the Use Case ring for simple domains.
- **Not an excuse for over-engineering.** A CRUD app with 3 use cases doesn't need 6 layers. The architecture serves the software; the software doesn't serve the architecture.

The test of the architecture is always: **does this structure make the system easier to understand, change, and test?** If adding a layer makes it harder, don't add the layer.

**REQUIRED COMPANION:** martin-solid covers the SOLID principles that clean-architecture depends on. clean-architecture covers the four-layer implementation in detail. persistence-repository implements the Gateway/Repository pattern from this ring structure.
