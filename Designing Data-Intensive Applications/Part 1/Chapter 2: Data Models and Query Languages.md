# Chapter 2: Data Models and Query Languages

Data models are the most important part of software. Each layer should hide the complexity of the layers below it by providing a clean data model. This chapter will compare use cases of the relational model, document model, and a few graph-based data models. It will also look at various querying languages and their use cases.

## Relational Model Versus Document Model

SQL has dominated for an eternity in computer history (it was proposed in 1970). Competing ideas often arise with a lot of hype then lose out to SQL.

### The Birth of NoSQL

NoSQL is an unfortunate name as it is actually 'Not only SQL'

Driving forces between the adoption of NoSQL:

* A need for greater scalability than relational databases can easily achieve, including very large datasets or very high write throughput. (NOTE: what is the definition of throughput?)
* A widespread preference for free and open source software over commercial database products.
* Specialized query operations that are not well supported by the relational model.
* Frustration with the restrictiveness of relational schemas, and a desire for a more dynamic and expressive data model.

*polyglot persistence*: relational databases being used alongside non-relational databases.

The future likely will consist of polyglot persistence.

### The Object-Relational Mismatch

Most applications are made using object-oriented programming languages. Relational databases and object-oriented applications require an awkward transition layer a.k.a. *impedance mismatch*.

Object-Relational Mapping (ORM) tries to hide this awkwardness but cannot do it completely.

A resume is appropriate for non-relational JSON format as it is a mostly self contained *document*.

A JSON representation has better *locality* than a multi-table schema. Meaning structuring a specific scenario is much easier. (**NOTE**: Implying that relational representations are better at global representation?)

### Many-to-One and Many-to-Many Relationships

In a resume relational model, a user table would have an `region_id` instead of a text field, linking them to a region tables record. This helps with:

* Consistent style and spelling across profile.
* Avoiding ambiguity (ex: several cities with same name)
* Ease of updating as the name is stored in only one place, id's are just pointers.
* Localization support, when the site is translated into other languages, the standardized list can be localized so the region and industry can be displayed in the viewers language (NOTE: I had not considered this before!)
* Better Search, ex: if `region` was stored as the text 'Greater Seattle Area' we could not generalize for a search looking for 'State of Washington'.

Storing an ID or a text string is a question of duplication. Human-meaningful information should be stored in as few places as possible and the rest should be computer-meaningful such as `id`. computer-meaningful information never needs to change, human-meaningful does. (**NOTE**: This could be helpful when trying to design a database as well as an application!)

Normalization is intended to minimize human-meaningful duplication.
Unfortunately normalization requires *many-to-one* relationships ex: many people live in one particular region. This does not fit nicely in the document model where support for joins is weak and usually must be done at the application level.

### Are Document Databases Repeating History

Before SQL, the most popular database was IBM's *Information Management System* (IMS). It used the *hierarchical model* which has a lot of similarities with to the JSON model used by document databases. It represented all data as a tree of records nested within records. This worked well for one-to-many relationships but made many-to-many relationships difficult with no join support. To resolve either have to duplicate (denormalize) or manually resolve references from one record to another. This is the same situation as with NoSQL.

To resolve this issue there was a great debate between two alternative solutions, the *network model* or the *relational model* a.k.a. SQL.

We ill revisit these debates.

#### The network model

The hierarchical model was a tree where each node had one parent. The network model allows a node to have multiple parents allowing for many-to-many relations. This implementation is essentially graph theory. To access a record we need to start at the root then go along these chains of links. This was called an *access path*. Programmers needed to keep track of access paths. Access paths weren't a bad way for looking up but updating and querying was essentially navigating through an n-dimensional data space.

(**NOTE**: It will be interesting to learn if is this is what GraphQL does but with a more standardized method for access paths... actually thinking about it, it is sort of like the hierarchical model. Something to look into.)

#### The relational model

Easier to add new features to applications. Data was laid out in the open. Access paths could be automatically calculated for most optimal look-ups. Query optimizers are very complicated, but you only need to make them once then all applications that use the database benefit from it.

#### Comparison to document databases

Document databases use the hierarchical model for records, but for many-to-one and many-to-many relationships it uses foreign-keys/document-reference which is resolved at read time by using joins or follow up queries.

### Relational Versus Document Databases Today

Benefits of document model:

* schema flexibility.
* better performance due to locality.
* for some applications it is closer to the data structures used by the application.

Benefits of relational model:

* better support for joins.
* better at many-to-one and many-to-many relationships.

#### Which data model leads to simpler application code?

If the data in your application has a document-like structure (i.e. a tree of one-to-many relationships, where typically the entire tree is loaded at once), the you should probably use the document modal. The relational technique of *shredding* -splitting a document like structure into multiple tables can lead to a cumbersome schema and needlessly complicated code.

If your application uses many-to-many relationships then the document model is less appealing.

(NOTE: Can joins be done with mongoDB? Or must they be done at the application level?)
Application level joins are less efficient and adds complexity to the application.

For highly interconnected data, the document model is awkward, the relational model is acceptable, and the graph model are the most natural.

#### Schema flexibility in the document model

Document models usually have an 'implicit' schema that is not enforced by the database The application is expecting a certain format but only validates when the data is read *schema-on-read*.

How to enforcement a schema has no right or wrong answer.

Schema changes have a reputation of being time consuming (for relational), they don't always have to be. MySQL is an exception as it's method takes a long time (but can be circumvented).

Some reasons for wanting flexible schemas:

* There are many different types of objects and it is not practical to have a table for all of them
* The structure of the data is determined by external sources over which you have no control and may change over time.

#### Data locality for queries

If your application often needs to access the entire document (ex: render on webpage), there is a performance advantage to *storage locality*. If data is split across tables, this could be a hinderance.

(NOTE: For my current project (Veracity), an article has copies of mbfc data. This is duplication. Each document should have a reference.)

The database model needs to load the whole document even if you only need a small portion of it. Locality is only beneficial when you need most of the document on every lookup.

Document updates usually rewrite the entire document causing them to be inefficient.

General recommendation:

* Keep documents small.
* avoid writes that increase the size of a document.

Googles spanner database offers locality properties in a relational database.

Oracle uses *multi table indexing cluster tables* to do the same.

#### Convergence of document and relational databases

PostgreSQL and MySQL have added support for JSON documents. Other relational databases are likely to follow suit. MongoDB automatically resolves joins through references however this is less optimized than a relational join.
Relational and document models are becoming more similar over time as they complement each other. A hybrid model is a good route for future databases.

## Query Languages for Data

* Declarative query language: looks like relational algebra where the conditions that must be met are specified but how that is done is not specified. It is instead taken care of by the system's query optimizer.
* Imperative query language: Tells the computer to perform a certain operation in a certain order. This resembles code.

Declarative languages are attractive because they add an extra layer of abstraction. A facade. This allows for improvement performances to be done behind the scenes without changing the queries.

The fact that SQL is more limited in functionality gives the databases much more room for automatic optimizations.

Imperative languages are difficult to execute in parallel as the specific instructions of how to run the query are specified, whereas declarative languages abstract away the specific implementation and can more easily have a query executed in parallel.

### Declarative Queries on the Web

CSS and XSL are both declarative languages. This simplifies how to find the correct elements in HTML. For example in css.

```css
li.selected > p {
    background-color: blue;
}
```

The CSS selector `li.selected > p` declares the pattern of elements to which we want to apply the blue style: namely, all `<p>` elements whose direct parent is an `<li>` element is a CSS class of `selected`.

This provides the conditions that must be met but not how to search for elements that meet these conditions.

Alternatively we would have to use JavaScript to implement this functionality through some very messy code that is difficult to change or update and must be done manually.

(NOTE: He mentions 'imperative query APIs' which is what JavaScripts `document.getElementById('selected')` is. I have not thought of this as an API before.)

### MapReduce Querying

*MapReduce* is a programming model for processing large amounts of data in bulk across many machines, popularized by Google.

MongoDB uses a limited version of this for performing read-only queries across many documents.

This combines both declarative and imperative languages. The overall structure of a query may be declarative, however there can be pure functions that are called when certain tables or conditions are met that.

MongoDB uses a query language called *aggregation pipeline* which uses a JSON based language and essentially is reinventing SQL. (NOTE: I have had some experience with this language when using `$`)

A usability issue with MapReduce is that you need to write 2 carefully crafted Javascript functions which can be harder than writing a single query. That is why MongoDB added support for aggregation pipeline.

Moral of the story: NoSQL may find itself reinventing SQL albeit in disguise.

## Graph-Like Data Models

What happens if many-to-many relationships are very common in your data? It becomes natural to model your data as a graph.

Graphs consist of two kinds of objects:

* vertices (a.k.a. nodes or entities)
* edges (a.k.a. relationships or arcs)

Examples:

* Social graphs: Vertices are people and edges indicate which people know each other.
* The web graph: Vertices are web pages and the edges indicate HTML links to other pages.
* Road or rail networks: Vertices are junctions, and edges represent the roads or railway lines between them.

Graphs do not require that all vertices or edges are the same type. Facebook uses vertices for people, locations, events etc...

The following are ways to structure and query graphs.

### Property Graphs

Vertex contains:

* unique ID
* set of outgoing edges
* set of incoming edges
* collection of properties (key-value pairs)

Edge contains:

* unique ID
* vertex where the edge starts (tail vertex)
* head vertex
* label describing the kind of relationship between the two vertices
* a collection of properties (key-value pairs)

This can be stored in two relational tables. One for Vertices and one for Edges.

Graphs are good for evolvability. Just add new vertices and edges to add new types of relationships with new things.

### The Cypher Query Language

*Cypher* is a declarative query language for property graphs. It is named after the character in the matrix not after cryptography.

The language is quite comprehensive but does not specify how to look for the query, just the conditions that must be met.

### Graph Queries in SQL

You can have a graph in SQL and query it with SQL but there are some difficulties. The number of JOINS is not fixed in advanced (as it usually is with most SQL queries) as the query could be trying to find a relationship that is a chained together rather than one link away. SQL can handle this using *recursive common table expressions* the syntax us 'WITH RECURSIVE'. But the query syntax becomes quite clunky.

An easy indication on if you are using the correct data model is how large and cumbersome your queries are, the smaller and simpler the better.

### Triple-Stores and SPARQL

The triple-store model is very similar to the property graph model.

All information is stored in very simple three part statements (Subject, verb, object) example (Jim, likes, bananas). Hence 'triple-store'. The subject is a vertex. The object is either a vertex or a primitive datatype. If it is a primitive datatype is is essentially a property of the subject (Lucy, age, 33).

#### The semantic web

This is completely separate from triple-stores but are usually linked in peoples minds.
The idea is that webpages right now are computer readable but the same method could be used for computer readable data. The internet would essentially become a database of everything. There was a lot of hype on this in the 2000s but not much has happened.

#### The RDF data model

The *Resource Description Framework* was a standardized way for data to be published on the web for the semantic web.

#### The SPARQL query language

A query language for triple-store using the RDF data model. Pronounced *sparkle*.

It is a very concise language and looks like a combination of SQL and Cypher. A language that we should consider using internally for applications.

### Graph Databases Compared to the Network Model

In summary, they are very different and the graph database is not repeating history.

### The Foundation: Datalog

A much older language that other query languages build on. Essentially what C is for programming languages.

Similar to triple-store except instead of *(subject, predicate, object)* it is written as *predicate(subject, object)*.

Complicated queries can be built up bit by bit through *rules*. Similar to how functions can call other functions. We can define new predicates that we use further down in the query.

This language is less convenient for one off queries but can cope better for with complicated data.

## Summary

* document databases target use cases where data comes in self-contained documents and relationships between one document and another are rare.
* graph databases go in the opposite direction, targeting use cases where anything is potentially related to everything.
