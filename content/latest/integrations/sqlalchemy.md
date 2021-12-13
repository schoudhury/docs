---
title: Using SQLAlchemy with YugabyteDB
linkTitle: SQLAlchemy
description: Using SQLAlchemy with YugabyteDB
aliases:
menu:
  latest:
    identifier: sqlalchemy
    parent: integrations
    weight: 650
isTocNested: true
showAsideToc: true
---

This document describes how to use [SQLAlchemy](https://www.sqlalchemy.org/), a Python SQL tool and object-relational mapping (ORM) tool, with YugabetyDB.

## Prerequisites

Before you can start using SQLAlchemy, ensure that you have the following available:

- YugabyteDB version 2.6 or later (see [YugabyteDB Quick Start Guide](/latest/quick-start/)).

- Yugabyte cluster (see [Create a local cluster](/latest/quick-start/create-local-cluster/macos/)). 

- Python version 2.7 or later.

- The latest version of SQLAlchemy, which you can install using pip by executing the following command:

  ```shell
  pip3 install sqlalchemy
  ```

  You can verify the installation as follows: 

  - Open the Python prompt by executing the following command:

    ```shell
    python3
    ```

  - From the Python prompt, execute the following commands to check the SQLAlchemy version:

    ```python prompt
    import sqlalchemy
    ```

    ```python prompt
    sqlalchemy.version
    ```

- Psycopg2, the PostgreSQL database adapter for Python, which you can install using pip by executing the following command:

  ```shell
  pip3 install psycopg2
  ```

  Alternatively, you can install [psycopg2-binary](https://www.psycopg.org/docs/install.html), a pre-compiled version of the module, by executing the following command:

  ```shell
  pip3 install psycopg2-binary
  ```

## Using SQLAlchemy

You can start using SQLAlchemy with YugabyteDB as follows:

- Create a demo project and add a `main.py` and `config.py` files to it.

- Add the followingn code to the `config.py` file:

  ```python
  db_user = 'yugabyte'
  db_password = 'yugabyte'
  database = 'yugabyte'
  db_host = 'localhost'
  db_port = 5433
  ```

- Add the followingn code to the `main.py` file:

  ```python
  import config as cfg
  from sqlalchemy.ext.declarative import declarative_base
  from sqlalchemy.orm import sessionmaker, relationship
  from sqlalchemy import create_engine
  from sqlalchemy import MetaData
  from sqlalchemy import Table, Column, Integer, String, DateTime, ForeignKey
  Base = declarative_base()
   
  class Test(Base):
   
     __tablename__ = 'test'
   
     id = Column(Integer, primary_key=True)
     name = Column(String(255), unique=True, nullable=False)
   
  # create connection
  engine = create_engine('postgresql://{0}:{1}@{2}:{3}/{4}'.format(cfg.db_user, cfg.db_password, cfg.db_host, cfg.db_port, cfg.database))
   
  # create metadata
  Base.metadata.create_all(engine)
   
  # create session
  Session = sessionmaker(bind=engine)
  session = Session()
   
  # insert data
  tag_1 = Test(name='Bob')
  tag_2 = Test(name='John')
  tag_3 = Test(name='Ivy')
   
  session.add_all([tag_1, tag_2, tag_3])
  session.commit()
  
  ```

- Execute the code using the following command:

  ```shell
  python3 main.py
  ```

## Testing the code

You can verify the code execution by looking for the changes inside the database, as follows:

-  Navigate to your YugabyteDB installation directory by running the following command:

  ```shell
  cd /<path-to-yugabytedb>
  ```

- Run the ysqlsh client by executing the following command: 

  ```shell
  ./bin/ysqlsh
  ```

- Obtain the list of all the tables in the database by executing the following command:  

  ```shell
  \dt
  ```

- Check if rows have been inserted into the table by executing the following:

  ```sql
  SELECT * FROM TEST;
  ```
  
  The output should be as follows:
  
  ```output
  id | name
  ---+--------
   1 | Bob
   2 | John
   3 | Ivy
  (3 rows)
  ```

## Limitations

Consider the following limitations:

- Since YugabyteDB does not support savepoints in transactions, you cannot use savepoints with SQLAlchemy ORM. 
- Due to the distributed nature of YugabyteDB, rows returned by a query might not be in sequential or expected order. It is, therefore, recommended that you use the `orderby()` function to avoid the wrong data when executing functions such as `first()`. 
- YugabyteDB does not support columns that contain a `PRIMARY KEY` of type `user_defined_type`.

