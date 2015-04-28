/*
 *	Copyright 2007 Paul Marcotte
 *
 *  Licensed under the Apache License, Version 2.0 (the "License");
 *  you may not use this file except in compliance with the License.
 *  You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 *  Unless required by applicable law or agreed to in writing, software
 *  distributed under the License is distributed on an "AS IS" BASIS,
 *  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 *  See the License for the specific language governing permissions and
 *  limitations under the License.
 *
 *  Iterator.cfc (.3)
 *  [2007-05-31]	Added Apache license.
 *					Fixed bug in currentRow() method.
 *					Dropped cfscript blocks in favour of tags in all methods but queryRowToStruct().
 *  [2007-05-27]	Added end() and currentRow() methods.
 					Renamed rewind() to reset().
 *					Acknowlegement to Aaron Roberson for suggesting currentRow() method and other enhancements.
 *	[2007-05-10]	Initial Release.
 *					Acknowlegement to Peter Bell for the "Iterating Business Object" concept that Iterator seeks to provide as a composite.
 */

Iterator provides methods to load ColdFusion queries and traverse records.  It is designed to be composed into your domain objects.

For instance, if you use constructor injection, your init() method would accept Itertator as an argument.  You would then store it 
in variables scope and add delegate methods to utilize Iterator in your domain object.

The following examples assume you are using either ColdSpring or LightWire to inject your object dependencies and that you
maintain a reference to either as "Application.beanFactory".


Here is a snippet of code from a ColdSpring XML bean definition.  I currently place Iterator in a generic "util"
package, but you could place the file anywhere you like.

<bean id="Product" class="model.product.Product" singleton="false">

	<constructor-arg name="iterator">

		<ref bean="Iterator" />

	</constructor-arg>

</bean>
	

<bean id="Iterator" class="util.Iterator" singleton="false" />


Example init() method:

<cffunction name="init" access="public" returntype="model.product.Product" output="false">

	<cfargument name="iterator" type="Iterator" required="true" />

	<cfset variables.iterator = arguments.iterator />

	<cfreturn this />

</cffunction>



The minimum delegate methods required are loadQuery(), hasNext(), next() [reset() is optional but recommended].

Example delegate methods:



<cffunction name="loadQuery" access="public" returntype="void" output="false">

	<cfargument name="rs" type="query" required="true">

	<cfargument name="maxRecordsPerPage" type="numeric" required="false">

	<cfset variables.iterator.loadQuery(argumentCollection=arguments) />

</cffunction>



<cffunction name="hasNext" access="public" output="false" returntype="boolean">

	<cfreturn variables.iterator.hasNext() />

</cffunction>

	

<cffunction name="next" access="public" output="false" returntype="void">

	<cfif variables.iterator.hasNext()>

		<cfset variables.instance = variables.iterator.next() />

	</cfif>

</cffunction>

	

<cffunction name="reset" access="public" output="false" returntype="void">

	<cfset variables.iterator.reset() />

</cffunction>



How it works:

=============



The example delegate function next() prvovides insight into how to use Iterator.  Assuming you maintain your

domain object properties in variables.instance, calling next() will replace variables.instance with the next

record from the loaded query.  Iterator is useful as an alternative to maintaining an array of objects, or if

you list has calculated fields (an employee has
a startdate, but display format is "years of service").




Let's say you have a list page for a family of "Products" for a given "Category".  To display the Product list,

you might make a method call to your Service Layer to retrieve a query result and subsequently loop over the query

to display the Products.



Example:



<cfset productList = Application.beanFactory.getBean("ProductService").getProductsByCategory(CategoryId) />



<cfloop query="productList">

	{display code here}

</cfloop>



Using an Iterating Domain Object, you would first instantiate an object instance, then load the query into

the domain object.



Example 2:



<cfset products = Application.beanFactory.getBean("Product") />

<cfset productList = Application.beanFactory.getBean("ProductService").getProductsByCategory(CategoryId) />

<cfset products.loadQuery(productList) />



<cfloop condition="#products.hasNext()#">

	<cfset products.next() />

	{display code here}

</cfloop>



At a glance one might say, "well, the second example has more lines of code."  True, but if you has ever had

to display totals or other calculated values in your view, you would need to create variables in your views and

manage the setting of those variables.  To reduce the amount of logic in your views, any calculated values can be

encapsulated in the iterating domain object.  This facilitates better cohesion and reusability in your code.



If you are faced with some business problems that would benefit from using iteration, try the Iterator out.

