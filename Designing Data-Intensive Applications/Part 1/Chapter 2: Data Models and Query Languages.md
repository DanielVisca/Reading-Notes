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
