<properties
	pageTitle="PowerApps: Errors function"
	description="Reference information for the Errors function in PowerApps, including syntax and examples"
	services=""
	suite="powerapps"
	documentationCenter="na"
	authors="gregli-msft"
	manager="dwrede"
	editor=""
	tags=""/>

<tags
   ms.service="powerapps"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="11/11/2015"
   ms.author="gregli"/>

# Errors function in PowerApps #

Provides error information for previous changes to a [data source](working-wtih-data-sources.md).

## Overview ##

Errors can happen when making changes to a record of a data source.  There are many different causes possible, including network outages, inadequate permissions, edit conflicts, etc.  

**[Patch](function-patch.md)** and other data functions do not directly return errors, instead they return the result of their operation.  After execution of a data function, you can use the **Errors** function to obtain the details of any errors.  You can check for the existence of errors with **[IsEmpty](function-isempty)( Errors ( ... ) )**.

You can avoid some errors before they happen by using the **[Validate](function-validate.md)** and **[DataSourceInfo](function-datasourceinfo.md)** functions.  See [working with data sources](working-with-data-sources.md) for more suggestions on working with and avoiding errors.

## Description ##

The **Errors** function returns a table of errors with the following columns:

- **Record**.  The [record](working-with-tables.md) in the data source that had the error.  If the error occurred during the creation of a new record, this column will be *blank*.

- **Column**.  The [column](working-with-tables.md) that caused the error, if the error can be attributed to a single column.  If not, this will be *blank*.

- **Message**.  Description of the error.  This error string can be displayed for the end user.  Be aware that this message may be generated by the data source and could be long and contain raw column names that may not have any meaning to the user. 

- **Error**.  An error code that can be used in formulas to help resolve the error:

| ErrorKind | Description |
|------------|-------------|
| ErrorKind!Conflict | There was another change to the same record, resulting in a change conflict.  Use **[Revert](function-refresh.md)** to reload the record and try the change again. |
| ErrorKind!ConstraintViolation | One or more constraints have been violated. |
| ErrorKind!CreatePermission | An attempt was made to create a record, and the current user does not have permission to create records. |
| ErrorKind!DeletePermission | An attempt was made to delete a record, and the current user does not have permission to delete records. |
| ErrorKind!EditPermission | An attempt was made to edit a record, and the current user does not have permission to edit records. |
| ErrorKind!GeneratedValue | An attempt was made to change a column that is automatically generated by the data source. |
| ErrorKind!MissingRequired | The value for a required column is missing from the record. |
| ErrorKind!None | The error is of an unknown kind. |
| ErrorKind!NotFound | An attempt was made to edit or delete a record, but the record could not be found.  The record may have been changed by another user. |
| ErrorKind!ReadOnlyValue | An attempt was made to change a column that is read only. |
| ErrorKind!Sync | There was an error while synchronizing the change with the data service. |

Errors can be returned for the entire data source, or for only a selected row by providing the *Record* argument to the function.  

In some cases, **Patch** or another data function may return a *blank* value, if for example, a record could not be created.  You can pass *blank* to **Errors** and it will return appropriate error information in these cases.  Subsequent use of data functions on the same data source will clear this error information. 

If there are no errors, the table returned by **Errors** will be empty and can be tested with the **IsEmpty** function.

## Syntax ##

**Errors**( *DataSource* [, *Record* ] )

- *DataSource* – Required.  Errors will be returned for this data source.

- *Record* – Optional.  Errors for only a specific record will be returned.  If this argument is not provided, errors for the entire data source are returned.

## Examples ##

### Step by Step ###

For this example, we'll be working with the **IceCream** data source:

![](media/function-errors/icecream.png)

Through PowerApps, a user loads the Chocolate record into a data entry form and changes the value of **Quantity** to 90.  The record to be worked with is placed in the context variable **EditRecord**:

- **UpdateContext( { EditRecord: First( Filter( IceCream, Flavor = "Chocoalte" ) ) } )**

To make this change in the data source, the **Patch** function is used:

- **Patch( IceCream, EditRecord, Gallery!Updates )**

where **Gallery!Updates** evaluates to **{ Quanitty: 90 }**, since only the **Quantity** has been modified.

Unfortunately, just before the **Patch** function was invoked, somebody else modifies the **Quantity** for Chocolate to 80.  PowerApps will detect this and not allow the conflicting change to occur.  You can check for this situation with the formula:

- **IsEmpty( Errors( IceCream, EditRecord ) )**

which returns **false**, because the **Errors** function returned the following table:

| Record | Column | Message | Error |
|--------|--------|---------|-------|
| { Flavor: "Chocolate", Quantity: 100 } | *blank* | "The record you are trying to modify has been modified by another user.  Please reload the record and try again." | ErrorKind!Conflict |

You can place a label on the form to show this error to the user with the formula

- **Label!Text = Errors( IceCream, EditRecord )!Message**

You can also place a "Reload" button on the form, so that the user can efficiently resolve the conflict

- **ReloadButton!OnSelect = Revert( IceCream, EditRecord )**
- **ReloadButton!Visible = !IsEmpty( Lookup( Errors( IceCream, EditRecord ), Error = ErrorKind!Conflict ) )**





