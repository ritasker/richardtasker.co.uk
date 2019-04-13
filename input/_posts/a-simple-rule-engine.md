Published: 2014-01-19
Title: A Simple Rule Engine
Lead: After a brainstorming session, and the use of a few interfaces and design patterns a potentially complicated rules engine turned out to be quite simple.
Author: richtasker
permalink: /2014/01/19/a-simple-rule-engine
Tags:
  - C#
  - Design Patterns
  - Strategy
---
This week orders from the board of directors re-prioritised the current sprint to include a new feature. The feature would add a process which would trigger a chain of events once costs exceeded a predefined threshold. 
  
The feature could of been completed by checking the predefined threshold once costs had been submitted. However a future piece of work will also require the checking of predefined values, we decided to write the feature in a more versatile way.
  
The system needed some sort of rules engine to check if there are any discounts available or if any internal costs have gone beyond a threshold. Sitting down with [Macs Dickinson](http://www.macsdickinson.com) and [Shahid Azim](http://hexona.com/) we started to spec out the rules engine. Initial thoughts were that it was going to be a complicated monster, having lots of different customisable rule types to be stored in a database.
However with a couple of well known design patterns we were able to switch that to be a super simple rules engine. With two interfaces and an extension method we were able to create the rules engine. At the lowest level we have Conditions,
  
    public interface ICondition
    {
        public bool IsSatisfied()
    }
  
    public class IsGreaterThan : ICondition
    {
        public IsGreaterThan(decimal threshold, decimal actual)
        {
            _threshold = threshold;
            _actual = actual;
        }
    }
  
And a Rule has a list of conditions plus some setup methods, a validation method and a action method.
  
    public interface IRule<T>
    {
        void ClearConditions();
        void Initialize(T obj);
        bool IsValid();
        T Apply(T obj);
    }
  
    public class MyRule<MyClass> : IRule<MyClass>
    {
        public MyRule()
        {
            Conditions = new List<ICondition>();
        }
  
        public void ClearConditions()
        {
            Conditions.Clear();
        }
  
        public bool IsValid()
        {
            return Conditions.All(x => x.IsSatisfied());
        }
  
        public void Initialize(MyClass obj)
        {
            Conditions.Add(new IsGreaterThan(5, obj.SomeProperty))
        }
  
        public MyClass Apply(MyClass obj)
        {
            obj.SomeProperty = obj.SomeProperty * 0.9; // 10% reduction
        }
  		
         public IList<ICondition> Conditions { get; set; }
    }
  	
The main rule engine is an extension method which can be applied to any class.
  
    public static class RulesEngine
    {
        public static T ApplyRule<T>(this T obj, IRule<T> rule) where T : class
        {
            rule.ClearConditions();
            rule.Initialize(obj);
            if (rule.IsValid())
            {
                rule.Apply(obj);
            }
            return obj;
        }
    }
  
Finally to implement a rule, extend the object you want to apply the rule on.
  
    public class Program
    {
        public void Main()
        {
            var myClass = new MyClass{ SomeProptey = 6.0m };
            myClass.ApplyRule(new MyRule());
        }
    }
  	
You may have noticed the use of the [Strategy Pattern](http://www.oodesign.com/strategy-pattern.html). With the rules being the strategy, encapsulating the different rule logic and the `RulesEngine` executing the common interface to call the different rule algorithms. Using an extension method means we can apply a rule to any class.
  
You can check this out and an example on [github](https://github.com/ritasker/SuperSimple.RulesEngine).
