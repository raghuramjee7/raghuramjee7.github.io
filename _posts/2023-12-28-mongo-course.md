---
layout: post
title: "A MongoDB Crash Course"
categories: junk
author:
- Raghuramjee Janapareddy
meta: "Springfield"
---

# NoSQL Databases

**Resource** - [YouTube Video](https://www.youtube.com/watch?v=c2M-rlkkT5o&t=43s&ab_channel=BroCode)

It allows different types of data to be stored in the same database. It is a non-relational database. It is a database that does not require a fixed schema, avoids joins, and is easy to scale. The type of data that can be stored is
1. Key-value pairs
2. Tabular data
3. Document data
4. Graph data

These are non relational, scalable and distributed databases. The support high transaction throughput. They are also partition tolerant and highly available.

# MongoDB
1. This is a document type db
2. Each document can be considered a row in the table
3. Data in each document is stored as key-value pairs. This is in BSON
4. A collection is a group of documents
5. A database is a group of collections

## Commands
1. To show all databases in the server, type `show dbs`
2. To open a database, type `use <db_name>`, if a db with the name does not exist, it will be created
3. The db wont be shown up initially in `show dbs` until a document is inserted into it
4. To create a collection in the db, type `db.createCollection("<collection_name>")`
5. To insert a document into a collection, type `db.<collection_name>.insert({<key>:<value>})`
6. To drop a database, type `db.dropDatabase()`. This will drop the database that you are currently in
7. To drop a collection, type `db.<collection_name>.drop()`
8. To insert a document into a collection, type `db.<collection_name>.insert({<key>:<value>})`. We can use insertOne() or insertMany() to insert one or many documents. If a collection does not exist, it will be created. For insertMany(), we need to pass an array of documents.
9. To get all the documents in a collection, type `db.<collection_name>.find()`

## Data Types
1. String - both double and single quotes can be used
2. Integer 
3. Boolean - true or false
4. Double
5. Date - `new Date()` will return the current date. If no arg, then it returns current date and time in UTC or we could pass an arg
6. null - null value
7. Array - `["a","b","c"]`
8. Nested Documents - `{name:"John", address:{city:"New York", state:"NY"}}`

**Let us have an imaginary student collection for simplicity**

## Sort and Limit
1. Return all documents - `db.<collection_name>.find()`
2. Return all documents with name John and age 25 - `db.<collection_name>.find({name:"John", age:25})`
3. Return all documents in sorted order - `db.<collection_name>.find().sort({name:1, gpa:-1})`. 1 for ascending and -1 for descending
4. We can limit the number of documents returned - `db.<collection_name>.find().limit(5)`
5. We can chain sort() and limit() - `db.<collection_name>.find().sort({name:1, gpa:-1}).limit(5)`

## Find
1. The parameters of find() can be used to filter the documents. 
2. We can get the documents with name John and age 25 - `db.<collection_name>.find({name:"John", age:25})`
3. We can use logical operators to filter the documents. We can use $or, $and, $not, $nor. Eg: `db.<collection_name>.find({$or:[{name:"John"},{age:25}]})`
4. There is a second parameter to find(), the first is query parameter, the second is projection parameter. 
5. If we specify the projection parameter, then only the fields specified will be returned. Eg - `db.students.find({}, {name: true})`

## Update
1. To update a single document, we use updateOne(). Eg - `db.students.updateOne({name:"John"},{$set:{age:26}})`
2. There are two parameters to updateOne(), the first is query parameter, the second is update parameter. The first is filter, the other is to update.
3. We can unset a field - `db.students.updateOne({name:"John"},{$unset:{age:1}})`. This helps in removing a field from a document.
4. We can update multiple documents - `db.students.updateMany({name:"John"},{$set:{age:26}})`. If we give the filter as empty, then all documents will be updated.
5. We can replace a document - `db.students.replaceOne({name:"John"},{name:"John", age:26})`. This will replace the document with the new document. The _id will be the same.
6. We can also insert a field in a document if it doesnt exist - `db.students.updateMany({age:{$exists: false}},{$set:{age:26}})`. This will insert a new field gpa in the document.

## Delete
1. To delete a single document, we use deleteOne(). Eg - `db.students.deleteOne({name:"John"})`
2. To delete multiple documents, we use deleteMany(). Eg - `db.students.deleteMany({name:"John"})`

## Comparison Operators
1. We can add operators to our search that will help us filter the documents. Eg - `db.students.find({age:{$gt:20}})`. This will return all documents with age greater than 20. The operators are $gt, $gte, $lt, $lte, $eq, $ne
2. We can use $in operator to search for documents with a field value in a list of values. Eg - `db.students.find({age:{$in:[20,25]}})`. This will return all documents with age 20 or 25.
3. We can use $nin operator to search for documents with a field value not in a list of values. Eg - `db.students.find({age:{$nin:[20,25]}})`. This will return all documents with age not 20 or 25.
4. We can combine multiple operators - `db.students.find({age:{$gt:20, $lt:25}})`. This will return all sthde ts with age greater than 20 and less than 25.

## Indexes
1. Indexes are used to improve the performance of queries.
2. To create an index, we use createIndex(). Eg - `db.students.createIndex({name:1})`. This will create an index on the name field in ascending order.
3. To get all indexes, we use getIndexes(). Eg - `db.students.getIndexes()`
4. We can drop an index using dropIndex(). Eg - `db.students.dropIndex("<index_name>")`