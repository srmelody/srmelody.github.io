---
layout: post
title:  "Painters Algorithm in the Wild"
date:   2017-06-28 13:25:00 -0400
categories: groovy grails mongo debugging hacking
---

*Originally published 2015, Rally Engineering Blog*

# It Was a Dark and Stormy Night

When we hire the great engineers that we get to work with at CA, one of the key things we look for is how good they are at understanding algorithms. At the end of the day, algorithms and data structures are the fundamentals of computer science and software engineering. And, a few weeks ago, we had to flex our algorithm muscles.

One of our largest customers was restructuring their project hierarchy. I was talking to one of the engineers on the team that was doing back-end analytical processing of their data. Our customer's revision processing was stuck due to the massive scale of data that had traversed into a code path which hadn't ever seen that volume of data. The engineer asked for my help and we started poking at the code. What we saw was a [painter's algorithm](https://en.wikipedia.org/wiki/Joel_Spolsky#Schlemiel_the_Painter.27s_algorithm).

# What a Reproject!

A reproject happens when a project moves from one node in the project hierarchy to another. In this example, the customer was organizing their hierarchy into a "strategy" node (product management, architecture) and an "execution" node (development, operations). The snapshots that we capture include a project hierarchy in them to ease with authorization checks. So, a reproject of the scope the customer was doing turned all of the snapshot documents that look like this: 

````mongo
{
  ObjectId : 42,
  ValidFrom: '2015-1-31 12:00:00'
  ValidTo: '9999-12-31 23:59:00',
  ProjectHierarchy: [
    1, // root of project, say "My Company"
    100, // "R&D"
    1000, // "Development"
    1020 // "My Team"
  ]
}
````

... into a pair of snapshots, one containing the previous project hierarchy before the reprojecting and another showing the current hierarchy:

````mongo
{
  ObjectId : 42,
  ValidFrom: '2015-1-31 12:00:00'
  ValidTo: 9999-12-31,
  ProjectHierarchy: [
    1, // root of project, say "My Company"
    100, // "R&D"
    1000, // "Development"
    1020 // "My Team"
  ]
}
 
{
  ObjectId : 42,
  ValidTo: 9999-12-31,
  ProjectHierarchy: [
    1, // root of project, say "My Company"
    100, // "R&D"
    1400, // Engineering
    1000, // "Development"
    1020 // "My Team"
  ]
}
````

The code that does this transformation looks for documents with the old project hierarchy and then creates the two snapshots, ensuring that duplicate Object IDs are treated as a special case.

````groovy
def query = {ValidTo: END_OF_TIME, ProjectHierarchy: oldHierarchy}
def cursor = mongo.collection.find(query).sortBy("ObjectId")
def objectIdsWeSaw = []
while (cursor.hasNext() ) {
  def document = cursor.next()
  def objectId = document.ObjectId
  if ( ! objectIdsWeSaw.contains(objectId)) {
    objectIdsWeSaw.add(objectId)
    // do some stuff
 
  }
}
 
````

# What is Going On?

We did some logging and profiling on this code, but in this simplified example, you might notice a few problems with it. The first is that the "objectIdsWeSaw" variable is defined as an empty collection. In Groovy, a dynamically typed language, it's not clear from this line what the concrete type is of the object. In this case, the object is an Array List. Array Lists are great for certain things—they have a constant access time when you know the index of the element you're looking for—but have linear performance when searching the array. They also have to traverse the entire array on insertion and potentially resize the backing array of the array list. So, the more snapshots we were analyzing, the worse the performance. Like Schlemiel, we kept getting farther from the paint bucket when we were checking the array list and then inserting the new object ID.

There's also a problem with how Mongo is being queried here. In this case, the query is passing two indexed parameters to the query. By adding a sort clause with "ObjectId" to the find statement, we're asking Mongo to sort by a field that it doesn't have indexed. This causes Mongo to load the result set and sort the results in memory. In the case of the amount of data we were adding, this meant extra overhead. Also, the fact that we were sorting meant that our use of an Array List was always searching the entire array each iteration. That's worst-case performance.
How We Solved It

We fixed these two problems with two straightforward refactors. To solve the Mongo in-memory sorting issue, we were originally going to try to use distinct, however that has a limitation of only being able to handle the maximum BSON size (16 MB). So, we ended up removing the sort. That's it! Since we're already checking for uniqueness of Object IDs, the order in which we processed those snapshots didn't matter.

The data structure issue was also a simple fix. We needed a data structure that could track uniqueness and give us both constant time for insertion and lookup. HashSet to the rescue: It removes duplicates so the contains() and add() method calls become constant time operations.

Our new refactored code looks like this:

````groovy
def query = {ValidTo: END_OF_TIME, ProjectHierarchy: oldHierarchy}
def cursor = mongo.collection.find(query)
Set objectIdsWeSaw = new HashSet()
while (cursor.hasNext() ) {
  def document = cursor.next()
  def objectId = document.ObjectId
  if ( ! objectIdsWeSaw.contains(objectId)) {
    objectIdsWeSaw.add(objectId)
    // do some stuff
 
  }
}
 
````

Thanks to the fixes, we saw an operation that was projecting to finish in 5 days take just 25 minutes!

*Author's note - this post originally appeared in the Rallydev.com Engineering blog, which has been lost in the internet.  But thanks to the [wayback machine](https://web.archive.org), it is back online!*
