
.. module:: datatest

###################
Developer Interface
###################


************
DataTestCase
************

This class inherits from
`unittest.TestCase <http://docs.python.org/library/unittest.html#unittest.TestCase>`_
and adds additional properties and methods to help with testing data.
In addition to the new functionality, the familiar ``TestCase`` methods
(like ``setUp``, ``assertEqual``, etc.) are still available.

.. autoclass:: datatest.DataTestCase

    .. autoattribute:: subjectData
    .. autoattribute:: referenceData


    .. _assert-methods:

    +-----------------------------------------------------------------+------------------------------------------+
    | Method                                                          | Checks that                              |
    +=================================================================+==========================================+
    | :meth:`assertDataColumns(required=None)                         | column names match *required*            |
    | <datatest.DataTestCase.assertDataColumns>`                      |                                          |
    +-----------------------------------------------------------------+------------------------------------------+
    | :meth:`assertDataSet(column, required=None)                     | *column* contains *required* values      |
    | <datatest.DataTestCase.assertDataSet>`                          |                                          |
    +-----------------------------------------------------------------+------------------------------------------+
    | :meth:`assertDataSum(column, group_by, required=None)           | sums of *column*, grouped by *group_by*, |
    | <datatest.DataTestCase.assertDataSum>`                          | match *required* values dict             |
    +-----------------------------------------------------------------+------------------------------------------+
    | :meth:`assertDataCount(column, group_by, required=None)         | row counts of *column*, grouped by       |
    | <datatest.DataTestCase.assertDataCount>`                        | *group_by*, match *required* values dict |
    +-----------------------------------------------------------------+------------------------------------------+
    | :meth:`assertDataRegex(column, required)                        | *required*.search(val) for each val in   |
    | <datatest.DataTestCase.assertDataRegex>`                        | *column*                                 |
    +-----------------------------------------------------------------+------------------------------------------+
    | :meth:`assertDataNotRegex(column, required)                     | not *required*.search(val) for each val  |
    | <datatest.DataTestCase.assertDataNotRegex>`                     | in *column*                              |
    +-----------------------------------------------------------------+------------------------------------------+

    .. automethod:: assertDataColumns
    .. automethod:: assertDataSet
    .. automethod:: assertDataSum
    .. automethod:: assertDataCount
    .. automethod:: assertDataRegex
    .. automethod:: assertDataNotRegex


    +-----------------------------------------------------------------+------------------------------------------+
    | Context Manager                                                 | Allows                                   |
    +=================================================================+==========================================+
    | :meth:`allowOnly(differences)                                   | only specified *differences*             |
    | <datatest.DataTestCase.allowOnly>`                              |                                          |
    +-----------------------------------------------------------------+------------------------------------------+
    | :meth:`allowAny(number)                                         | given *number* of differences of any     |
    | <datatest.DataTestCase.allowAny>`                               | class                                    |
    +-----------------------------------------------------------------+------------------------------------------+
    | :meth:`allowAny(**filter_by)                                    | unlimited number of differences of any   |
    | <datatest.DataTestCase.allowAny>`                               | class that match given                   |
    |                                                                 | :ref:`keyword filters <filter-by>`       |
    +-----------------------------------------------------------------+------------------------------------------+
    | :meth:`allowExtra(number=None)                                  | given *number* of :class:`Extra          |
    | <datatest.DataTestCase.allowExtra>`                             | <datatest.Extra>` differences or         |
    |                                                                 | unlimited number if ``None``             |
    +-----------------------------------------------------------------+------------------------------------------+
    | :meth:`allowMissing(number=None)                                | given *number* of :class:`Missing        |
    | <datatest.DataTestCase.allowMissing>`                           | <datatest.Missing>` differences or       |
    |                                                                 | unlimited number if ``None``             |
    +-----------------------------------------------------------------+------------------------------------------+
    | :meth:`allowDeviation(tolerance)                                | :class:`Deviations <datatest.Deviation>` |
    | <datatest.DataTestCase.allowDeviation>`                         | of plus or minus given *tolerance*       |
    +-----------------------------------------------------------------+------------------------------------------+
    | :meth:`allowPercentDeviation(tolerance)                         | :class:`Deviations <datatest.Deviation>` |
    | <datatest.DataTestCase.allowPercentDeviation>`                  | of plus or minus given *tolerance*       |
    |                                                                 | percentage                               |
    +-----------------------------------------------------------------+------------------------------------------+

    .. automethod:: allowOnly

    .. automethod:: allowAny

    .. automethod:: allowMissing

    .. automethod:: allowExtra

    .. method:: allowDeviation(tolerance, /, msg=None, **filter_by)
                allowDeviation(lower, upper, msg=None, **filter_by)

        Context manager to allow for deviations from required
        numeric values without triggering a test failure.

        Allowing deviations of plus-or-minus a given *tolerance*::

            with self.allowDeviation(5):  # tolerance of +/- 5
                self.assertDataSum('column2', group_by=['column1'])

        Specifying different *lower* and *upper* bounds::

            with self.allowDeviation(-2, 3):  # tolerance from -2 to +3
                self.assertDataSum('column2', group_by=['column1'])

        All deviations within the accepted tolerance range are
        suppressed but those that exceed the range will trigger
        a test failure.

        .. note:: The "``/``" in this method's signature means that the
                  preceding argument (*tolerance*) is a positional-only
                  parameter---it cannot be specified using keyword syntax.

    .. automethod:: allowPercentDeviation


.. _filter-by:

Filter by Keywords (using \*\*filter_by)
========================================

Many datatest methods support optional keyword arguments to quickly filter the
data being tested.  For example, adding ``state='Ohio'`` to a data assertion
would limit the test to those records where the "state" column contains the
value "Ohio"::

    self.assertDataSet('postal_code', state='Ohio')

Multiple keywords can be used to further specify the data being tested.  The
keyword filter ``state='Ohio', city='Columbus'`` limits the test to records
where the "state" column contains the value "Ohio" *and* the "city" column
contains the value "Columbus"::

    self.assertDataSet('postal_code', state='Ohio', city='Columbus')

Keyword arguments can also contain multiple values.  Using ``state=['Indiana',
'Ohio']`` limits the test to records where the "state" column contains the
value "Indiana" *or* the value "Ohio"::

    self.assertDataSet('postal_code', state=['Indiana', 'Ohio'])


************
Data Sources
************

Data source objects are used to access data in various formats.

+---------------------------------------------+-----------------------------------------+
| Class                                       | Loads                                   |
+=============================================+=========================================+
| :class:`CsvSource(file)                     | CSV from path or file-like object       |
| <datatest.CsvSource>`                       | *file*                                  |
+---------------------------------------------+-----------------------------------------+
| :class:`SqliteSource(connection, table)     | *table* from SQLite *connection*        |
| <datatest.SqliteSource>`                    |                                         |
+---------------------------------------------+-----------------------------------------+
| :class:`MultiSource(*sources)               | multiple data *sources* which can be    |
| <datatest.MultiSource>`                     | treated as single data source           |
+---------------------------------------------+-----------------------------------------+
| :class:`ExcelSource(path, worksheet=None)   | Excel *worksheet* from XLSX or XLS      |
| <datatest.ExcelSource>`                     | *path*, defaults to the first worksheet |
|                                             | if ``None`` (requires `xlrd             |
|                                             | <http://pypi.python.org/pypi/xlrd>`_)   |
+---------------------------------------------+-----------------------------------------+
| :class:`PandasSource(df)                    | pandas DataFrame *df* (requires `pandas |
| <datatest.PandasSource>`                    | <http://pypi.python.org/pypi/pandas>`_) |
+---------------------------------------------+-----------------------------------------+


CsvSource
=========
.. autoclass:: datatest.CsvSource
   :members: create_index


SqliteSource
============
.. autoclass:: datatest.SqliteSource
   :members: create_index, from_records


MultiSource
===========
.. autoclass:: datatest.MultiSource

    Data is aligned by column name and missing values are filled with empty
    strings:

        .. image:: _static/multisource.*

--------

.. _extra-sources:

If you have the appropriate, optional dependencies installed, datatest
provides a variety of other data sources:


ExcelSource
===========
.. autoclass:: datatest.ExcelSource


PandasSource
============
.. autoclass:: datatest.PandasSource
   :members: from_records


*******************
Data Source Methods
*******************

Data sources implement a common set of methods which are used by
DataTestCase to access data and report meaningful failure messages.
Typically, these methods are used indirectly via DataTestCase but it is
also possible to call them directly:

.. autoclass:: datatest.BaseSource

    .. method:: columns()

        Return a list or tuple of column names.

    .. method:: distinct(column, **filter_by)

        Return ResultSet containing distinct *column* values.

    .. method:: sum(column, group_by=None, **filter_by)

        Returns sum of *column* grouped by *group_by* as ResultMapping.

    .. method:: count(group_by=None, **filter_by)

        Returns count of rows grouped by *group_by* as ResultMapping.

    .. method:: reduce(function, column, group_by=None, initializer=None, **filter_by)

        Apply *function* of two arguments cumulatively to the values in *column*,
        from left to right, so as to reduce the iterable to a single value.  If
        *column* is a string, the values are passed to *function* unchanged.  But
        if *column* is, itself, a function, it should accept a single dict-row and
        return a single value.  If *group_by* is omitted, the raw result is
        returned, otherwise returns a ResultMapping object.


***************
Results Objects
***************

Querying a data source with various methods will return a ResultSet or a
ResultMapping.


ResultSet
=========
.. autoclass:: datatest.ResultSet
    :members: make_rows, compare


ResultMapping
=============
.. autoclass:: datatest.ResultMapping
    :members: make_rows, compare

    .. py:attribute:: key_names

        Column names for result keys.


**********************
Errors and Differences
**********************

.. autoclass:: datatest.DataAssertionError
   :members:

.. autoclass:: datatest.Extra
   :members:

.. autoclass:: datatest.Missing
   :members:

.. autoclass:: datatest.Invalid
   :members:

.. autoclass:: datatest.Deviation
   :members:


*******************
Test Runner Program
*******************

.. autoclass:: datatest.DataTestRunner
           :members:
           :inherited-members:

.. autoclass:: datatest.DataTestProgram(module='__main__', defaultTest=None, argv=None, testRunner=datatest.DataTestRunner, testLoader=unittest.TestLoader, exit=True, verbosity=1, failfast=None, catchbreak=None, buffer=None, warnings=None)
   :members:
   :inherited-members:

|

.. autoclass:: datatest.main
   :members:
   :inherited-members: