## Chapter 2: Modeling and Visualizing Data

### Skill 2.2 Create calculated columns, calculated tables, and measures

#### DAX Operators

Logical Operators:
* `NOT` is negation.
* `IN` can search elements in a list.
* `&&` and `AND` both represent the logical AND.
* `||` and `OR` both represent the logical OR.
> @icon-info-circle See pg. 112 for complete list of DAX operators.

#### Using DAX functions in calculated columns

* Functions that return a 1x1 table are implicitly converted into Scalars.
* Unlike M functions, DAX functions *can* perform implicit type conversion.
> @icon-info-circle `LEN` should be used on text values or with a `FORMAT` nested inside of it.

* `FIND` and `SEARCH` are useful for finding substrings.
    > @icon-warning `FIND` is case-sensitive, where as `SEARCH` is not. <br>
    > @icon-warning `FIND` will propagate an error if no match is found. To avoid this, specifiy the 4th parameter in the function.
* `IFERROR` also exists in DAX and works the same way as in Excel.
> @icon-info-circle DAX indexes from 1, while M indexes from 0.
* To format code with a free tool, visit [The Dax Formatter Tool](http://www.daxformatter.com/).
* `SUBSTITUTE` also exists in DAX and works by replacing a specified substring with a replacement string.
    > @icon-warning `SUBSTITUTE` *is* case-sensitive. <br>

* `RELATED` works in a 1:Many relationship the following way: (Data flow is 1 @icon-arrow-right Many).

> @icon-info-circle See pg. 117 for a list of DAX mathematical functions. <br>
> @icon-info-circle See pg. 118 for a list of DAX datetime functions.

#### Using LOOKUPVALUE

* `LOOKUPVALUE` is invaluable for JOINING material based on multiple conditions. Think of it as a T-SQL JOIN with multiple ON statements.
* `LOOKUPVALUE`'s syntax is as follows:
    1. Column to retrieve values from.
    2. ( [Column to search] Scalar value ) pairs.
* `LOOKUPVALUE` returns an error if multiple matches exist for the given search conditions.
> @icon-info-circle To see other techniques for looking up values in DAX, [read this article](https://www.sqlbi.com/articles/lookup-multiple-values-in-dax/) by Marco Russo.

#### Grouping values

* `SWITCH`'s parameters are as follows:
    1. Expression to be searched for switching.
    2. ( Value to evaluate against the expression, Return value in the event of a match ) pairs.
    
    > @icon-info-circle `SWITCH` is great for replacing multiple nested `IF` statements.
    > @icon-info-cirlce Use the `SWITCH ( TRUE(), ... )` pattern for matching Boolean statements.
    
> @icon-warning Calculated columns *cannot* be sorted by **subsequent**, **dependent** calculated columns.
    
* Grouping can also take place in the user interface by selecting a field from the Fields pane on the right-hand side, and then selecting **Modeling** @icon-arrow-right **Groups** @icon-arrow-right **New Group**. This is where Binning can take place.
* Bins can be created in two ways through the **Groups** window:
    1. Size of bins (uses Min and Max scalar values to determine bin size).
    2. Number of bins (adjusts bin size by specifying total bin count).
* Grouping by **List group type** is also an option.
    * Single and/or multi-item lists can be specified by the user and renamed, if desired.
    * Additionally, ungrouped items can be thrown in an Other group by @icon-check the **Include Other group** checkbox.
* Calculated column icons:
    * Bins have two overlapping squares.
    * Numeric columns have a capital Sigma.
    * Other data type (text) columns have *fx*.
    
#### Using variables in calculated columns

* The names of variables cannot:
    * Include spaces.
    * Be reserved keywords.
* Variables can reference previously defined variables.

> @icon-warning Variables can only be accessed within the DAX *EXPRESSION* that they are defined in.

> @icon-warning Calculated columns consume RAM.

#### Evaulation Context

* Row Context
    : the current location across an iterative structure, either *materialized* (such as an actual data model table) or *virtualized* (such as in an X function).
* Filter Context
    : the combination of all filters from:
        * Visual, page, and report-level filters
        * Axes in a visual
        * Slicers
        * Rows and columns in the Matrix visual
        * etc.
    > @icon-info-circle Filter context *cannot* distinguish between identical rows in an iterative structure. <br>
    > @icon-info-circle Source columns used for sorting are also propagated throughout filter context.
* Both contexts always co-exist, and either of them can be empty at a given time.
* Basic aggregation functions ignore row context, such as `SUM`, `AVERAGE`, `COUNTROWS`, etc.
> @icon-warning By default, **row context** ignores any established relationships unless `RELATED` or `RELATEDTABLE` are explicitly used.
* Only filter context naturally utilizes existing relationships.
* `CALCULATE` transforms **row context** into **filter context** (`CALCULATE`: row context @icon-arrow-right filter context).
    * One required parameter: a DAX expression that works in filter context.
* Context Transition
    : Row context is transformed into filter context.
    : This means that *for each row*, **the table is filtered down to contain only rows whose values match those of the currently iterated row**.
    : Essentially,
        1. A TSQL WHERE is performed.
        2. The target column(s) of the result set are then aggregated with a primative aggregation function (e.g. `SUMX` @icon-arrow-right 1. Filter table rows. 2. Perform primative `SUM` aggregation).
> @icon-info-circle `RELATEDTABLE` is an alias for `CALCULATETABLE`. <br>
> @icon-info-circle To learn more context transition, read [Understanding Context Transition](https://www.sqlbi.com/articles/understanding-context-transition/) by Alberto Ferrari. Visit the various articles on [www.sqlbi.com](https://www.sqlbi.com/) to better understand all aspects of DAX evaulation context.

#### Circular dependencies in calculated columns

* Circular Dependencies
    * Two separate calculated columns using `CALCULATE` with identical DAX expressions create an implicit `Circular Dependency Error`. This is a byproduct of the context transition.
    * To fix the above problem, use Marco Russo's technique: implement a **dummy fact table**.
        1. **Home** @icon-arrow-right **External Data** @icon-arrow-right **Get Data** @icon-arrow-right **Blank Query**.
        2. Open the M Advanced Editor.
        3. Create an identically named target column in the dummy fact table.
            > For example: <br>
            > `#table ( type table [ Scale = Int64.Type ], {} )`
        4. Create a relationship between the target data table and the dummy fact table. Make sure that the relationship is *1:Many* where the target data table is on the *1 side* of the relationship.
    * Circular dependencies can occur in tables with no **Primary Key**.
    > @icon-info-circle When DAX detects a **primary key column**, context transition works differently: DAX knows it can rely on the column have unique row values, so it uses that column to filter the remaining columns in the table *without using values from the other columns*. <br>
    > In other words, the context transition is column @icon-arrow-right columns instead of row @icon-arrow-right columns.
    
    > @icon-info-circle For more details on circular dependencies in DAX, see the following two articles:
    > 1. https://www.sqlbi.com/articles/understanding-circular-dependencies
    > 2. https://www.sqlbi.com/articles/avoiding-circular-dependency-errors-in-dax
    
#### Calculated tables
    
* `CALCULATETABLE` only has one *required* parameter.
* Table expressions can be used inside formulas of calculated columns and measures, as well as by themselves to materialize calculated tables.
* **Duplicate** calculated tables can be created by setting them equal to the name of the original table (e.g. `Sales Duplicate Table = 'Sales'`).

#### FILTER

* The filter condition parameter in the `FILTER` function is evaluated in **row context** for each row of the table from the table parameter.
* However, `FILTER` **does *not* trigger context transition**. It is essentially a TSQL `WHERE` statement.
> @icon-warning When `FILTER` is used in context transition, it generates new row context. This means that *for each row in the base table (where the calculated column is being created)*, **the iteration is performed against the rows of the table in `FILTER`** and *not* the rows in the base table.
* It is possible to access the original row context from the new one with the `EARLIER` function. This action is simialr to an excel `SUMIF`.
* Context transition (by way of calculated tables) can be created inside of `FILTER`.
    
#### VARIABLES

* Variables behave like constants in that they are **constant** after being evaluated.
> @icon-info-circle To aid in DAX expressions where a table references its `EARLIER` rows, use a variable before the `CALCULATE`.
> @icon-info-circle It is important to remember that when you perform context transition in large tables, the operation is much slower that filtering column values. Therefore, instead of filtering a table and doing context transition, it is advisable to pre-calculate the results in a calculated column and then filter by the column.
    
#### ALL

* `ALL` can be used to create calculated tables.
* Parameter Behavior
    * When a *table* pararmeter is used, a copy of the original table is returned (including duplicate rows).
    * When a *one-column* parameter is used, the result will be a table containing **distinct** values.
    * When *multiple columns from the same table* are used, `ALL` returns a table of every *disticnt combination* of the columns.
* Parameter Restrictions
    * `ALL` cannot accept another function as an argument.
    * `ALL` cannot accept multiple columns from *different* tables as its parameter.
    > @icon-info-circle Recall that a calculation function such as `SUM` that is *not* wrapped in a `CALCULATE` is often equivalent to wrapping the same function in a `CALCULATE` and nesting `ALL` inside the filter parameter. <br>
* Virtual rows
    > @icon-warning Recall that, for a table on the *Many* side of a 1:Many relationship, if it contains rows not found in the table on the 1 side, DAX creates a virtual `BLANK` row in the 1 table that is invisible by default. <br>
    > @icon-warning `ALL` includes virtual rows. <br>
    * Virtual rows inside tables can be removed in 3 ways:
        1. Reference the original table in a new table by name.
        2. Filter out the blank row with `FILTER`.
        3. Use `ALLNOBLANKROW` function.
            > @icon-info-circle Recall that `ALLNOBLANKROW` does **not** filter out *true* blank rows. Otherwise, it works the same as `ALL`.
* `ALLEXCEPT`
    * `ALLEXCEPT` receives a table as the first parameter and a list of excluded columns in the following parameter.
    * `ALLEXCEPT` is also useful for calculating subtotals in calculated columns without the need of `EARLIER` or `var`.
        
#### CALCULATETABLE

* `RELATEDTABLE` is an alias for `CALCULATETABLE` only when one argument is used.
* Unlike `FILTER`, `CALCULATETABLE` can accept more than one filter parameter.
    * When multiple fitler parameters are used for `OR`, they must be on the **same column**.
> @icon-info-circle Recall that `CALCULATE` and `CALCULATETABLE` internally transform Boolean filter parameters into equivalent `CALCULATE(...,ALL()...)` tables.

> @icon-info-circle For more information on the filter arguments in `CALCULATE`, see "Fitler Arguments in `CALCULATE`" at https://www.sqlbi.com/articles/filter-arguments-in-calculate/.

#### VALUES and DISTINCT

* Both functions receive a column reference as a parameter and return tables that contain only *distinct* rows.
* Table Parameter
    * Both tables can receive a table as a parameter.
    > @icon-warning `VALUES` can only receive a physical table. `DISTINCT` can also receive a table expression.
* Both functions do **not** remove filters from their table parameters.
> @icon-warning `VALUES` includes virtual blank rows. `DISTINCT` does **not** include virtual blank rows.

#### SUMMARIZE and SUMMARIZECOLUMNS

* `SUMMARIZE`'s two parameters:
    1. A table to summarize.
    2. At least one column to group by.
* Nonsurjective Table Relationships:
    > @icon-info-circle If referencing the Many table in a 1:Many relationship where the Many table contains values missing from the 1 table, then `SUMMARIZE` and `SUMMARIZECOLUMNS` *will* include the virtual blank row. <br>
    However, if summarizing the 1 table, the virtual blank row *will not* be included. <br>
    > @icon-info-circle However, `SUMMARIZECOLUMNS` *will* include the virtual blank row, regardless of which table.
* The *columns to group by* parameter is optional if at least one new named column and expression is included in its place.
* `SUMMARIZE` does *not* grant access to the row context of the table being summarized. Instead, `SUMMARIZE` divides the table into parts, grouping them by the selected columns, **where each part of the original table retains its *filter context***. <br> In other words, filter context is automatically present with `SUMMARIZE`.
* `SUMMARIZECOLUMNS`, unlike `SUMMARIZE`, does *not* require a table parameter. Instead, the columns themselves are simply listed.
> @icon-info-cirlce Read "All the secrets of `SUMMARIZE`" by Marco Russo and Alberto Ferrari at https://www.sqlbi.com/articles/introducing-summarizecolumns.

#### ADDCOLUMNS and SELECTCOLUMNS

* `ADDCOLUMNS` creates row context in the proecss of it adding new columns.
* The new columns are known as extension columns.
* `ADDCOLUMNS` takes three *required* parameters:
    1. Base table to add columns to.
    2. New column name.
    3. New column expression.
* `ADDCOLUMNS` Functionality:
    > @icon-warning Unlike in `SUMMARIZE`, `ADDCOLUMNS` requires context transition that *must* be initialized through a nested `CALCULATE`.
    * After creating context transition by way of `CALCULATE`, you can reference columns in the table to which you are adding more columns to.
* `SELECTCOLUMNS` is very similar to `ADDCOLUMNS`, except that the original columns are not kept.
    > @icon-warning `SELECTCOLUMNS` does *not* remove duplicate rows.
    * The results of `SELECTCOLUMNS` can be grouped by extension columns referencing the column(s) inside of `SELECTCOLUMNS` within `SUMMARIZE`.
    * In other words, `SELECTCOLUMNS` can be nested inside of a `SUMMARIZE`, and the second parameter of `SUMMARIZE` can reference the column(s) from `SELECTCOLUMNS`.
    > @icon-info-circle Recall that when parameter two of `SUMMARIZE` references parameter one of `SUMMARIZE`, the column(s) being referenced in parameter two must be referenced *without a table name*.
    > @icon-info-circle `SELECTCOLUMNS` can be used to rename columns. See https://blog.crossjoin.co.uk/2015/06/01/using-selectcolumns-to-alias-columns-in-dax/.
    
#### TOPN

* `TOPN` has *three* **required** parameters and *one* **optional** parameter:
    1. N (a number).
    2. Table expression.
    3. Expression to order rows by.
    4. *(optional)* ASC/DESC (the default is DESC).
> @icon-info-circle `TOPN` uses row context, so `CALCLATE` is required inside of `TOPN` to force context transition. <br>
> @icon-warning In the case of a **tie**, more rows are returned than expected. <br>
> @icon-info-circle With `TOPN`, it is also possible to order by more than one expression: in this case, expressions and orders come in pairs after the first expression and order pair.

#### CROSSJOIN, GENERATE, and GENERATEALL

* `CROSSJOIN` creates a **Cartesian product** between two or more tables.
    * All columns in the resulting tables must be unique.
    * Consequently, taking the Cartesian product of a table with itself requires renaming one of the columns of the tables in advance (use `SELECTCOLUMNS`, see pg. 153).
    > @icon-warning There is no row context or context transition in `CROSSJOIN` for the second or subsequent tables. Consequently, to reference the current row of the first table in `CROSSJOIN`, use `GENERATE`. However, `CALCULATE` can be used as a nested function for the second (or subsequent) table expression parameters.
* `GENERATE` always receives two table expressions as parameters.
* `GENERATEALL` works the same way as `GENERATE` except that it returns all possible combinations.

#### GENERATESERIES

* `GENERATESERIES` generates a table with one column, called Value, containing a list of numbers with predefined increment.
    > @icon-info-circle These values need not exist in the DAX Data Model.
* `GENERATESERIES` parameters:
    1. Start value.
    2. End value.
    3. *(optional)* Increment value (the default is 1).
    > @icon-warning If Start value > End value, the result is a zero-row table.
* `GENERATESERIES` automatically detects the data type being generated. Data types can also be explicitly specified in `GENERATESERIES`.
* `GENERATESERIES` can be used to generate a list of datetime values.
    > @icon-info-circle To use hourly increments, utilize the `TIME` function.
* As an additional note, `GENERATESERIES` can also generate a list of letters by combining `SELECTCOLUMNS` and `UNICHAR`.
* `UNICHAR` takes a positive integer as its only parameter and returns a Unicode character.

#### CALENDAR and CALENDERAUTO

* `CALENDER` generates a table of datetime data type.
* `CALENDERAUTO` searches all date and datetime columns in the entire DAX Data Model and finds the minimum date and maximum date. It then takes **1 January of the minimum date's year** through **31 Decemeber of the maximum date's year**.
    * `CALENDARAUTO` has one optional third parameter, which is the fiscal year end month number.
> @icon-info-circle `MIN` and `MAX` allow for the comparison of two scalar values.

#### ROW

* `ROW` allows for the creation of one-row tables containing multiple columns.
    * `ROW`'s parameters come in pairs:
        1. Column name.
        2. DAX expression.
> @icon-info-circle `ROW` can be useful when adding a new row to a table using `UNION`.

#### UNION

* `UNION` combines tables vertically.
* If both tables have rows that are identical in the other table, the duplicate rows are retained in the `UNION`.
* `UNION`'s output table will have the same column names as the first table parameter.
* `UNION` can be used to create common dimensions from several different tables.

> @icon-warning If aligned columns in a `UNION` have differing data types, they will be combined in accordance with DAX data type coercion (e.g. type Text overrules type Whole Number). <br>

> @icon-info-circle Power Query's `APPEND` combines tables vertically. It is essentially a `UNION`. <br>

> @icon-info-circle Recall that tables can be hidden in the DAX Data Model.

#### INTERSECT

* Table expressions are accepted.
* Both table parameters must have the same number of columns, and `INTERSECT` combines tables based on the position of those columns.
* The result table retains the column names from the *first* table parameter.
> @icon-warning `INTERSECT` will retain duplicate matches from the *first* table parameter. This is why, technically, order matters in `INTERSECT`.

#### EXCEPT

* Table expressions are accepted.
* `EXCEPT` takes two tables as arguments and outputs all rows that exist in the first table parameter but not the second table parameter.
* The result table retains the column names from the *first* table parameter.
> @icon-warning `EXCEPT` will retain duplicate matches from the *first* table parameter. This is why, technically, order matters in `EXCEPT`.

#### NATURALINNERJOIN

* Table expressions are accepted.
* `NATURALINNERJOIN` is quite similar to Power Query's `Merge` function: it receives two tables as arguments and joins them based on common column names.
* `NATURALINNERJOIN` joins two tables and outputs a table that has the same ***values*** present in join columns of both tables.
> @icon-info-circle If the JOIN columns have different names in each table, then `NATURALINNERJOIN` will add each name as a separate column in the result table.
* Order only matters in `NATURALINNERJOIN` when considering the order of the JOIN columns.
* `NATURALINNERJOIN` can also join physical tables that have a relationship between them.
    * In this case, order may not matter in the `JOIN`.
* Shared column names must not exist in both tables in order to use `NATURALINNERJOIN`. Otherwise, the function will produce an error.
> @icon-info-circle In general, all column names in *materialized* DAX tables should be unique. However, *virtual* tables are allowed to share column names in some cases.
* The ability to join tables with common column names can be useful when passing filters to `CALCULATE` and `CALCULATETABLE`.

#### NATURALLEFTOUTERJOIN

* `NATURALLEFTOUTERJOIN` returns:
    1. A table with all rows from the first table **and** 
    2. Extra columns from the second table where values in the join columns of the second (right) table are present in the join columns of the first (left) table.
* `NATURALLEFTOUTERJOIN` can also join physical tables that have a relationship between them.

#### DATATABLE

* `DATATABLE` allows you to create calculated tables with data that you enter manually.
* `DATATABLE` as a minimum of three parameters:
    1. Column name.
    2. Data type (BOOLEAN, CURRENCY, DATETIME, DOUBLE, INTEGER, STRING).
    3. List of values (the list itself is in {} **and** the values themselves are in {}).
> @icon-info-circle Think of a value in the third parameter of `DATATABLE` as an entire row. Thus, *the row itself* is wrapped in {}, not each column value of a row (e.g. {1, "String", TRUE}). <br>

> @icon-warning The values in the third parameter must be **constants**. DAX expressions are *not* allowed.
* An actual table can be created within `DATATABLE` by listing column names and data types in pairs before the third parameter (the list of values).
> @icon-info-circle Think of `DATATABLE` similar to `SUMMARIZE` over a manually created table.

* Anonymous tables can also be defined in DAX by doing the following:
    1. Use one set of {} for the entire table.
    2. Use a comma separated list of () for the values. If the anonymous table is a multicolumn table, then the column values should be listed in each () set.
    * In anonymous tables, DAX will name the columns **Value1**, **Value2**, **Value3**, etc.
> @icon-info-circle To read more about `DATATABLE`, see Marco Russo's article [Create Static Tables in DAX Using the DATATABLE Function](https://www.sqlbi.com/articles/create-static-tables-in-dax-using-the-datatable-function/).

#### Using variables in calculated tables

> @icon-warning Remember that variables are only evaluated **once** in the context in which they are defined. Because of this, they are **USELESS** in iterator functions like `CALCULATE` and `CALCULATETABLE`. In other words, variables are **immutable** after the **first evaulation**.

> @icon-info-circle To see more information about using `GENERATE`/`ROW` instead of `ADDCOLUMNS`, see [Using GENERATE and ROW instead of ADDCOLUMNS in DAX](https://www.sqlbi.com/articles/using-generate-and-row-instead-of-addcolumns-in-dax/).

#### Measures

> @icon-info-circle Measures aggregate columns and tables, and they **always** work in filter context. Consequently, *by default*, there is no concept of the **current row** in measures. You cannot create a measure in the same way that you can a calculated column, iterating row by row through a table.

* The basic aggregation functions, such as `SUM`, `AVERAGE`, `MIN`, `MAX`, etc. take a column reference as their once parameter. They are essentially syntatic sugar for their iterator foundations.
* Please note that the following two DAX expressions below are equivalent:
    * `Total Net Profit = SUM ( Example[Net Profit] )`
    * `Total Net Profit = SUMX ( Example, Example[Net Profit] )`
* For iterator functions, the internal calculations of the DAX expression are evaluated first, and then the set of those calculations are aggregated by the iterator function.
* Since iterators generate row context, all row functions (such as `RELATED`) can be used in them.

#### Measures vs. calculated columns

* Calculated columns are computed when:
    * They are first defined.
    * A data refresh occurs.
* Calculated columns are materialized; therefore, they consume RAM and disk space.
* On the other hand, measures are calculated at query time. Every interaction with a visual or slicer triggers measures to recalculate themselves. Therefore, measures consume CPU resources.

> @icon-warning Recall that measures *cannot* be placed into slicers.

#### Counting values in DAX

* There are several count functions within DAX:
    * `COUNT`
    * `COUNTA`
    * `COUNTAX`
    * `COUNTBLANK`
    * `COUNTROWS`
    * `COUNTX`
    * `DISTINCTCOUNT`
> @icon-info-circle Recall that:
> * `COUNTROWS` takes a *table expression* as its parameter.
> * `COUNT` takes a column reference as its parameter and counts the number of *non-blank values*.
> * `COUNT` cannot handle Boolean values.
> * `COUNTA` counts the number of non-blank values *regardless of their datatype*.

* Pg. 177

