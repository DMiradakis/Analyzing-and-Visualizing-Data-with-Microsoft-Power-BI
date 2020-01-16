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

* Pg. 126