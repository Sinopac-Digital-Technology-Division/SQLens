# SQLens

SQLens is a Python library designed to extract data lineage from SQL statements, prioritizing support for all dialects. Traditional token-based parsers, like [sqlglot](https://github.com/tobymao/sqlglot) and [sql-metadata](https://github.com/macbre/sql-metadata), often encounter parsing issues when a query includes dialect-specific keywords. Instead of parsing the query statement, SQLens extracts the necessary information from the database execution plan. Therefore, regardless of the dialect or version you use, as long as the database can process and output the execution plan, SQLens will function correctly.

## Examples

### TSQL for SQL Server

A simple code snippet to get all `Relation` objects from the input query statement.

python

```python
from sqlens.dialects import tsql
from sqlens.utils import Connection

conn = Connection(
  "127.0.0.1",  # DB IP address
  "my_username",
  "my_password",
  1433,
)
query = "SELECT col_a, col_b FROM MyDB.dbo.MyTable"
relations = tsql.TSQL(conn).get_relation(query)
# Output list of referenced columns as namedtuple
# {
#     Relation(host='127.0.0.1', database='MyDB', schema='dbo', table='MyTable', column='col_a'),
#     Relation(host='127.0.0.1', database='MyDB', schema='dbo', table='MyTable', column='col_b'),
# }

```

### Integrate into Existing Analysis Workflow

SQLens is designed to fully support [OpenLineage](https://openlineage.io/docs/) and make integrating with existing workflows as easy as possible. Developers can create a [SQLAlchemy engine](https://docs.sqlalchemy.org/en/20/core/engines.html) with hook functions attached by `sqlens.create_engine`. An OpenLineage event can be easily recorded by passing the engine connection into `pandas.read_sql`.

```python
from sqlens import create_engine
import pandas as pd

hooked_engine = create_engine(
    db_url="mssql+pymssql://username:password@my-db.com:1433",
)

with hooked_engine.connect() as conn:
    # OpenLineage event is sent implicitly when the SQLAlchemy
    # after_cursor_execute event is triggered.
    pd.read_sql(
        "SELECT col_a, col_b FROM MyDB.dbo.MyTable",
        conn,
    )

```

## Keep In Touch

Currently, SQLens is under active development. The interface and implementation are subject to change. Please feel free to open an issue if you have any feature requests or implementation suggestions.
