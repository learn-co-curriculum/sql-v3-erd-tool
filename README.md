# Entity Relationship Modeling Tools

## Learning Goals

- Use `dbdiagram.io` to create an **Entity Relationship Diagram (ERD)**
- Write `DBML` code to add entities to the ERD.
- Write `DBML` code to add one-to-one, one-to-many, and many-to-many relationships to the ERD
- Export an ERD as an `SQL` script.

## Introduction

An entity relationship model is visually depicted as an **entity relationship diagram (ERD)**.
We will create an ERD using a free, browser-based software tool available at [dbdiagram.io](https://dbdiagram.io/home).

## Code-Along 

## dbdiagram.io

There are many software tools available for creating ERDs.  We will
use [dbdiagram.io](https://dbdiagram.io/home) to create an ERD by writing
code in an open-sourced database markup language named `DBML`.

1. Go to [https://dbdiagram.io/home](https://dbdiagram.io/home)
2. Click either "Create your diagram" or "Go to App".  
   ![dbdiagram.io](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-erd-tool/dbdiagram.png)
3. A sample ERD is generated as shown below.    We will create a new ERD from scratch.
   Move your mouse over the `dbdiagram.io` icon in the upper left part of the window and select "New Diagram".    
   ![new diagram](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-erd-tool/new_diagram.png) 
4. Sign in using email, Google, or Github.   
   ![signin dbdiagram.io](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-erd-tool/signin.png)
5. You may need to select "New Diagram" again after signing in to get an empty diagram.  
   ![empty diagram](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-erd-tool/empty_diagram.png)

## Creating entities

Let's create an ERD for a book publishing database.

- A book has one publisher and may have multiple authors.
- An author may write multiple books.
- A publisher publishes multiple books.
- A publisher's headquarters is located at a unique address.

We will model address as a separate entity to demonstrate a one-to-one relationship
with the publisher. Thus, our ERD will include four entities: book, author, publisher, and address.

The `DBML` syntax to define an entity looks like this:

```text
Table table_name {
    column_name column_type [column_settings]
}
```

The `dbdiagram.io` tool requires at least one field to be defined for each entity,
so we will initially define each entity with a unique integer id for the primary key:

```text
Table Author {
  id INTEGER PK
}

Table Book {
  id INTEGER PK
}

Table Publisher {
  id INTEGER PK
}

Table Address {
  id INTEGER PK
}
```

1. Enter the entity definitions in the code editor panel.
   You can drag the entities to rearrange the layout.
2. Save the ERD as "publishing ERD".

![publishing entities](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-erd-tool/publishing_entities.png)

The `PK` constraint indicates the field is a primary key, which is displayed in bold-face font.

Let's refine the model to include attributes for each entity, along with attribute constraints (unique, not null, default values, etc).

```text
Table Author {
  id INTEGER PK
  first_name TEXT [NOT NULL]
  last_name TEXT [NOT NULL]
}

Table Book {
  id INTEGER PK
  title TEXT [NOT NULL]
  isbn TEXT [NOT NULL, UNIQUE]
  publication_date DATE 
  pages INTEGER 
  edition INTEGER [DEFAULT: 1]
}

Table Publisher {
  id INTEGER PK
  name TEXT [NOT NULL, UNIQUE]
  website TEXT 
}

Table Address {
  id INTEGER PK
  street TEXT 
  city TEXT [NOT NULL]
  state TEXT
  zip TEXT
  country TEXT [NOT NULL, DEFAULT: 'United States']
}
```

![publishing attributes and constraints](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-erd-tool/publishing_attributes.png)

The attribute constraints do not appear in the ERD.
However, they will be reflected in the SQL table creation
statements when we export the model.

## Creating relationships

Now we will add relationships between entities.
`DBML` supports four relationship types:

```text
//4 Relationship Types
   <   one-to-many
   >   many-to-one
   -   one-to-one
   <>  many-to-many
```

- There is a one-to-one relationship between `Publisher` and `Address`.
- There is a one-to-many relationship between `Publisher` and `Book`.
  - This can also be modeled as a many-to-one relationship between `Book` and `Publisher`.
- There is a many-to-many relationship between `Book` and `Author`.


### One-To-One Relationship 

Let's start with the one-to-one relationship between
`Publisher` and `Address` entity.  Each publisher's headquarters
is located at a unique address.

The one-to-one relationship can be stored in either the
`Publisher` or `Address` entity.  We will store the relationship
in the `Publisher` entity as a new field named `address_id`.

Edit the `Publisher` entity to add the new field and foreign key reference:

```text
Table Publisher {
  id INTEGER PK
  name TEXT [NOT NULL, UNIQUE]
  website TEXT 
  address_id INTEGER [UNIQUE, ref: - Address.id]
}
```

- The`ref` constraint defines `address_id` as a foreign key referencing
  the `Address` entity's primary key `id`.
- The `UNIQUE` constraint ensures the relationship is one-to-one,
  i.e. two publishers can't have the same value for `address_id`.

Click the "Highlight" toggle to display a numeric value for the relationship cardinality.

![one-to-one relationship](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-erd-tool/one_to_one.png)

### One-To-Many Relationship 

Now we will add the one-to-many relationship between `Publisher` and `Book`
The relationship can also be modeled as many-to-one between `Book` and `Publisher`.

It is best to store the relationship as an attribute in the entity
that is on the "many" side (i.e. `Book`). Thus, we will add an attribute
named `publisher_id` in the `Book` entity and use  the many-to-one syntax `>`  for
the relationship symbol (many books may have the same publisher).

Edit the `Book` entity to add the new field and foreign key reference:

```text
Table Book {
  id INTEGER PK
  title TEXT [NOT NULL]
  isbn TEXT [NOT NULL, UNIQUE]
  publication_date DATE 
  pages INTEGER 
  edition INTEGER [DEFAULT: 1]
  publisher_id INTEGER [ref: > Publisher.id]
}
```

- The`ref` constraint defines `publisher_id` as a foreign key referencing
  the `Publisher` entity's primary key `id`.
- Omitting the `UNIQUE` constraint allows many books to have the same publisher.

![one-to-many relationship](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-erd-tool/one_to_many.png)

Now we will add the many-to-many relationship between `Book` and `Author`.
We will look at two approaches for modeling the many-to-many relationship.

### Many-To-Many Relationship using `<>` syntax

We can't store the many-to-many relationship as an attribute in `Book` or `Author`
since both entities are on the "many" side of the relationship.
We can however define a `Ref` constraint separate from any entity
using the `<>` syntax to indicate many-to-many:

```text
Ref Book_Author: Book.id <> Author.id   // many-to-many
```

![many-to-many version1](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-erd-tool/many_to_many_v1.png)

Recall that a many-to-many relationship requires a join table `Book_Author`
with a composite primary key consisting of the primary keys
of both `Book` and `Author`.   While the ERD does not create
a new entity to represent the join table, we can export
the ERD to see the SQL statements for creating the schema
defined by the entity relationship model.

1. Select `Export to PostgreSQL` from the toolbar.   
   ![export erd](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-erd-tool/export_v1.png)
2. Open the exported file `publishing ERD.sql`.  The SQL creates a table `Book_Author` with
   a composite primary key to represent the many-to-many relationship:
    ```text
    CREATE TABLE "Book_Author" (
      "Book_id" INTEGER,
      "Author_id" INTEGER,
      PRIMARY KEY ("Book_id", "Author_id")
    );
    ALTER TABLE "Book_Author" ADD FOREIGN KEY ("Book_id") REFERENCES "Book" ("id");
    ALTER TABLE "Book_Author" ADD FOREIGN KEY ("Author_id") REFERENCES "Author" ("id");
    ```
   
The exported code also contains a foreign key constraint to implement
the one-to-one relationship:

```text
ALTER TABLE "Publisher" ADD FOREIGN KEY ("address_id") REFERENCES "Address" ("id");
```

The one-to-many relationship is implemented as a foreign key constraint as well:

```text
ALTER TABLE "Book" ADD FOREIGN KEY ("publisher_id") REFERENCES "Publisher" ("id");
```

### Many-To-Many Relationship As New Entity

The many-to-many relationship can also be modeled as a new entity
`Book_Author` with a composite primary key that is the result
of `Book` and `Author` each having a one-to-many relationship with `Book_Author`.

We store the relationship in the entity that is one the "many" side,
which is `Book_Author` (`Book_Author` has two separate many-to-one relationships
with `Book` and `Author`).  

Remove the `Book_Author` reference that implemented the relationship
using the many-to-many `<>` syntax, and
add the new `Book_Author` entity as shown:

```text
Table Book_Author {
  book_id INTEGER [PK, ref: > Book.id] // many-to-one
  author_id INTEGER [PK, ref: > Author.id] // many-to-one
}
```

![many-to-many version2](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-erd-tool/many_to_many_v2.png)

Exporting the ERD shows a similar schema as produced with modeling the
many-to-many relationship using `<>` syntax:

```text
CREATE TABLE "Book_Author" (
  "book_id" INTEGER,
  "author_id" INTEGER,
  PRIMARY KEY ("book_id", "author_id")
);

ALTER TABLE "Book_Author" ADD FOREIGN KEY ("book_id") REFERENCES "Book" ("id");
ALTER TABLE "Book_Author" ADD FOREIGN KEY ("author_id") REFERENCES "Author" ("id");
```

Using a new entity `Book_Author` to model the many-to-many relationship
between `Book` and `Author`is particularly useful when the relationship results in additional attributes.
For example, we may want to store the royalties (i.e. 20% or 30% of book sales)
an author receives for writing a particular book.  In this case, we can
add the attribute to the `Book_Author` entity as shown:

```text
Table Book_Author {
  book_id INTEGER [PK, ref: > Book.id] // many-to-one
  author_id INTEGER [PK, ref: > Author.id] // many-to-one
  royalties DECIMAL
}
```

![many-to-many with attributes](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-erd-tool/royalties.png)

## Export final version of ERD

Let's export the ERD to see the final version of the database schema:

```text
CREATE TABLE "Author" (
  "id" INTEGER PRIMARY KEY,
  "first_name" TEXT NOT NULL,
  "last_name" TEXT NOT NULL
);

CREATE TABLE "Book" (
  "id" INTEGER PRIMARY KEY,
  "title" TEXT NOT NULL,
  "isbn" TEXT UNIQUE NOT NULL,
  "publication_date" DATE,
  "pages" INTEGER,
  "edition" INTEGER DEFAULT 1,
  "publisher_id" INTEGER
);

CREATE TABLE "Publisher" (
  "id" INTEGER PRIMARY KEY,
  "name" TEXT UNIQUE NOT NULL,
  "website" TEXT,
  "address_id" INTEGER UNIQUE
);

CREATE TABLE "Address" (
  "id" INTEGER PRIMARY KEY,
  "street" TEXT,
  "city" TEXT NOT NULL,
  "state" TEXT,
  "zip" TEXT,
  "country" TEXT NOT NULL DEFAULT 'United States'
);

CREATE TABLE "Book_Author" (
  "book_id" INTEGER,
  "author_id" INTEGER,
  "royalties" DECIMAL,
  PRIMARY KEY ("book_id", "author_id")
);

ALTER TABLE "Book" ADD FOREIGN KEY ("publisher_id") REFERENCES "Publisher" ("id");

ALTER TABLE "Publisher" ADD FOREIGN KEY ("address_id") REFERENCES "Address" ("id");

ALTER TABLE "Book_Author" ADD FOREIGN KEY ("book_id") REFERENCES "Book" ("id");

ALTER TABLE "Book_Author" ADD FOREIGN KEY ("author_id") REFERENCES "Author" ("id");

```

## Conclusion

We can use `DBML` to model and entity and its attributes:

```text
Table Publisher {
  id INTEGER PK
  name TEXT [NOT NULL, UNIQUE]
  website TEXT 
}
```

A one-to-one relationship between `Publisher` and `Address`
is modeled as a unique reference to the associated
entity's primary key:

```text
Table Publisher {
  id INTEGER PK
  name TEXT [NOT NULL, UNIQUE]
  website TEXT 
  address_id INTEGER [UNIQUE, ref: - Address.id]  // one-to-one
}
```

A one-to-many relationship between `Publisher` and `Book`
is a many-to-one relationship between `Book` and `Publisher`.
The relationship is defined using `DBML`
in the entity on the **many** side of the relationship (`Book`)
as a reference to the **one** associated entity (`Publisher`):

```text
Table Book {
  id INTEGER PK
  title TEXT [NOT NULL]
  isbn TEXT [NOT NULL, UNIQUE]
  publication_date DATE 
  pages INTEGER 
  edition INTEGER [DEFAULT: 1]
  publisher_id INTEGER [ref: > Publisher.id] // one-to-many
}
```

A many-to-many relationship between `Book` and `Author`
can be modeled as a stand-alone reference:


```text
ref Book_Author: Book.id <> Author.id // many-to-many
```

The many-to-many relationship
can also be modeled as a new entity `Book_Author` that has two
separate many-to-one relationships with `Book` and `Author`:

```text
Table Book_Author {
  book_id INTEGER [PK, ref: > Book.id] // many-to-one
  author_id INTEGER [PK, ref: > Author.id] // many-to-one
}
```

## Resources

- [dbdiagram.io](https://dbdiagram.io)   
- [dbdiagram.io documentation](https://dbdiagram.io/docs/)      
- [DBML](https://www.dbml.org/docs/)