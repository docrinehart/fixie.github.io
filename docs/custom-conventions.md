---
layout: documentation
title: Fixie - Custom Conventions
---
## Custom Conventions

Although useful for simple scenarios, the [default convention](../default-convention) may not meet your needs. Fortunately, you can replace it with your own.

If you don't want to go with the behaviors defined in the default convention, simply place a subclass of `Convention` beside your tests.  A custom subclass of `Convention` will reach out into the containing test assembly, looking for tests to execute.  Each convention can customize [test discovery](../test-discovery) and test execution.  For test discovery, you describe what your test classes and test methods look like.  For test execution, you can take control over how frequently your test classes are constructed and how they are constructed.  Additionally, you can wrap custom behavior around each test case, around each test class instance, and around each test class.

For instance, let's say we want all of our integration tests to be automatically wrapped in a database transaction.  Beside our tests, we place a custom convention class:

{% highlight csharp %}
using Fixie;

namespace IntegrationTests
{
    public class IntegrationTestConvention : Convention
    {
        public IntegrationTestConvention()
        {
            Classes
                .NameEndsWith("Tests");

            Methods
                .Where(method => method.IsVoid());

            FixtureExecution
                .Wrap<Transaction>();
        }

        class Transaction : FixtureBehavior
        {
            public void Execute(Fixture context, Action next)
            {
                using (new TransactionScope())
                    next();
            }
        }
    }
}
{% endhighlight %}

Here, we're declaring that for each *Fixture* (in other words, each test class instance), wrap it in its own `TransactionScope` instance.  When the test class is constructed, you get a new `TransactionScope`.  After the tests are run for that instance, the `TransactionScope` is disposed.  You could alternatively wrap per-test-case (using the `CaseExecution` property) or per-test-class (using the `ClassExecution` property).

Contrasting with NUnit, `CaseExecution` wrapping corresponds with NUnit's `[SetUp]` and `[TearDown]`, and 'FixtureExecution' wrapping corresponds with NUnit's `[FixtureSetUp]` and `[FixtureTearDown]`.

Several sample conventions are available under the [Fixie.Samples](https://github.com/plioi/fixie/tree/master/src/Fixie.Samples) project:

* [Imitate NUnit](https://github.com/plioi/fixie/blob/master/src/Fixie.Samples/NUnitStyle/CustomConvention.cs)
* [Imitate xUnit](https://github.com/plioi/fixie/blob/master/src/Fixie.Samples/xUnitStyle/CustomConvention.cs)
* [Simplified NUnit for cleaner test inheritance](https://github.com/plioi/fixie/blob/master/src/Fixie.Samples/LowCeremony/CustomConvention.cs)
* [Construct integration test classes with your IoC container](https://github.com/plioi/fixie/blob/master/src/Fixie.Samples/IoC/CustomConvention.cs)
* [Support arbitrary command line flags such as NUnit-style categories](https://github.com/plioi/fixie/blob/master/src/Fixie.Samples/Categories/CustomConvention.cs)