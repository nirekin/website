---
title: "Entities"
type: "home"
zones:
    - "Business"
sections:
    - "BusinessManual"
tags:
    - domain-driven design
menu:
    BusinessManual:
        weight: 20
---

{{% callout def %}}
**An entity is used to represent a domain concept distinguished by an identity.<br>
This identity must remain the same through the whole entity lifecycle.**
{{% /callout %}}
<!--more-->

# Characteristics

Entities represent a thread of continuity and identity, going through a lifecycle, though their attributes may change.
They are not defined primarily by their attributes but by their identity that stays the same through time and across 
distinct representations.

## Identity

The identity of an entity must be unique and immutable. It must be chosen carefully and well defined in the model. 
Identification can come from:

* The outside: a user of the system can provide the identity, handling the uniqueness himself.
* The inside: the entity can generate its own identity using an algorithm.
* An identity generator, like a database sequence.

{{% callout tips %}}
It is often a good idea to use [value objects]({{< ref "docs/business/manual/value-objects.md" >}}) as identifiers, particularly
in the case of a composite identity. Its consistency and immutability can then be delegated to the value object.
{{% /callout %}}

## Behavior

Entities should not be merely holders of attributes, but should also contain the behavior that is directly relevant to
them. Do not create entities with only getters and setters but add methods with meaningful names, implementing domain behavior. 

{{% callout tips %}}
When the behavior does not fit naturally into a specific entity, for instance if multiple entities are involved, you move
the behavior to [domain services]({{< ref "docs/business/manual/services.md" >}}).
{{% /callout %}}

# Declaration

To declare an entity with the business framework, you have two alternatives. 

{{% tabs list="Basic|Interface" %}}
{{% tab "Basic" true %}}
Extend the {{< java "org.seedstack.business.domain.BaseEntity" >}} class:

```java
public class SomeEntity extends BaseEntity<SomeEntityId> {
    private SomeEntityId id;

    public SomeEntity(SomeEntityId id) {
        this.id = id;
    }

    @Override
    public SomeEntityId getEntityId() {
        return this.id;
    }
    
    // Other methods
}
```

By extending {{< java "org.seedstack.business.domain.BaseEntity" >}}, you will have a default implementation of the
`equals()` and `hashCode()` methods, consistent with the definition of an entity. A `toString()` is also provided by default.
{{% /tab %}}
{{% tab "Interface" %}}
Implement the {{< java "org.seedstack.business.domain.Entity" >}} interface:

```java
public class SomeEntity implements Entity<SomeEntityId> {
    private SomeEntityId id;

    public SomeEntity(SomeEntityId id) {
        this.id = id;
    }
    
    public int hashCode() {
        // TODO: implement using identity attribute only
    }

    public boolean equals() {
        // TODO: implement using identity attribute only
    }

    @Override
    public SomeEntityId getEntityId() {
        return this.id;
    }
    
    // Other methods
}
```

Implementing {{< java "org.seedstack.business.domain.Entity" >}} allows you to fully control the inheritance of your
entity. However, you will have to implement `equals()` and `hashCode()` methods yourself, consistently with the definition 
of an entity (i.e. based on the identity only). 
{{% /tab %}}
{{% /tabs %}}

You must implement the `getEntityId()` method as the framework will often need to retrieve the entity identity.

# Example

Notice:

* How the identity is defined as an immutable [value object]({{< ref "docs/business/manual/value-objects.md" >}}) and cannot 
be changed by external objects.
* How method names have meaningful names and implement domain behavior.

```java
public class Customer extends BaseEntity<CustomerId> {
    private final CustomerId id;
    private String name;
    private String email;
    
    public Customer(CustomerId id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }
    
    @Override
    public CustomerId getEntityId() {
        return this.id;
    }
    
    public void changeEmail(String newEmail) {
        if (!isEmailValid(newEmail)) {
            throw new CustomerException("Invalid email: " + newEmail);
        }
        email = newEmail;
    }
    
    public void changeName(String newName) {
        if (newName.isEmpty()) {
            throw new CustomerException("Name cannot be blank");
        }
        name = newName;
    }
    
    // other methods 
}
```
