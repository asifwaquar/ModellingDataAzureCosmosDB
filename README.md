# Modelling Data and Best Practices for Azure COSMOSDB

### What is good partition key in COSMOS DB?

A good partition key ensures well balanced partition in both terms 

* Storage 

Data should be distributed uniformly accross all logical pk's.

* Throughput

Workloads uniformly distributed accross all logical pk's.

### Model your data by referencing & embedding.

#### When to Embed #1

* “Data that is queried together, should live together” 


{
  "ID": 1,
  "ItemName": "hamburger",
  "ItemDescription": "cheeseburger, no cheese",
  "Category": "sandwiches",
  "CategoryDescription": "2 pieces of bread + filling",
  "Ingredients": [
        {"ItemName": "bread", "calorieCount": 100, "Qty": "2 slices"},
        {"ItemName": "lettuce", "calorieCount": 10, "Qty": "1 slice"}
        {"ItemName": "tomato","calorieCount": 10, "Qty": "1 slice"}
        {"ItemName": "patty", "calorieCount": 700, "Qty": "1"}
        ]
}

* E.g. in Recipe, ingredients are always queried with the item 


#### When to Embed #2

* Child data is dependent on a parent 


{
    "id": "Order1", 
    "customer": "Customer1",
    "orderDate": "2018-09-26",
    "itemsOrdered": [
        {"ID": 1, "ItemName": "hamburger", "Price":9.50, "Qty": 1}
        {"ID": 2, "ItemName": "cheeseburger", "Price":9.50, "Qty": 499}
    ]
}

* Items Ordered depends on Order

#### When to Embed #3

* 1:1 relationship


    {
        "id": "1",
        "name": "Alice",
        "email": "alice@contoso.com",
        “phone": “555-5555"
        “loyaltyNumber": 13838359,
        "addresses": [
            {"street": "1 Contoso Way", "city": "Seattle"},
            {"street": "15 Fabrikam Lane", "city": "Orlando"}
        ]
   }
   
   
* All customers have email, phone, loyalty number for1:1 relationship
 
#### When to embed #4, #5

* Similar rate of updates – does the data change at the same pace ?

* 1:few relationships


{
    "id": "1",
    "name": "Alice",
    "email": "alice@contoso.com",
    "addresses": [
        {"street": "1 Contoso Way", "city": "Seattle"},
        {"street": "15 Fabrikam Lane", "city": "Orlando"}
    ]
}

* Usually embedding provides better read performance

#### When to reference #1

* 1 : many (unbounded relationship)

{
    "id": "1",
    "firstName": "Thomas",
    "lastName": "Andersen",
    "addresses": [
        {
            "line1": "100 Some Street",
            "line2": "Unit 1",
            "city": "Seattle",
            "state": "WA",
            "zip": 98012
        }
    ],
    "contactDetails": [
        {"email": "thomas@andersen.com"},
        {"phone": "+1 555 555-5555", "extension": 5555}
    ]
}


#### When to reference #2

* Data changes at different rates #2


Number of orders, amount spent will likely change faster than email, so reference these


#### When to reference #3

* many : many relationships
 
#### When to reference #4

* What is referenced, is heavily referenced by many others

### e.g for embed

Modelling data in relational database .

![Choose imge](https://docs.microsoft.com/en-us/azure/cosmos-db/media/sql-api-modeling-data/relational-data-model.png)

* query to to show details 

SELECT p.FirstName, p.LastName, a.City, cd.Detail
FROM Person p
JOIN ContactDetail cd ON cd.PersonId = p.Id
JOIN ContactDetailType cdt ON cdt.Id = cd.TypeId
JOIN Address a ON a.PersonId = p.Id

Modelling same data in cosmos db .

{
    "id": "1",
    "name": "What's new in the coolest Cloud",
    "summary": "A blog post by someone real famous",
    "comments": [
        {"id": 1, "author": "anon", "comment": "something useful, I'm sure"},
        {"id": 2, "author": "bob", "comment": "wisdom from the interwebs"},
        …
        {"id": 100001, "author": "jane", "comment": "and on we go ..."},
        …
        {"id": 1000000001, "author": "angry", "comment": "blah angry blah angry"},
        …
        {"id": ∞ + 1, "author": "bored", "comment": "oh man, will this ever end?"},
    ]
}



### e.g for Reference

Post item:
{
    "id": "1",
    "name": "What's new in the coolest Cloud",
    "summary": "A blog post by someone real famous",
    "recentComments": [
        {"id": 1, "author": "anon", "comment": "something useful, I'm sure"},
        {"id": 2, "author": "bob", "comment": "wisdom from the interwebs"},
        {"id": 3, "author": "jane", "comment": "....."}
    ]
}

Comment items:
{
    "postId": "1"
    "comments": [
        {"id": 4, "author": "anon", "comment": "more goodness"},
        {"id": 5, "author": "bob", "comment": "tails from the field"},
        ...
        {"id": 99, "author": "angry", "comment": "blah angry blah angry"}
    ]
},
{
    "postId": "1"
    "comments": [
        {"id": 100, "author": "anon", "comment": "yet more"},
        ...
        {"id": 199, "author": "bored", "comment": "will this ever end?"}
    ]
}


The problem with this example is that the comments array is unbounded, meaning that there is no (practical) limit to the number of comments any single post can have. This may become a problem as the size of the item could grow infinitely large.

As the size of the item grows the ability to transmit the data over the wire as well as reading and updating the item, at scale, will be impacted.

Post item:
{
        "id": "1",
        "name": "What's new in the coolest Cloud",
        "summary": "A blog post by someone real famous",
        "recentComments": [
            {"id": 1, "author": "anon", "comment": "something useful, I'm sure"},
            {"id": 2, "author": "bob", "comment": "wisdom from the interwebs"},
            {"id": 3, "author": "jane", "comment": "....."}
        ]
    }

    Comment items:
    {
        "postId": "1"
        "comments": [
            {"id": 4, "author": "anon", "comment": "more goodness"},
            {"id": 5, "author": "bob", "comment": "tails from the field"},
            ...
            {"id": 99, "author": "angry", "comment": "blah angry blah angry"}
        ]
    },
    {
        "postId": "1"
        "comments": [
            {"id": 100, "author": "anon", "comment": "yet more"},
            ...
            {"id": 199, "author": "bored", "comment": "will this ever end?"}
        ]
}



This model has the three most recent comments embedded in the post container, which is an array with a fixed set of attributes. The other comments are grouped in to batches of 100 comments and stored as separate items. The size of the batch was chosen as 100 because our fictitious application allows the user to load 100 comments at a time.

Another case where embedding data is not a good idea is when the embedded data is used often across items and will change frequently.


### What about foreign keys?

Because there is currently no concept of a constraint, foreign-key or otherwise, any inter-document relationships that you have in documents are effectively "weak links" and will not be verified by the database itself. If you want to ensure that the data a document is referring to actually exists, then you need to do this in your application, or through the use of server-side triggers or stored procedures on Azure Cosmos DB.
