---
title: SQLAlchemy
description: Guide for using SQLAlchemy with QuestDB
---

[SQLAlchemy](https://www.sqlalchemy.org/) is an open-source SQL toolkit and ORM
library for Python. It provides a high-level API for communicating with
relational databases, including schema creation and modification, an SQL
expression language, and database connection management. The ORM layer abstracts
away the complexities of the database, allowing developers to work with Python
objects instead of raw SQL statements.

QuestDB implements a dialect for SQLAlchemy using the
[QuestDB Connect](https://github.com/questdb/questdb-connect).

## Prerequisites

- Python from 3.8.x to 3.10.x
- Psycopg2
- SQLAlchemy
- A QuestDB instance

## Installation

You can install this package using `pip`:

```shell
pip install questdb-connect
```

## Example usage

```python
import datetime
import os

os.environ.setdefault('SQLALCHEMY_SILENCE_UBER_WARNING', '1')

import questdb_connect.dialect as qdbc
from sqlalchemy import Column, MetaData, create_engine, insert
from sqlalchemy.orm import declarative_base

Base = declarative_base(metadata=MetaData())


class Signal(Base):
    __tablename__ = 'signal'
    __table_args__ = (qdbc.QDBTableEngine('signal', 'ts', qdbc.PartitionBy.HOUR, is_wal=True), )
    source = Column(qdbc.Symbol)
    value = Column(qdbc.Double)
    ts = Column(qdbc.Timestamp, primary_key=True)


def main():
    engine = create_engine('questdb://localhost:8812/main')
    ## Establishing connectivity
    try:
        Base.metadata.create_all(engine)
        ## Setting up metadata with table objects

        with engine.connect() as conn:
            conn.execute(insert(Signal).values(
                source='coconut',
                value=16.88993244,
                ts=datetime.datetime.utcnow()
            ))
        ## The insert() SQL expression construct

    finally:
        if engine:
            engine.dispose()


if __name__ == '__main__':
    main()
```

## See also

- The
  [SQLAlchemy tutorial](https://docs.sqlalchemy.org/en/14/tutorial/index.html)
- The [QuestDB Connect](https://pypi.org/project/questdb-connect/) project
