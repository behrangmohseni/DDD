#### How to implement Aggregate Roots and Value Objects

Implementing Aggregate Roots and Value Objects is an important part of implmemnting the domain object. Be aware that in this context we are not talking about designing aggregates and other domain entities. that is another topic that we need to focus in some other part of this series. The concern of this article is that how we can implement the base class and interfaces for Aggregate Roots and Value objects. Now lets start with value objects. 

##### implementing ValueObject base class
Let see what are our constrains using value objects. 
- value objects do not have identifiers. It means that the identity of value object is combination of it's values not an exact identity property.
- comparing two value objects is to compare their type (obviously!) and their properties. It means that shape of the object (type) and the value of it fields and properties should be the same to be considered as equal.
- value objects are immutable, it means that we cannot change the value of it properties in any form. for changing the value of a value object we need to create a new one based on the attributes of the first with the changed attributes. 
some programming languages provide us with some utilities to handle value objects easier. C# 9.0 provided the record type that can handle some of the required logic implicitly. However using record has it's own drawbacks

```csharp
public class ValueObjectBase
{
    
}
```