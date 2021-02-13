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
