<div id="wikicontent">

<div id="wikimaincol" class="vt">

*   [Introduction](#Introduction)
*   [Details](#Details)
*   [Full API Reference](#Full_API_Reference)

*   [General](#General)
*   [What to search For](#What_to_search_For)
*   [Filtering](#Filtering)
*   [Selecting and Transforming Results](#Selecting_and_Transforming_Results)
*   [Sorting](#Sorting)
*   [Paging](#Paging)
*   [Controlling Association Fetching](#Controlling_Association_Fetching)
*   [Miscellaneous](#Miscellaneous)

*   [Additional Usage examples](#Additional_Usage_examples)

# <a name="Introduction"></a>Introduction[](#Introduction)

The framework features a powerful and flexible search functionality. This is used by passing a search object to search methods on general and generic DAOs.

The search object provides flexible search options:

*   Filtering on properties using standard operators ( =, !=, >, <, >=, <=, LIKE, IN, IS NULL, IS EMPTY ).
*   Filtering on collections and associations using ( SOME, ALL, NONE ).
*   Combining individual filters with any combination of logical operators ( AND, OR, NOT ).
*   Sorting on properties.
*   Paging.
*   Defining a search remotely from client code.
*   Transforming search results into objects, lists, arrays and maps
*   Specifying which associations to fetch eagerly.
*   Specifying column operators such as COUNT, SUM, AVG, MAX, etc.
*   All of the above work with nested properties (i.e. properties of related objects)

# <a name="Details"></a>Details[](#Details)

A Search object is the POJO that carries the various parameters to pass to a search. The core elements are lists of Filter, Sort and Field objects, and there are a number of other options.

**Javadocs**:

*   [http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/index.html?com/googlecode/genericdao/search/ISearch.html](http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/index.html?com/googlecode/genericdao/search/ISearch.html)
*   [http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/index.html?com/googlecode/genericdao/search/IMutableSearch.html](http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/index.html?com/googlecode/genericdao/search/IMutableSearch.html)
*   [http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/index.html?com/googlecode/genericdao/search/Search.html](http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/index.html?com/googlecode/genericdao/search/Search.html)

For more help with Searches look at the [SearchExamples](/p/hibernate-generic-dao/wiki/SearchExamples) page.

# <a name="Full_API_Reference"></a>Full API Reference[](#Full_API_Reference)

### <a name="General"></a>General[](#General)

The framework's search functionality API takes an <tt>ISearch</tt>. This interface is meant to be a read-only interface for a search object. The framework will not modify an <tt>ISearch</tt>. There is another interface, <tt>IMutableSearch</tt> which provides more methods for altering search parameters.

Anyone can provide their own implementation of the <tt>ISearch</tt> and/or <tt>IMutableSearch</tt> objects, but the framework includes a general one with lots of extra convenience methods for manipulating search parameters. This is the <tt>Search</tt> class. This <tt>Search</tt> class will probably be all you need for all your server-side operations. (Note: The framework also has an example of another <tt>ISearch</tt> implementation with the Flex search.)

### <a name="What_to_search_For"></a>What to search For[](#What_to_search_For)

A search object has a <tt>searchClass</tt> property that specifies which entity to search for. This may be left <tt>null</tt> when using a specific generic DAO; the DAO already knows what entity it is searching for.

**_Example:_**

<pre class="prettyprint"><span class="com">//create a new search for Projects</span><span class="pln">  
</span><span class="kwd">new</span> <span class="pln"></span> <span class="typ">Search</span><span class="pun">(</span><span class="typ">Project</span><span class="pun">.</span><span class="kwd">class</span><span class="pun">);</span><span class="pln">  

</span><span class="com">//or set the search class on an existing IMutableSearch</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">setSearchClass</span><span class="pun">(</span><span class="typ">Project</span><span class="pun">.</span><span class="kwd">class</span><span class="pun">);</span></pre>

### <a name="Filtering"></a>Filtering[](#Filtering)

A search can have a collection of _filters_. By default there is an 'AND' condition between these; however, the _disjunction_ property can be set to TRUE, and an 'OR' condition will be used.

See [http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/index.html?com/googlecode/genericdao/search/Filter.html](http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/index.html?com/googlecode/genericdao/search/Filter.html).

Each Filter object has three fields:

*   **property:** The name of the property to filter on. It may be nested. Examples: "name", "dateOfBirth", "employee.age", "employee.spouse.job.title". _NOTE: the empty string ("") or the constant Filter.ROOT_ENTITY can be used to specify the root entity._
*   **operator:** The type of comparison to do between the property and the value.
*   **value:** The value to compare the property with. Should be of a compatible type with the property. Examples: "Fred", new Date(), 45

A filter can be created simply setting the fields and adding it to the search...

<pre class="prettyprint"><span class="typ">Filter</span> <span class="pln">f</span> <span class="pun">=</span> <span class="pln"></span> <span class="kwd">new</span> <span class="pln"></span> <span class="typ">Filter</span><span class="pun">();</span><span class="pln">  
f</span><span class="pun">.</span><span class="pln">setProperty</span><span class="pun">(</span><span class="str">"name"</span><span class="pun">);</span><span class="pln">  
f</span><span class="pun">.</span><span class="pln">setOperator</span><span class="pun">(</span><span class="typ">Filter</span><span class="pun">.</span><span class="pln">OP_EQUAL</span><span class="pun">);</span><span class="pln">  
f</span><span class="pun">.</span><span class="pln">setValue</span><span class="pun">(</span><span class="str">"Bob"</span><span class="pun">);</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addFilter</span><span class="pun">(</span><span class="pln">f</span><span class="pun">);</span><span class="pln">  

</span><span class="com">//or</span><span class="pln">  
</span><span class="typ">Filter</span> <span class="pln">f</span> <span class="pun">=</span> <span class="pln"></span> <span class="kwd">new</span> <span class="pln"></span> <span class="typ">Filter</span><span class="pun">(</span><span class="str">"name"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"Bob"</span><span class="pun">,</span> <span class="pln"></span> <span class="typ">Filter</span><span class="pun">.</span><span class="pln">OP_EQUAL</span><span class="pun">);</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addFilter</span><span class="pun">(</span><span class="pln">f</span><span class="pun">);</span></pre>

The Filter class also has static methods for creating filters with certain operators...

<pre class="prettyprint"><span class="typ">Filter</span> <span class="pln">f</span> <span class="pun">=</span> <span class="pln"></span> <span class="typ">Filter</span><span class="pun">.</span><span class="pln">equal</span><span class="pun">(</span><span class="str">"name"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"Bob"</span><span class="pun">);</span> <span class="pln"></span> <span class="com">//creates a filter with the OP_EQUAL operator</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addFilter</span><span class="pun">(</span><span class="pln">f</span><span class="pun">);</span><span class="pln">  

</span><span class="com">//or another example with the OP_GREATER_THAN operator</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addFilter</span><span class="pun">(</span> <span class="pln"></span> <span class="typ">Filter</span><span class="pun">.</span><span class="pln">greaterThan</span><span class="pun">(</span><span class="str">"age"</span><span class="pun">,</span> <span class="pln"></span> <span class="lit">7</span><span class="pun">)</span> <span class="pln"></span> <span class="pun">);</span></pre>

Finally, the <tt>Search</tt> class has methods for creating and adding a filter all in one step...

<pre class="prettyprint"><span class="pln">search</span><span class="pun">.</span><span class="pln">addFilterEqual</span><span class="pun">(</span><span class="str">"name"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"Bob"</span><span class="pun">);</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addFilterGreaterThan</span><span class="pun">(</span><span class="str">"age"</span><span class="pun">,</span> <span class="pln"></span> <span class="lit">7</span><span class="pun">);</span></pre>

The following filtering operators are available:

<table class="wikitable">

<tbody>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">**Operator**</td>

<td style="border: 1px solid #ccc; padding: 5px;">**Explanation**</td>

<td style="border: 1px solid #ccc; padding: 5px;">**Example**</td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">EQUAL</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.equal("name", "Bob")</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">NOT_EQUAL</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.notEqual("age", 5)</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">GREATER_THAN</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.greaterThan("age", 5)</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">GREATER_OR_EQUAL</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.greaterOrEqual("name", "M")</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">LESS_THAN</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.lessThan("name", "N")</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">LESS_OR_EQUAL</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.lessOrEqual("age", 65)</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">IN</td>

<td style="border: 1px solid #ccc; padding: 5px;">Equal to one of the items in the list of values. Value can be a collection or an array.</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.in("eyeColor", EyeColor.BLUE, EyeColor.HAZEL)</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">NOT_IN</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.notIn("hairColor", HairColor.BROWN, HairColor.BLACK)</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">LIKE</td>

<td style="border: 1px solid #ccc; padding: 5px;">Takes a SQL like expression</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.like("name", "Wil%")</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">ILIKE</td>

<td style="border: 1px solid #ccc; padding: 5px;">LIKE + ignore case (Note: Some databases ignore case by default)</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.ilike("name", "wiL%")</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">NULL</td>

<td style="border: 1px solid #ccc; padding: 5px;">SQL IS NULL</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.isNull("primaryDoctor")</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">NOT_NULL</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.isNotNull("phone")</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">EMPTY</td>

<td style="border: 1px solid #ccc; padding: 5px;">NULL or empty string or empty collection/assocation</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.isEmpty("children")</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">NOT_EMPTY</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.isNotEmpty("primaryDoctor.firstName")</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">SOME</td>

<td style="border: 1px solid #ccc; padding: 5px;">Applies to collection/association properties. Takes another <tt>Filter</tt> as a value, and matches when at least one of the values in the collection matches the filter.</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.some("children", Filter.equal("name", "Joey")) //has a child named 'Joey'</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">ALL</td>

<td style="border: 1px solid #ccc; padding: 5px;">Same as SOME, except that all values must match the filter.</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.all("children", Filter.greaterOrEqual("age", 18)) //all children are 18 or older</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">NONE</td>

<td style="border: 1px solid #ccc; padding: 5px;">Same as SOME, except that none of the values may match the filter.</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.none("pets", Filter.and(Filter.equal("species", "cat"), Filter.lessThan("age", .75)) //has no cats under 9 months old</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">AND</td>

<td style="border: 1px solid #ccc; padding: 5px;">Takes no property. Takes an array or collection of <tt>Filter</tt>s as a value. Matches when all the filters in the value match.</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.and(Filter.greaterOrEqual("age",40), Filter.lessThan("age", 65))</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">OR</td>

<td style="border: 1px solid #ccc; padding: 5px;">Same as AND, except that it matches when any of the filters match.</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.or(Filter.like("firstName","Wil%"), Filter.like("lastName","Wil%"))</tt></td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">NOT</td>

<td style="border: 1px solid #ccc; padding: 5px;">Takes no property. Takes a single <tt>Filter</tt> as a value. Matches when the filter in the value does not match.</td>

<td style="border: 1px solid #ccc; padding: 5px;"><tt>Filter.not(Filter.ilike("name", "W%")) //name does not start with 'w'</tt></td>

</tr>

<tr>

<td colspan="3" style="border: 1px solid #ccc; padding: 5px;"><tt>**</tt> For more examples of all these operators, see [SearchExamples](/p/hibernate-generic-dao/wiki/SearchExamples)</td>

</tr>

</tbody>

</table>

**NOTE:** If any filter with an operator that expects a value has a null value, that filter is ignored. This is very handy in the case of writing a find method where null arguments are ignored. For example:

<pre class="prettyprint"><span class="com">/**  
 * This is a DAO method that returns all the people with the given first and/or last name.  
 * If any parameter is null, that parameter is ignored. So only non-null parameters are used in defining the search criteria.  
 */</span><span class="pln">  
</span><span class="kwd">public</span> <span class="pln"></span> <span class="typ">List</span><span class="pun"><</span><span class="typ">Person</span><span class="pun">></span> <span class="pln">findPeopleByName</span><span class="pun">(</span><span class="typ">String</span> <span class="pln">firstName</span><span class="pun">,</span> <span class="pln"></span> <span class="typ">String</span> <span class="pln">lastName</span><span class="pun">)</span> <span class="pln"></span> <span class="pun">{</span><span class="pln">   </span> <span class="kwd">return</span> <span class="pln">search</span><span class="pun">(</span><span class="kwd">new</span> <span class="pln"></span> <span class="typ">Search</span><span class="pun">(</span><span class="typ">Person</span><span class="pun">.</span><span class="kwd">class</span><span class="pun">).</span><span class="pln">addFilterEqual</span><span class="pun">(</span><span class="str">"firstName"</span><span class="pun">,</span> <span class="pln">firstName</span><span class="pun">).</span><span class="pln">addFilterEqual</span><span class="pun">(</span><span class="str">"lastName"</span><span class="pun">,</span> <span class="pln">lastName</span><span class="pun">));</span><span class="pln">  
</span><span class="pun">}</span></pre>

**NOTE:** Filters also support using the special HQL properties ".id", ".class" and ".size". For example, the following expression would produce a filter that would match objects whose members collection contains more than 10 elements:

<pre class="prettyprint"><span class="typ">Filter</span><span class="pun">.</span><span class="pln">greaterThan</span><span class="pun">(</span><span class="str">"members.size"</span><span class="pun">,</span> <span class="pln"></span> <span class="lit">10</span><span class="pun">)</span></pre>

**_Filter by Example_**

The framework has a built-in utility for converting example objects into Filters that can be used in a search. All DAOs and the SearchFacade provide two methods for doing this:

<pre class="prettyprint"><span class="kwd">public</span> <span class="pln"></span> <span class="typ">Filter</span> <span class="pln">getFilterFromExample</span><span class="pun">(</span><span class="typ">Object</span> <span class="pln">example</span><span class="pun">);</span><span class="pln">  
</span><span class="kwd">public</span> <span class="pln"></span> <span class="typ">Filter</span> <span class="pln">getFilterFromExample</span><span class="pun">(</span><span class="typ">Object</span> <span class="pln">example</span><span class="pun">,</span> <span class="pln"></span> <span class="typ">ExampleOptions</span> <span class="pln">options</span><span class="pun">);</span></pre>

The first method uses the default options for doing the conversion, while the second provides several options for controlling the result. See [http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/com/googlecode/genericdao/search/ExampleOptions.html](http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/com/googlecode/genericdao/search/ExampleOptions.html) for all the options.

### <a name="Selecting_and_Transforming_Results"></a>Selecting and Transforming Results[](#Selecting_and_Transforming_Results)

By default, the results of a search will be a list of the entity objects that were searched on. However, as with Hibernate queries, it is possible to get different information back in each row.

**Fields**

A search can have a collection of _fields_. These are analogous to the "SELECT" clause in SQL or HQL. Each field corresponds to a property value or a column operator applied to a property. Each field may also be assigned a key string. This will be used instead of the property name as a map key when using the MAP result mode. If no fields are specified, the entity itself is used as the single result for each record.

See [http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/index.html?com/googlecode/genericdao/search/Field.html](http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/index.html?com/googlecode/genericdao/search/Field.html).

**NOTE:** the empty string ("") or the constant Filter.ROOT_ENTITY can be used to specify the root entity.

Here are the available field operators:

*   PROPERTY (i.e. no operator)
*   COUNT
*   COUNT_DISTINCT
*   MAX
*   MIN
*   SUM
*   AVG

**Result Modes**

The following result modes are available for a search:

<table class="wikitable">

<tbody>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">AUTO</td>

<td style="border: 1px solid #ccc; padding: 5px;">The result mode is automatically determined according to the following rules: If any field is specified with a key, use MAP mode. Otherwise, if zero or one fields are specified, use SINGLE mode. Otherwise, use ARRAY mode.</td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">ARRAY</td>

<td style="border: 1px solid #ccc; padding: 5px;">returns each result as an Object array (<tt>Object[]</tt>) with the entries corresponding to the fields added to the search.</td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">LIST</td>

<td style="border: 1px solid #ccc; padding: 5px;">returns each result as a list of Objects (<tt>List<Object></tt>).</td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">MAP</td>

<td style="border: 1px solid #ccc; padding: 5px;">returns each row as a map with properties' names or keys for keys to the corresponding values.</td>

</tr>

<tr>

<td style="border: 1px solid #ccc; padding: 5px;">SINGLE</td>

<td style="border: 1px solid #ccc; padding: 5px;">Exactly one field or no fields must be specified to use this result mode. The result list contains just the value of that property for each element or the entity if no fields are specified.</td>

</tr>

</tbody>

</table>

**Distinct**

The <tt>distinct</tt> option can be set on a search in order to filter out duplicate results. As with plain HQL or SQL, only use this option if your search requires it because it does affect performance.

"Distinct" can only be used on queries that return a single field. Here are some examples:

<pre class="prettyprint"><span class="pln">search</span><span class="pun">(</span><span class="kwd">new</span> <span class="pln"></span> <span class="typ">Search</span><span class="pun">().</span><span class="pln">setDistinct</span><span class="pun">(</span><span class="kwd">true</span><span class="pun">));</span> <span class="pln"></span> <span class="com">//would work. Only the root entity is returned for each result.</span><span class="pln">  
search</span><span class="pun">(</span><span class="kwd">new</span> <span class="pln"></span> <span class="typ">Search</span><span class="pun">().</span><span class="pln">setDistinct</span><span class="pun">(</span><span class="kwd">true</span><span class="pun">).</span><span class="pln">addField</span><span class="pun">(</span><span class="str">"firstName"</span><span class="pun">);</span> <span class="pln"></span> <span class="com">//would work. Only the single "firstName" value is returned for each result.</span><span class="pln">  
search</span><span class="pun">(</span><span class="kwd">new</span> <span class="pln"></span> <span class="typ">Search</span><span class="pun">().</span><span class="pln">setDistinct</span><span class="pun">(</span><span class="kwd">true</span><span class="pun">).</span><span class="pln">addField</span><span class="pun">(</span><span class="str">"firstName"</span><span class="pun">).</span><span class="pln">addField</span><span class="pun">(</span><span class="str">"lastName"</span><span class="pun">));</span> <span class="pln"></span> <span class="com">//would NOT work. Two fields are returned for each result.</span></pre>

**_Examples:_**

<pre class="prettyprint"><span class="typ">Search</span> <span class="pln">search</span> <span class="pun">=</span> <span class="pln"></span> <span class="kwd">new</span> <span class="pln"></span> <span class="typ">Search</span><span class="pun">(</span><span class="typ">Project</span><span class="pun">.</span><span class="kwd">class</span><span class="pun">);</span><span class="pln">  

</span><span class="com">//The first field is the name of the project.</span> <span class="pln">  
search</span><span class="pun">.</span><span class="pln">addField</span><span class="pun">(</span><span class="str">"name"</span><span class="pun">);</span><span class="pln">  
</span><span class="com">//Project has a many-to-one relationship with type. Type objects have a 'name' property.</span><span class="pln">  
</span><span class="com">//We are using "type" for the key (or alias) of this field.</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addField</span><span class="pun">(</span><span class="str">"type.name"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"type"</span><span class="pun">);</span><span class="pln">  

</span><span class="com">//Using resultMode AUTO, this will produce a Map for each result because a key is</span><span class="pln">  
</span><span class="com">//specified. Each result will be something like</span><span class="pln">  
</span><span class="com">//Map{ 'name': 'Hibernate Generic DAO', 'type': 'cooler than cool' }.</span><span class="pln">  
dao</span><span class="pun">.</span><span class="pln">search</span><span class="pun">(</span><span class="pln">search</span><span class="pun">);</span><span class="pln">  

search</span><span class="pun">.</span><span class="pln">setResultMode</span><span class="pun">(</span><span class="typ">Search</span><span class="pun">.</span><span class="pln">RESULT_ARRAY</span><span class="pun">);</span><span class="pln">  
</span><span class="com">//Now using resultMode ARRAY, each result will be something like</span><span class="pln">  
</span><span class="com">//Array[ 'Hibernate Generic DAO', 'cooler than cool' ]</span><span class="pln">  
dao</span><span class="pun">.</span><span class="pln">search</span><span class="pun">(</span><span class="pln">search</span><span class="pun">);</span><span class="pln">  
</span></pre>

### <a name="Sorting"></a>Sorting[](#Sorting)

A search can have a collection of _sorts_. These sorts are applied in order, just as in the SQL "ORDER BY" clause. Each sort specifies a property and a direction (asc or desc). There is also a flag for whether or not to ignore case. (Note: Some databases ignore case by default.)

See [http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/index.html?com/googlecode/genericdao/search/Sort.html](http://hibernate-generic-dao.googlecode.com/svn/trunk/docs/api/index.html?com/googlecode/genericdao/search/Sort.html).

**_Examples:_**

<pre class="prettyprint"><span class="com">//this produces: ORDER BY startDate DESC</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addSort</span><span class="pun">(</span> <span class="pln"></span> <span class="typ">Sort</span><span class="pun">.</span><span class="pln">desc</span><span class="pun">(</span><span class="str">"startDate"</span><span class="pun">)</span> <span class="pln"></span> <span class="pun">);</span><span class="pln">  

</span><span class="com">//this produces: ORDER BY lower(name) ASC</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addSort</span><span class="pun">(</span> <span class="pln"></span> <span class="typ">Sort</span><span class="pun">.</span><span class="pln">asc</span><span class="pun">(</span><span class="str">"name"</span><span class="pun">,</span> <span class="pln"></span> <span class="kwd">true</span><span class="pun">)</span> <span class="pln"></span> <span class="pun">);</span> <span class="pln"></span> <span class="com">//second parameter specifies ignore case</span><span class="pln">  

</span><span class="com">//this produces: ORDER BY lastName ASC, firstName ASC</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addSortAsc</span><span class="pun">(</span><span class="str">"lastName"</span><span class="pun">).</span><span class="pln">addSortAsc</span><span class="pun">(</span><span class="str">"firstName"</span><span class="pun">);</span></pre>

### <a name="Paging"></a>Paging[](#Paging)

Searches have three properties that define paging behavior.

*   <tt>maxResults</tt> - the maximum number of results to return. (If <tt><= 0, maxResults</tt> is ignored.)
*   <tt>firstResult</tt> - the zero-based offset of the first result to return. (If <tt><= 0, firstResult</tt> is ignored.)
*   <tt>page</tt> - the zero-based offset of the first page to return. <tt>page</tt> applies only if <tt>firstResult</tt> is <tt><= 0</tt>. <tt>maxResults</tt> is used as the page size for determining the record offset. For example, if <tt>maxResults</tt> is 10 and <tt>page</tt> is 3, results 30 through 39 (zero-based) will be returned.

**_Example:_**

<pre class="prettyprint"><span class="pln">search</span><span class="pun">.</span><span class="pln">setMaxResults</span><span class="pun">(</span><span class="lit">10</span><span class="pun">);</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">setPage</span><span class="pun">(</span><span class="lit">3</span><span class="pun">);</span></pre>

### <a name="Controlling_Association_Fetching"></a>Controlling Association Fetching[](#Controlling_Association_Fetching)

Search objects can also have a _fetches_ collection. Each fetch is simply a string that identifies an association to eagerly fetch.

**_Example:_**

<pre class="prettyprint"><span class="typ">Search</span> <span class="pln">search</span> <span class="pun">=</span> <span class="pln"></span> <span class="kwd">new</span> <span class="pln"></span> <span class="typ">Search</span><span class="pun">(</span><span class="typ">Project</span><span class="pun">.</span><span class="kwd">class</span><span class="pun">);</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addFetch</span><span class="pun">(</span><span class="str">"members"</span><span class="pun">);</span><span class="pln">  

</span><span class="com">//The members of each project in the result set will be join fetched with</span><span class="pln">  
</span><span class="com">//the results of this search.</span></pre>

### <a name="Miscellaneous"></a>Miscellaneous[](#Miscellaneous)

Search supports the use of HQL special properties ".id", ".class" and ".size" where ever they are supported in HQL. Just include them within the property. Ex: <tt>"members.size"</tt>, <tt>"class"</tt>, <tt>"id.project"</tt>.

# <a name="Additional_Usage_examples"></a>Additional Usage examples[](#Additional_Usage_examples)

<pre class="prettyprint"><span class="typ">Search</span> <span class="pln">search</span> <span class="pun">=</span> <span class="pln"></span> <span class="kwd">new</span> <span class="pln"></span> <span class="typ">Search</span><span class="pun">(</span><span class="typ">Project</span><span class="pun">.</span><span class="kwd">class</span><span class="pun">);</span><span class="pln">  

search</span><span class="pun">.</span><span class="pln">addFilterEqual</span><span class="pun">(</span><span class="str">"name"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"hibernate-generic-dao"</span><span class="pun">);</span><span class="pln">  

search</span><span class="pun">.</span><span class="pln">addFilterLessThan</span><span class="pun">(</span><span class="str">"completionDate"</span><span class="pun">,</span> <span class="pln"></span> <span class="kwd">new</span> <span class="pln"></span> <span class="typ">Date</span><span class="pun">());</span><span class="pln">  

search</span><span class="pun">.</span><span class="pln">addFilterOr</span><span class="pun">(</span><span class="pln">               </span> <span class="typ">Filter</span><span class="pun">.</span><span class="pln">equal</span><span class="pun">(</span><span class="str">"name"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"Jack"</span><span class="pun">),</span><span class="pln">               </span> <span class="typ">Filter</span><span class="pun">.</span><span class="kwd">and</span><span class="pun">(</span><span class="pln">                               </span> <span class="typ">Filter</span><span class="pun">.</span><span class="pln">equal</span><span class="pun">(</span><span class="str">"name"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"Jill"</span><span class="pun">),</span><span class="pln">                               </span> <span class="typ">Filter</span><span class="pun">.</span><span class="pln">like</span><span class="pun">(</span><span class="str">"location"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"%Chicago%"</span><span class="pun">),</span><span class="pln">                               </span> <span class="typ">Filter</span><span class="pun">.</span><span class="pln">greaterThan</span><span class="pun">(</span><span class="str">"age"</span><span class="pun">,</span> <span class="pln"></span> <span class="lit">5</span><span class="pun">)</span><span class="pln">               </span> <span class="pun">)</span><span class="pln">  
</span><span class="pun">);</span><span class="pln">  

search</span><span class="pun">.</span><span class="pln">addFilterIn</span><span class="pun">(</span><span class="str">"name"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"Jack"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"Jill"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"Bob"</span><span class="pun">);</span><span class="pln">  

search</span><span class="pun">.</span><span class="pln">addFilterNot</span><span class="pun">(</span><span class="typ">Filter</span><span class="pun">.</span><span class="kwd">in</span><span class="pun">(</span><span class="str">"name"</span><span class="pun">,</span><span class="str">"Jack"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"Jill"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"Bob"</span><span class="pun">));</span><span class="pln">  

search</span><span class="pun">.</span><span class="pln">addSort</span><span class="pun">(</span><span class="str">"name"</span><span class="pun">);</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addSort</span><span class="pun">(</span><span class="str">"age"</span><span class="pun">,</span> <span class="pln"></span> <span class="kwd">true</span><span class="pun">);</span> <span class="pln"></span> <span class="com">//descending</span><span class="pln">  

</span><span class="com">//paging</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">setMaxResults</span><span class="pun">(</span><span class="lit">15</span><span class="pun">);</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">setPage</span><span class="pun">(</span><span class="lit">3</span><span class="pun">);</span></pre>

Nested properties are also fully supported...

<pre class="prettyprint"><span class="pln">search</span><span class="pun">.</span><span class="pln">addFilterEqual</span><span class="pun">(</span><span class="str">"status.name"</span><span class="pun">,</span> <span class="pln"></span> <span class="str">"active"</span><span class="pun">);</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addFilterGreaterThan</span><span class="pun">(</span><span class="str">"workgroup.manager.salary"</span><span class="pun">,</span> <span class="pln"></span> <span class="lit">75000.00</span><span class="pun">);</span><span class="pln">  

search</span><span class="pun">.</span><span class="pln">addSort</span><span class="pun">(</span><span class="str">"status.name"</span><span class="pun">);</span></pre>

Calling a search:

<pre class="prettyprint"><span class="typ">Search</span> <span class="pln">search</span> <span class="pun">=</span> <span class="pln"></span> <span class="kwd">new</span> <span class="pln"></span> <span class="typ">Search</span><span class="pun">();</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addFilterGreaterThan</span><span class="pun">(</span><span class="str">"userCount"</span><span class="pun">,</span> <span class="pln"></span> <span class="lit">500</span><span class="pun">);</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">setMaxResults</span><span class="pun">(</span><span class="lit">15</span><span class="pun">);</span><span class="pln">  

</span><span class="com">//get one page of results</span><span class="pln">  
</span><span class="typ">List</span><span class="pun"><</span><span class="typ">Project</span><span class="pun">></span> <span class="pln">results</span> <span class="pun">=</span> <span class="pln">projectDAO</span><span class="pun">.</span><span class="pln">search</span><span class="pun">(</span><span class="pln">search</span><span class="pun">);</span><span class="pln">  

</span><span class="com">//get the total number of results (ignores paging)</span><span class="pln">  
</span><span class="kwd">int</span> <span class="pln">totalResults</span> <span class="pun">=</span> <span class="pln">projectDAO</span><span class="pun">.</span><span class="pln">count</span><span class="pun">(</span><span class="pln">search</span><span class="pun">);</span><span class="pln">  

</span><span class="com">//get one page of results and the total number of results without paging</span><span class="pln">  
</span><span class="typ">SearchResult</span><span class="pun"><</span><span class="typ">Project</span><span class="pun">></span> <span class="pln">result</span> <span class="pun">=</span> <span class="pln">projectDAO</span><span class="pun">.</span><span class="pln">searchAndCount</span><span class="pun">(</span><span class="pln">search</span><span class="pun">);</span><span class="pln">results</span> <span class="pun">=</span> <span class="pln">result</span><span class="pun">.</span><span class="pln">getResults</span><span class="pun">();</span><span class="pln">totalResults</span> <span class="pun">=</span> <span class="pln">results</span><span class="pun">.</span><span class="pln">getTotalCount</span><span class="pun">();</span><span class="pln">  

</span><span class="com">//get the average userCount for project matching the filter criteria</span><span class="pln">  
search</span><span class="pun">.</span><span class="pln">addField</span><span class="pun">(</span><span class="str">"userCount"</span><span class="pun">,</span> <span class="pln"></span> <span class="typ">Field</span><span class="pun">.</span><span class="pln">OP_AVG</span><span class="pun">);</span><span class="pln">  
</span><span class="kwd">long</span> <span class="pln">avgCount</span> <span class="pun">=</span> <span class="pln"></span> <span class="pun">(</span><span class="kwd">long</span><span class="pun">)</span> <span class="pln">projectDAO</span><span class="pun">.</span><span class="pln">searchUnique</span><span class="pun">(</span><span class="pln">search</span><span class="pun">);</span></pre>

More examples... [SearchExamples](/p/hibernate-generic-dao/wiki/SearchExamples)

</div>

</div>
