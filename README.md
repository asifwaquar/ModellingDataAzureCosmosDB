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

1 : many (unbounded relationship)

{
    "id": "1",
    "name": "Alice",
    "email": "alice@contoso.com",
    "Orders": [
        {
            "id": "Order1", 
            "orderDate": "2018-09-18",
                "itemsOrdered": [
                    {"ID": 1, "ItemName": "hamburger", "Price":9.50, "Qty": 1}
                    {"ID": 2, "ItemName": "cheeseburger", "Price":9.50, "Qty": 499}]
                    }, 
                    ...
                    {
                    "id": "OrderNfinity", 
                    "orderDate": "2018-09-20",
                    "itemsOrdered": [
                    {"ID": 1, "ItemName": "hamburger", "Price":9.50, "Qty": 1}]
            }]
}





