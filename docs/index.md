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

* @icon-refresh Pg. 132
