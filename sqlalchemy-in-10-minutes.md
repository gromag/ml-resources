---
layout: default
title: "SQLAlchemy in 10 minutes"
permalink: /sqlalchemy

---

# SQLAlchemy in 10 minutes

>_This document has redacted in various places however it remains mostly the output of a GPT 3 model. Large Language Models like GPT 3 might hallucinate and give give incorrect information, please use your own judgment and take this as an input to do further research._

Here's a quick guide to SQLAlchemy for people who are familiar with 
relational databases and other ORM Framework.

## Installation

You can install SQLAlchemy using pip. Simply open a terminal or command prompt and type pip install sqlalchemy.

## Importing

Once you have installed SQLAlchemy, you can import it into your Python project using the following line of code: 

```python 
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey, DateTime
```

This will import the create_engine function for connecting to a database, as well as various types of columns that you can use to define your database schema.

## Connecting to a database

 To connect to a database using SQLAlchemy, you will first need to create an engine. The engine is responsible for establishing a connection to the database. Here's an example of how to create an engine that connects to a Microsoft SQL Server database:

```python
engine = create_engine('mssql+pyodbc://<username>:<password>@<dsn_name>')
```
Replace <username> and <password> with your database username and password, and <dsn_name> with the name of your data source.

## Defining your database schema

 You can define your database schema using classes in SQLAlchemy. Here's an example of a simple schema for a blog:


```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class Blog(Base):
    __tablename__ = 'blogs'
    id = Column(Integer, primary_key=True)
    title = Column(String)
    content = Column(String)
    author_id = Column(Integer, ForeignKey('users.id'))
    author = relationship('User', backref='blogs')

class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    name = Column(String)
    email = Column(String)
```

In this example, we define two tables: blogs and users. The Blog table has a foreign key relationship to the User table.

## Querying the database

 You can use SQLAlchemy's query API to retrieve data from the database. Here's an example of how to retrieve all blogs written by a specific user:

```python
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()

user = session.query(User).filter_by(name='John Doe').first()
blogs = session.query(Blog).filter_by(author=user).all()

for blog in blogs:
    print(blog.title, blog.content)
```

In this example, we create a session using the engine we created earlier. We then query the User table to retrieve the user with the name "John Doe". We use the author attribute of the Blog table to filter for blogs written by this user, and we retrieve all of the matching blogs using the all() method. We then loop through the results and print the title and content of each blog.

## Modifying the database

You can use SQLAlchemy to modify the database as well. Here's an example of how to add a new blog:


```python
new_blog = Blog(title='New Blog Title', content='New Blog Content', author=user)
session.add(new_blog)
session.commit()
```

In this example, we create a new Blog object and add it to the session using the add() method. We then commit the changes to the database using the commit() method.

## Querying a table with multiple filters/conditions:

```python
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()

# Query all blogs written by a specific user that were posted in 2022
user = session.query(User).filter_by(name='John Doe').first()
blogs = session.query(Blog).filter_by(author=user).filter(Blog.posted_at.between('2022-01-01', '2022-12-31')).all()

for blog in blogs:
    print(blog.title, blog.content, blog.posted_at)
```

In this example, we use the `between()` method to filter blogs that were posted in 2022, and we chain the `filter_by()` and `filter()` methods to filter by both `author` and `posted_at`.

## Querying multiple tables with joins

```python
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()

# Query all blogs along with the name of the author
blogs = session.query(Blog, User.name).join(User).all()

for blog, author_name in blogs:
    print(blog.title, blog.content, author_name)
```
In this example, we use the `join()` method to join the Blog and User tables on the author_id foreign key. We then retrieve both the Blog object and the name of the author by passing Blog and User.name to the `query()` method. Finally, we loop through the results and print the title and content of each blog along with the name of the author.

You can also use the `outerjoin()` method to perform outer joins and the `select_from(`) method to specify the starting point for the query. For more information on querying multiple tables with SQLAlchemy, you can refer to the official documentation: https://docs.sqlalchemy.org/en/14/orm/query.html#joining-multiple-tables.

## An example of performing CRUD (Create, Read, Update, Delete) operations.

### Create operation

```python
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()

# Create a new blog
new_blog = Blog(title='New Blog Title', content='New Blog Content', author_id=1)
session.add(new_blog)
session.commit()
```

In this example, we create a new Blog object and add it to the session using the `add()` method. We then commit the changes to the database using the `commit()` method.

### Read operation

```python
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()

# Retrieve all blogs
blogs = session.query(Blog).all()

for blog in blogs:
    print(blog.title, blog.content)
```

In this example, we retrieve all Blog objects using the query() method and loop through the results to print the title and content of each blog.

### Update operation

```python
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()

# Update an existing blog
blog = session.query(Blog).filter_by(title='New Blog Title').first()
blog.content = 'Updated Blog Content'
session.commit()
```
In this example, we retrieve an existing Blog object using the `filter_by()` method and the `first()` method to return the first matching result. We then update the content attribute and commit the changes to the database using the `commit()` method.


### Delete operation

```python
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()

# Delete an existing blog
blog = session.query(Blog).filter_by(title='New Blog Title').first()
session.delete(blog)
session.commit()
```

In this example, we retrieve an existing Blog object using the `filter_by()` method and the `first()` method to return the first matching result. We then delete the object using the `delete()` method and commit the changes to the database using the `commit()` method.

Note that in all of these examples, we first create a session using the Session class and the `bind()` method to bind it to an engine. We then perform the desired CRUD operation using various methods of the session object.

## Transactions


```python
from sqlalchemy.orm import sessionmaker
from sqlalchemy.exc import IntegrityError

Session = sessionmaker(bind=engine)

# Start a transaction
session = Session()
transaction = session.begin()

try:
    # Create a new blog
    new_blog = Blog(title='New Blog Title', content='New Blog Content', author_id=1)
    session.add(new_blog)

    # Update the author's name
    author = session.query(User).filter_by(id=1).first()
    author.name = 'New Author Name'

    # Commit the transaction
    session.commit()
except IntegrityError as e:
    # Roll back the transaction if an error occurs
    transaction.rollback()
    print(f'An error occurred: {e}')
```

In this example, we start a transaction by calling the `begin()` method on the session object. We then perform multiple operations within the transaction, including creating a new Blog object and updating the name of the author. We then commit the transaction using the `commit()` method. If an error occurs during the transaction, such as a violation of a unique constraint, we catch the IntegrityError exception and roll back the transaction using the `rollback()` method. This ensures that all changes made during the transaction are rolled back and the database is left in a consistent state.

Note that transactions are particularly useful when you need to perform multiple related operations within a single atomic unit, such as transferring funds between bank accounts or updating multiple tables in a database. By wrapping these operations in a transaction, you can ensure that they are either all committed or all rolled back in the event of an error. This helps to maintain the integrity of your data and avoid situations where only part of an operation is completed, leaving the database in an inconsistent state.

## Example of writing efficient queries in SQLAlchemy

Use the `select()` method to select only the columns you need:

```python
from sqlalchemy.orm import sessionmaker
from sqlalchemy import select

Session = sessionmaker(bind=engine)
session = Session()

# Select only the title and content columns
blogs = session.execute(select(Blog.title, Blog.content)).fetchall()

for blog in blogs:
    print(blog.title, blog.content)
```

In this example, we use the `select()` method to select only the title and content columns of the Blog table, rather than selecting all columns using the `query()` method. This can help to improve query performance, especially if you have tables with many columns or if you only need to retrieve a subset of columns.

Use the `filter()` method to filter your query results:

```python
from sqlalchemy.orm import sessionmaker
from sqlalchemy import and_

Session = sessionmaker(bind=engine)
session = Session()

# Retrieve blogs written by a specific author that were posted in 2022
author = session.query(User).filter_by(name='John Doe').first()
blogs = session.query(Blog).filter(and_(Blog.author_id==author.id, Blog.posted_at.between('2022-01-01', '2022-12-31'))).all()

for blog in blogs:
    print(blog.title, blog.content)
```

In this example, we use the `filter()` method to filter our query results based on multiple conditions using the `and_()` method. This can help to reduce the amount of data that needs to be retrieved from the database, especially if you have large tables or complex queries.

Use the `join()` method to join tables efficiently:

```python
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()

# Retrieve all blogs along with the name of the author
blogs = session.query(Blog, User.name).join(User, Blog.author_id==User.id).all()

for blog, author_name in blogs:
    print(blog.title, blog.content, author_name)
```

In this example, we use the `join()` method to efficiently retrieve data from multiple tables in a single query, rather than performing separate queries and then merging the results. This can help to reduce the number of round-trips to the database and improve query performance.

These are just a few examples of how you can write efficient queries in SQLAlchemy. There are many other techniques you can use, such as indexing your database tables, using subqueries, and using SQLAlchemy's built-in query caching functionality. The key is to always think carefully about the data you need to retrieve and how best to retrieve it, and to use the tools provided by SQLAlchemy to optimize your queries accordingly.

## SQLAlchemy and raw SQL

SQLAlchemy provides several ways to execute raw SQL queries, including:

Using the `execute(`) method of the engine object:

```python
from sqlalchemy import create_engine

engine = create_engine('postgresql://user:password@host:port/dbname')

# Execute a raw SQL query
result = engine.execute('SELECT * FROM my_table')

for row in result:
    print(row)
```

In this example, we create an engine to connect to a PostgreSQL database, and then use the `execute()` method to execute a raw SQL query. The `execute()` method returns a ResultProxy object that we can use to retrieve the results of the query.

Using the `text()` function to define a SQL expression:

```python
from sqlalchemy.orm import sessionmaker
from sqlalchemy import text

Session = sessionmaker(bind=engine)
session = Session()

# Execute a raw SQL query using a text expression
result = session.query(Blog).from_statement(text('SELECT * FROM blogs WHERE author_id = :author_id')).params(author_id=1).all()

for blog in result:
    print(blog.title, blog.content)
```

In this example, we use the `text()` function to define a SQL expression that we can pass to the `from_statement()` method of a Query object. We then use the `params()` method to pass in any parameters required by the query.

Using the connection object to execute a raw SQL query:

```python
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()

# Execute a raw SQL query using a connection object
with session.connection() as con:
    result = con.execute('SELECT * FROM blogs WHERE author_id = %s', 1)

for row in result:
    print(row)
```

In this example, we use the `connection()` method of the session object to obtain a connection object, which we can use to execute a raw SQL query using the `execute()` method.

Using raw SQL with SQLAlchemy can be useful in situations where you need to execute a complex query that would be difficult or inefficient to express using SQLAlchemy's query API. However, be aware that using raw SQL can make your code more difficult to read and maintain, and can introduce the risk of SQL injection vulnerabilities. Be sure to carefully validate any user input before including it in a raw SQL query, and consider using SQLAlchemy's query API whenever possible.

### Preventing SQL Injection when using raw SQL

Parameterizing input is an important technique for preventing SQL injection attacks when using raw SQL queries. Here's an example of how to parameterize input in a raw SQL query using SQLAlchemy:

```python

from sqlalchemy.orm import sessionmaker
from sqlalchemy import text

Session = sessionmaker(bind=engine)
session = Session()

# Execute a raw SQL query with parameterized input
author_id = 1
query = text('SELECT * FROM blogs WHERE author_id = :author_id')
result = session.query(Blog).from_statement(query).params(author_id=author_id).all()

for blog in result:
    print(blog.title, blog.content)
```

In this example, we use a text expression to define a raw SQL query with a named parameter `:author_id`. We then use the `params()` method to pass in a value for the author_id parameter, which is sanitized by SQLAlchemy to prevent SQL injection attacks.

Note that the `params()` method can be used to pass in multiple named parameters at once, and can also accept positional arguments. 

For example:

```python
query = text('SELECT * FROM blogs WHERE author_id = :author_id AND posted_at > :posted_at')
result = session.query(Blog).from_statement(query).params(author_id=1, posted_at='2022-01-01').all()
```

In this example, we pass in values for both the author_id and posted_at parameters, and these values are parameterized to prevent SQL injection attacks.

By using parameterized input in your raw SQL queries, you can help to ensure that your code is secure and free from SQL injection vulnerabilities. Just be sure to validate any user input before including it in a raw SQL query, and to use the parameterized input feature of SQLAlchemy whenever possible.

## Fetching data efficiently using Generators


In SQLAlchemy, you can use a generator to efficiently fetch all records from a table in chunks, rather than fetching all records at once. This can be especially useful when working with large tables that may not fit into memory all at once.

Here's an example of how to fetch all records from a table using a generator in SQLAlchemy:

```python
from sqlalchemy.orm import sessionmaker
from typing import Generator, TypeVar

Session = sessionmaker(bind=engine)
session = Session()

T = TypeVar('T')

def get_records_in_chunks(query) -> Generator[T, None, None]:
    offset = 0
    while True:
        chunk = query.offset(offset).limit(chunk_size).all()
        if not chunk:
            return
        offset += chunk_size
        for record in chunk:
            yield record

query = session.query(MyTable)

for record in get_records_in_chunks(query):
    # Process each record here

```

In this example, we define a generator function called `get_records_in_chunks()` that takes a SQLAlchemy query object and a chunk_size parameter. The function uses the `offset()` and `limit()` methods of the query object to fetch records in chunks of `chunk_size` at a time. The function then yields each record in the chunk, and continues fetching more records until there are no more records to fetch.

We then define a SQLAlchemy query object called query to query the desired table, and pass this object to `get_records_in_chunks()` to fetch records in chunks. We can then process each record in the table by iterating over the generator returned by `get_records_in_chunks()`.

By using a generator to fetch records in chunks, we can reduce memory usage and improve performance when working with large tables in SQLAlchemy.


## Disclaimer and Acknowledgement

This document has redacted in various places however it remains mostly the output of a GPT 3 model.

Large Language Models like GPT 3 might hallucinate and give give incorrect information, please use your own judgment and take this as an input to do further research.