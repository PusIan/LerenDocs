# What is it?
Leren is a simple Excel-based reporting engine. It helps to make a report in excel in fast-and-easy-to-make manner. You have to describe data sources right inside a cell of excel worksheet. It supports multi-level data structures and poor formatting options.

# Quick start!
1. Make an empty excel file and name it "file1.xlsx"
2. Type in cell A1 of worksheet: `{COLL=Apples/Green;WIDTH=2}`
3. Type in cell B1 of worksheet: `{Size}`
4. Now it looks like:  

|   |                       A                   |   B    |   C   |
|---|-------------------------------------------|--------|-------|
| 1 | {COLL=Apples/Green;WIDTH=2}               | {Size} |       |
| 2 |                                           |        |       |

5. Let's describe data model:
```c#
public class GreenApple
{
   public double Size {set;get;}
}
public class Apples
{
  public List<GreenApple> Green {set;get;}
}
public class Root
{
  public Apples Apples {set;get;}
}
```
5. Ok, it's time to generate a report
```c#
// add some data
var root = new Root();
root.Apples = new Apples();
root.Apples.Green = new List<GreenApple>();
root.Apples.Green.Add(new GreenApple{Size = 5});
root.Apples.Green.Add(new GreenApple{Size = 10});

// making report
IReportEngine engine = new Engine();
engine.Provider = new ReflectionProvider(root);
engine.Go(@"file.xlsx", @"report.xlsx");
```
Now open the file `report.xlsx` to see what's generated. Or, you can see it here:

|   |                       A                   |   B    |   C   |
|---|-------------------------------------------|--------|-------|
| 1 |                                           | 5      |       |
| 2 |                                           | 10     |       |

# How it works?

## Data provider
At first, select a data provider. Some of them are ready out-of-box:
- Reflection provider for data stored in a tree of objects
- Xml Provider for data stored in xml file
- MySql provider for data stored in MySql database
- Oracle provider for data stored in Oracle database

## Custom data provider
If you aren't satisfied with capabailities that are ready out-of-the-box you can always implement data provider by yourself. Implement **IProvider** interface in order to use your own data provder.

```c#
public interface IProvider
    {
        int GetCollectionCount(string path, string tag, List<ContextItem> context);
        object GetValue(string path, string tag, List<ContextItem> context);
        ImageInfo GetImage(string path, string tag, List<ContextItem> context);
    }
```
## Report definition language
There is a syntax for data source definition: curvy bracets and special words. There are only three types of definitions: **collection**, **property** and **picture**.

## Language for Reflection Provider

Use collection definition in order to make some cells repeat itself x times. Here is a sample of such definition:
```
{COLL=Root/SomeProperty/SomeCollection;HEIGHT=1;WIDTH=10;GROW=DOWN;INSERT=NO;TAG=sometag}
```
Arguments are described below:
- COLL= is a path to property, each element (property) is separated with **"/"**, starting from the root of data model. When you place one collection inside another, you have to specify path to collection starting from current item (context).
- HEIGHT - ...
- WIDTH - ...
- GROW - grow direction. Use 'right' to make it grow right, or 'down' for growing down.
- INSERT - when it's set to 'no', inserting of cells is not performed while processing current collection. Default value is 'no'. It is useful when you want to generate a chess board, for example.
- TAG - anything you want to store here. Tags are passed to data providers.
- NESTED - useful for DB providers, we will talk about it later.

Use property definition to display data. Here is full sample:
```
{Car1/Wheel1/Diam;MULT=3.1;ADD=100;FORMAT=0.000;TAG=ha-ha}
```
Arguments are described below:
- necessary argument is a path to property. Use fully-qualified path, starting from data root, or starting from current item, when you are in a collection context.
- MULT multiples property value by it's argument.
- ADD adds argument to property value or to result of MULT, if MULT is specified.
- FORMAT is formatting numeric property value. For example, to get only 3.14 from PI, use format `0.00`
- TAG - anything you want to store here. Tags are passed to data providers.

**!TODO: add some help about `{PIC=...}`**

## Language for Oracle/MySql Provider

As for Oracle provider, use COLL to describe sql that returns data. Other properties are used to perform same things, excluding **nestes**. When you write a column name (of sql) in nested, it automatically becomes available for querying as a parameter in undelying collections/requests. There is a sample that makes it simple to understand.

At first, let's make a provider to DB. I have Oracle DB.
```c#
IReportEngine re = new Engine();
re.Provider = new OracleProvider(@"DATA SOURCE=localhost/sid;PASSWORD=SWORDFISH;PERSIST SECURITY INFO=True;USER ID=JOHN");
(re.Provider as OracleProvider).Parameters.Add("ARG1", "OP");
re.Go(@"C:\TEMP\template.xlsx", @"C:\TEMP\generated.xlsx");
```
^ Take a look, we passed a parameter: `ARG1`.

And here is what we placed in Excel worksheet. In cell A1 we use passed parameter.

|   |                       A                     | B |
|---|---------------------------------------------|---|
| 1 | {Coll=*select object_name from user_objects where object_type='TABLE' and object_name like **:ARG1** \|\| '%'*;width=2;height=3;**nested=object_name**} |   |
| 2 | {OBJECT_NAME} |   |
| 3 | {COLUMN_NAME}{Coll=*select column_name, table_name from all_tab_columns where table_name=**:object_name** order by column_name*;width=1;height=1;grow=down} |   |
| 4 |                                             |   |

Nested value **object_name** from query of cell A1 is passed as a parameter to query of cell A3.

An result is here:

MySql provider is same as Oracle, exepting one fact that you have to write sql in MySql dialect and pass parameters via MySql's `@param` syntax.

## Language for XML Provider

not ready yet :(


**TODO: write more docs!**
