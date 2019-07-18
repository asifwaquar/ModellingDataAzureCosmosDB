# Modelling Data and Best Practices for Azure COSMOSDB

### What is good partition key in COSMOS DB?

A good partition key ensures well balanced partition in both terms 

* Storage 
Data should be distributed uniformly accross all logical pk's.

* Throughput
Workloads uniformly distributed accross all logical pk's.

### Model your data by referencing & embedding.

