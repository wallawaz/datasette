The Datasette JSON API
======================

Datasette provides a JSON API for your SQLite databases. Anything you can do
through the Datasette user interface can also be accessed as JSON via the API.

To access the API for a page, either click on the ``.json`` link on that page or
edit the URL and add a ``.json`` extension to it.

If you started Datasette with the ``--cors`` option, each JSON endpoint will be
served with the following additional HTTP header::

    Access-Control-Allow-Origin: *

This means JavaScript running on any domain will be able to make cross-origin
requests to fetch the data.

If you start Datasette without the ``--cors`` option only JavaScript running on
the same domain as Datasette will be able to access the API.

Different shapes
----------------

The default JSON representation of data from a SQLite table or custom query
looks like this::

    {
        "database": "sf-trees",
        "table": "qSpecies",
        "columns": [
            "id",
            "value"
        ],
        "rows": [
            [
                1,
                "Myoporum laetum :: Myoporum"
            ],
            [
                2,
                "Metrosideros excelsa :: New Zealand Xmas Tree"
            ],
            [
                3,
                "Pinus radiata :: Monterey Pine"
            ]
        ],
        "truncated": false,
        "next": "100",
        "next_url": "http://127.0.0.1:8001/sf-trees-02c8ef1/qSpecies.json?_next=100",
        "query_ms": 1.9571781158447266
    }

The ``columns`` key lists the columns that are being returned, and the ``rows``
key then returns a list of lists, each one representing a row. The order of the
values in each row corresponds to the columns.

The ``_shape`` parameter can be used to access alternative formats for the
``rows`` key which may be more convenient for your application. There are three
options:

* ``?_shape=arrays`` - ``"rows"`` is the default option, shown above
* ``?_shape=objects`` - ``"rows"`` is a list of JSON key/value objects
* ``?_shape=array`` - the entire response is an array of objects
* ``?_shape=object`` - the entire response is a JSON object keyed using the primary keys of the rows

``objects`` looks like this::

    {
        "database": "sf-trees",
        ...
        "rows": [
            {
                "id": 1,
                "value": "Myoporum laetum :: Myoporum"
            },
            {
                "id": 2,
                "value": "Metrosideros excelsa :: New Zealand Xmas Tree"
            },
            {
                "id": 3,
                "value": "Pinus radiata :: Monterey Pine"
            }
        ]
    }

``array`` looks like this::

    [
        {
            "id": 1,
            "value": "Myoporum laetum :: Myoporum"
        },
        {
            "id": 2,
            "value": "Metrosideros excelsa :: New Zealand Xmas Tree"
        },
        {
            "id": 3,
            "value": "Pinus radiata :: Monterey Pine"
        }
    ]

``object`` looks like this::

    {
        "1": {
            "id": 1,
            "value": "Myoporum laetum :: Myoporum"
        },
        "2": {
            "id": 2,
            "value": "Metrosideros excelsa :: New Zealand Xmas Tree"
        },
        "3": {
            "id": 3,
            "value": "Pinus radiata :: Monterey Pine"
        }
    ]

The ``object`` shape is only available for queries against tables - custom SQL
queries and views do not have an obvious primary key so cannot be returned using
this format.

The ``object`` keys are always strings. If your table has a compound primary
key, the ``object`` keys will be a comma-separated string.

Special table arguments
-----------------------

The Datasette table view takes a number of special querystring arguments:

``?_size=1000`` or ``?_size=max``
    Sets a custom page size. This cannot exceed the ``max_returned_rows`` limit
    passed to ``datasette serve``. Use ``max`` to get ``max_returned_rows``.

``?_sort=COLUMN``
    Sorts the results by the specified column.

``?_sort_desc=COLUMN``
    Sorts the results by the specified column in descending order.

``?_search=keywords``
    For SQLite tables that have been configured for
    `full-text search <https://www.sqlite.org/fts3.html>`_ executes a search
    with the provided keywords.

``?_search_COLUMN=keywords``
    Like ``_search=`` but allows you to specify the column to be searched, as
    opposed to searching all columns that have been indexed by FTS.

``?_group_count=COLUMN``
    Executes a SQL query that returns a count of the number of rows matching
    each unique value in that column, with the most common ordered first.

``?_group_count=COLUMN1&_group_count=column2``
    You can pass multiple ``_group_count`` columns to return counts against
    unique combinations of those columns.

``?_timelimit=MS``
    Sets a custom time limit for the query in ms. You can use this for optimistic
    queries where you would like Datasette to give up if the query takes too
    long, for example if you want to implement autocomplete search but only if
    it can be executed in less than 10ms.

``?_next=TOKEN``
    Pagination by continuation token - pass the token that was returned in the
    ``"next"`` property by the previous page.
