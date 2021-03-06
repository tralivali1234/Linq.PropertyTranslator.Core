## Linq.PropertyTranslator.Core

[![Build status](https://ci.appveyor.com/api/projects/status/u6qabmbi56bf4nhi?svg=true)](https://ci.appveyor.com/project/StefH/linq-propertytranslator-core)
<a href="https://scan.coverity.com/projects/stefh-linq-propertytranslator-core">
<img alt="Coverity Scan Build Status" src="https://scan.coverity.com/projects/8709/badge.svg"/>
</a>

This is a **.net Core port** of the PropertyTranslator (https://github.com/peschuster/PropertyTranslator).

The following frameworks are supported:
- net40
- net45
- net451
- net452
- net46
- net461
- netstandard1.3
- netstandard1.4
- netstandard1.5


Translates computed properties in LINQ queries into their implementation (based on [Microsoft.Linq.Translations](https://github.com/damieng/Linq.Translations)). 

### What does it?

*PropertyTranslator* exchanges properties in linq queries before execution. This is especially useful if the underlying LINQ provider does not support some kind of operation or you want to add client-side calculations in your business logic to e.g. an EntityFramework model.

For a general introduction into the topic, have a look at [this blog post](http://damieng.com/blog/2009/06/24/client-side-properties-and-any-remote-linq-provider). *PropertyTranslator* actually is a enhancement of the presented solution in the post.

For an introduction specifically to PropertyTranslator have a look at these two blog posts: [LINQ: How to dynamically map properties](http://www.peschuster.de/2012/03/linq-how-to-dynamically-map-properties/) and [PropertyTranslator and Interfaces](http://www.peschuster.de/2012/03/propertytranslator-and-interfaces/)

### What's the difference to Linq.Translations?

PropertyTranslator plays well together with [QueryInterceptor](https://github.com/stefh/QueryInterceptor.Core) and thus can be added to every query in some kind of "data context" or general table / *ObjectSet* provider.

Furthermore it internally adds one more layer of abstraction to allow property translation depending on the ui culture of the current thread.

### Examples

#### Basic example

A POCO entity class from EntityFramework. Although in the database only a `FirstName` and a `LastName` field exists, the property `Name` can be used in queries, because right before execution of the query it is translated to `FirstName + ' ' + LastName`.

    public class Person
    {
    	private static readonly CompiledExpressionMap<Person, string> fullNameExpression = 
    	    DefaultTranslationOf<Person>.Property(p => p.FullName).Is(p => p.FirstName + " " + p.LastName);
    	    
    	public string FullName
    	{
    		get { return fullNameExpression.Evaluate(this); }
    	}
    	
    	public string FirstName { get; set; }
    	
    	public string LastName { get; set; }    	
    }

#### A more advanced example with ui culture dependent translations

The context: a database table, mapped with entity framework to POCO entity classes with two fields: `EnglishName` and `GermanName`. With the following snippet, you can use the `Name` property in linq queries which resolves to the name (either `EnglishName` or `GermanName`) depending on the current ui culture.

    public class Country
    {
    	private static readonly CompiledExpressionMap<Country, string> nameExpression = 
    	    DefaultTranslationOf<Country>.Property(c => c.Name).Is(c => c.EnglishName);
    	
    	static Country()
    	{
    	    DefaultTranslationOf<Country>.Property(c => c.Name).Is(c => c.EnglishName, 'en');
    	    DefaultTranslationOf<Country>.Property(c => c.Name).Is(c => c.GermanName, 'de');
    	}    	
    	
    	public string Name
    	{
    		get { return nameExpression.Evaluate(this); }
    	}
    	
    	public string EnglishName { get; set; }
    	
    	public string GermanName { get; set; }    	
    }

### How to enable PropertyTranslator

You can *enable* PropertyTranslator by adding the `PropertyVisitor` to your EntityFramework ObjectSets (of course it works not only with EntityFramework but with any LINQ provider):

    using QueryInterceptor;
    using PropertyTranslator;

    public class MyDataContext
    {
        ObjectContext context = new MyEfContext();
        
        public IQueryable<Person> PersonTable
        {
            get
            {
                var objectSet = context.CreateObjectSet<Person>("Persons");
                
                return objectSet.InterceptWith(new PropertyVisitor());
            }
        }
    }

## How to use it
PropertyTranslator is on Nuget: [http://nuget.org/packages/Linq.PropertyTranslator.Core](http://nuget.org/packages/Linq.PropertyTranslator.Core)
I'd recommend to use it together with [QueryInterceptor](http://nuget.org/packages/QueryInterceptor.Core) by Stef Heyenrath.
