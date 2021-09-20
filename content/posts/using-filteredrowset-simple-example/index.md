---
title: "Using JDBC FilteredRowSet - Simple Example"
date: "2016-01-25"
categories: 
  - "java"
tags: 
  - "filteredrowset"
  - "jdbc"
  - "rowset"
---

This post shows how to use a [FilteredRowSet](https://docs.oracle.com/javase/7/docs/api/javax/sql/rowset/FilteredRowSet.html) object. It lets you filter the number of rows that are visible in a `RowSet` object so that you can work with only the relevant data that you need. You may decide how you want to "filter" the data and apply that filter to a `FilteredRowSet` object. In other words, the `FilteredRowSet` object makes visible only the rows of data that fit within the limits you set.

You may know a [`JdbcRowSet`](https://docs.oracle.com/javase/7/docs/api/javax/sql/rowset/JdbcRowSet.html) object, which always has a connection to its data source, so you can do filtering with a query to the data source (using `WHERE` clause which defines the filtering criteria). A `FilteredRowSet` object provides a way for a [disconnected `RowSet`](http://javarevisited.blogspot.cz/2014/04/Connected-vs-disconnected-rowsetprovider-rowsetfactory-and-rowset-JDBC-Java.html) object for filtering without having to execute a query on the data source, consequently avoiding having to get a connection to the data source and sending queries to it.

![RowSet Classes in JDBC Java, FilteredRowSet](images/RowSet-Classes-in-JDBC-Java.png "RowSet Classes in JDBC Java (source: http://javarevisited.blogspot.cz/2014/04/Connected-vs-disconnected-rowsetprovider-rowsetfactory-and-rowset-JDBC-Java.html)")

There are two main steps to use the [FilteredRowSet:](https://docs.oracle.com/javase/7/docs/api/javax/sql/rowset/FilteredRowSet.html)

1. Create a new filter (an implementation of [Predicate](https://docs.oracle.com/javase/7/docs/api/javax/sql/rowset/Predicate.html) interface).
2. Use your filter (setting a filter to your [FilteredRowSet).](https://docs.oracle.com/javase/7/docs/api/javax/sql/rowset/FilteredRowSet.html)

## Create a new filter

You need to implement the `evaluate` method, which accepts a `Rowset object`. The [documentation](https://docs.oracle.com/javase/7/docs/api/javax/sql/rowset/Predicate.html) about this method says:

> This method is typically called a `FilteredRowSet` object internal methods (not public) that control the `RowSet` object's cursor moving from row to the next. In addition, if this internal method moves the cursor onto a row that has been deleted, the internal method will continue to ove the cursor until a valid row is found.

Other methods, like `evaluate(Object value, int column)` and `evaluate(Object value, String columnName),` are called when you are inserting new rows to a `FilteredRowSet` instance.

My example tries to match any regular expression pattern, which was set during a filter initialization. If a pattern found, method returned `true` (= yes, we want this row).

```java
/**
 * Search Filter for {@link FilteredRowSet}
 *
 * @author pavel.sklenar
 *
 */
class SearchFilter implements Predicate {
    private Pattern pattern;
 
    public SearchFilter(String searchRegex) {
        if (searchRegex != null && !searchRegex.isEmpty()) {
            pattern = Pattern.compile(searchRegex);
        }
    }
 
    public boolean evaluate(RowSet rs) {
        System.out.println("SearchFilter.evaluate called ");
        try {
            if (!rs.isAfterLast()) {
                String name = rs.getString("name");
                System.out.println(String.format(
                        "Searching for pattern '%s' in %s", pattern.toString(),
                        name));
                Matcher matcher = pattern.matcher(name);
                return matcher.matches();
            } else
                return false;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
 
    public boolean evaluate(Object value, int column) throws SQLException {
        throw new UnsupportedOperationException("Not supported yet.");
    }
 
    public boolean evaluate(Object value, String columnName)
            throws SQLException {
        throw new UnsupportedOperationException("Not supported yet.");
    }
}
```

## Use your filter

You need to set your new instance of `Predicate` interface to your `FilteredRowSet` using method `setFilter:`

```java
usersRS.setFilter(new SearchFilter("^[A-L].*"));
```

The complete example of my code is bellow. To run it, you need to add an H2 library (a memory database driver) to your classpath, e.g. available [here](http://www.h2database.com/html/download.html).

```java
/**
 * Example usage of {@link FilteredRowSet}
 *
 * @author pavel.sklenar
 *
 */
public class FilteredRowSetTest {
 
    /*
     * Sample names for test
     */
    private static final String[] NAMES = { "Bill Gates", "Steve Jobs",
            "Mark Zuckerberg", "Alan Turing", "Linus Torlvalds" };
 
    /**
     * The main class to run test
     */
    public static void main(String[] args) throws Exception {
        Connection c = DriverManager.getConnection("jdbc:h2:mem:db1", "test",
                "test");
        // Just only prepare data for test
        prepareData(c);
        RowSetFactory rsf = RowSetProvider.newFactory();
        FilteredRowSet usersRS = rsf.createFilteredRowSet();
        usersRS.setCommand("select * from USER");
        usersRS.execute(c);
        usersRS.setFilter(new SearchFilter("^[A-L].*"));
        dumpRS(usersRS);
    }
     
    /**
     * Dump {@link ResultSet}
     *
     * @param rs
     *            input{@link ResultSet} to dump
     * @throws SQLException
     * @throws Exception
     */
    public static void dumpRS(ResultSet rs) throws SQLException {
        ResultSetMetaData rsmd = rs.getMetaData();
        int cc = rsmd.getColumnCount();
        while (rs.next()) {
            for (int i = 1; i <= cc; i++) {
                System.out.println(rsmd.getColumnLabel(i) + " = "
                        + rs.getObject(i) + " ");
            }
            System.out.println("");
        }
    }
 
    /**
     * Prepare data for test
     *
     * @param c
     *            {@link Connection} will be used to prepare data
     * @throws SQLException
     */
    private static void prepareData(Connection c) throws SQLException {
        c.createStatement().execute("create table USER (name varchar(256))");
        PreparedStatement prepareStatement = c
                .prepareStatement("insert into USER (name) values (?)");
        for (String name : NAMES) {
            prepareStatement.setString(1, name);
            prepareStatement.execute();
        }
    }
}
```
**Output of the previous code:**

```
ContainFilter.evaluate called
Searching for pattern '^[A-L].*' in Bill Gates
NAME = Bill Gates
 
ContainFilter.evaluate called
Searching for pattern '^[A-L].*' in Steve Jobs
ContainFilter.evaluate called
Searching for pattern '^[A-L].*' in Mark Zuckerberg
ContainFilter.evaluate called
Searching for pattern '^[A-L].*' in Alan Turing
NAME = Alan Turing
 
ContainFilter.evaluate called
Searching for pattern '^[A-L].*' in Linus Torlvalds
NAME = Linus Torlvalds
```

## Resources

[https://docs.oracle.com/javase/7/docs/api/javax/sql/rowset/FilteredRowSet.html](https://docs.oracle.com/javase/7/docs/api/javax/sql/rowset/FilteredRowSet.html)

[https://docs.oracle.com/javase/7/docs/api/javax/sql/rowset/Predicate.html](https://docs.oracle.com/javase/7/docs/api/javax/sql/rowset/Predicate.html)

[https://docs.oracle.com/javase/tutorial/jdbc/basics/filteredrowset.html](https://docs.oracle.com/javase/tutorial/jdbc/basics/filteredrowset.html)

[http://javarevisited.blogspot.cz/2014/04/Connected-vs-disconnected-rowsetprovider-rowsetfactory-and-rowset-JDBC-Java.html](http://javarevisited.blogspot.cz/2014/04/Connected-vs-disconnected-rowsetprovider-rowsetfactory-and-rowset-JDBC-Java.html)
