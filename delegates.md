Delegates
=========

Introduction
------------------
 
 - if you're familiar with C, a delegate is very similar to a function pointer
 - essentially, delegates provide a level of indirection: instead of specifying the behaviour to be executed immitiadely, the behaviour can be "contained" in an object
 - that object can then be used like any other, and one operation you can perform is to execute the encapsulated action
 - alternatively, you can think of a delegate type as a single-method interface, and a delegate instance as an object implementaing that interface.


"Morbid" example
--------------------------

- consider your will - your last will and testament
- it is a set of instructions that you write before your death and leave it in a safe place.
- after your death, an attorney will act on those instructions


- a delegate in C# acts like your will does in the real world - it allows you to specify a sequence of actions to be executed at the appropriat etime.
- delegates are typically used when the code that wants to execute the actions doesn't know the details of what those actions should be.
- for instance, the only reason why the `Thread` class knows what to run in a new thread when you start it is because you provide the constructor with a `ThreadStart` or `ParametrizedThreadStart` delegate instance.

Recipe for simple delegates
----------------------------------------

In order for a delegate to do anything, four things need to happen:

- the *delegate type* needs to be declared
- the code to be executed must be contained in a method
- a *delegate instance* must be created
- the delegate instance must be *invoked*


Declaring the delegate type
---------------------------------------

- a *delegate type* is effectively a list of parameter types and a return type
- it specifies what kind of action can be represented by instances of the type


`delegate void StringProcessor(string input);`

The code says that if you want to create an instance of `StringProcessor`, you'll need a method that takes a `string` parameter and has a `void` return type.


- it is important to understand that `StringProcessor` really is a type, deriving from `System.MulticastDelegate`, which in turn derives from `System.Delegate`. It has methods, you can create instances of it and pass around references to instances, the whole works.
-there are obviously a few special features, but if you're ever stuck wondering what will happen in a particular situation, think about what would hapen if you were using a normal reference type

> **Source of confusion**: the ambiguous term *delegate* - delegates can be misunderstood because the world *delagete* is often used to describe both a *delegate type* and a *delegate instance*. The disctinction between these two is the same as between any other type and instances of that type.


`void PrintString(string x)` - this is an appropriate method for the delegate instance's action

Creating a delegate instance
-----------------------------------------

- now that we hace a delegate type and a method, we can create an instance of that delegate type, specifying that this method be executed when the delegate instance is invoked.

- the for of the expression used to create the delegate instance depends on wether the action uses an sintance method or a static method.

- suppose `PrintString` is a static method in a static class`StaticMethods` and an instance method in a non-static class `InstanceMethods`.


```
StringProcessor processor1, processor2;

processor1 = new StringProcessor(StaticMethods.PrintString);

var instance = new InstanceMethods();
processor2 = new StringProcessor(instance.PrintString);
```

- when the action is an instance method, we need to create an instance of that type (or a derived type)
- this obhect is called the *target* of the action, and when the delegate instance is invoked, **the method will be called on that object.**



> **Garbage collection**: it's worth noting that a delegate instance will prevent its target from being garbage collected if the dlegate instance itself cannot be collected. This can result in an apparent memory leak, particularly when a sort-lived object subscribes to an event in a long-lived object, using itself as the target. The long-lived object indirectly holds a reference to the sort-lived one, prolonging its lifetime.


Invoking a delegate instance
-----------------------------------------

- invoking (synchrounously) a delegate instance is the really easy part: it's just a matter of calling a method in the delegate instance
- the method itself is called `Invoke` and it's always present in a delegate type with the same list of parameters and return type that the delegate type declaration specifies - in our case `void Invoke(string input)`.

- calling `Invoke` will execute the action of the delegate instance, passing on whatever arguments you've specified in the call to `Invoke`, returning the return value of the action

Complete example
---------------------------
```
using System;

delegate void StringProcessor(string input);

public class Person
{
    string name;
    public Person(string name)
    {
        this.name = name;
    }

    public void Say(string message)
    {
        Console.WriteLine("{0} says: {1}", name, message);
    }
}

public class Background
{
    public static void Note(string note)
    {
        Console.WriteLine("({0})", note);
    }
}

public class Program
{
    public static void Main()
    {
        Person jon = new Person("Jon");
        Person tom = new Person("Tom");

        StringProcessor jonsVoice, tomsVoice, background;

        jonsVoice = new StringProcessor(jon.Say);
        tomsVoice = new StringProcessor(tom.Say);

        background = new StringProcessor(Background.Note);

        jonsVoice.Invoke("Hello, son!");
        tomsVoice.Invoke("Hi, dad!");

        background("This is akward!");
    }
}
```

- we declare the delegate type, we create two classes, `Person` and `Background`. Then in `Main`, we create two instances of `Person`, we create three delegate instances of type `StringProcessor` and we assign the instance method `Say` from the two instances to each delegate instance. For the third delegate, we assign it to the static method from the `Background` class.

- then we `Invoke` each delegate: 
	- when `jonsVoice` is invoked, it calls the `Say` method on the `Person` object with the name "Jon", likewise for `tomsVoice`.
	- for the `background` delegate, we used the shorthand way of invoking without explicitly using the `Invoke` method.

- the output is obvious:


```
Jon says: Hello, son!
Tom says: Hi, dad!
(This is akward!)
```


There is a lot of code just for displaying three lines of code. Even if we wanted to use the `Person` and the `Background` classes, there is no real need to use delegates.

- just because we want something to happen (a method to be executed), it doesn't mean we are at the right place and at the right time for it. (remember the will and attorney example!).

- sometimes we need to give instructions - to *delegate* responsability. Often the object that creates a delegate instace is still alive when the delegate instance is invoked. 
- instead, it's about specifying some code to be executed at a particular time, when you may not be able (or want) to change the code that's running at that point. 
- if I want something to happen when a button is clicked, I don't want to have to change the code of the button, I just want to tell the button to call one of my methods, which will take the appropriate action. It's a matter of adding a level of indirection, as so much of the OOP is.

This adds complexity, but also flexibility.



Combining  and removing delegates
---------------------------------------------------

So far, all delegates instances we've looked at have had a single action. In reality, life is a bit more complicated: a delegate instance actually has a list of actions associated woth it called *the invocation list*. 

The static `Combine` and `Remove` methods of the `System.Delegate` type are responsible for creating new delegate instances by respectively splicing together the invocation lists of two delegate instances or removing the invocation list of one delegate instance form another.

> **Delegates are immutable:** Once you've created a delegate instance, nothing about it can be changed. This makes it safe to pass around references to delegate instances and combine them with others without worrying about consistency, thread safety or anyone trying to change their actions.

> `Delegate.Combine` is just like `String.Concat`:  they both combine existing instances to form a new one without changing the originals.

> Note that if you try to combine `null` with a delegate instance, the `null` is treated as if it were a delegate instance with an empty invocation list.

- you will rarely see an explicit call to `Delegate.Combine` in C# code - Usually the `+` and `+=` operatos are used.

- just as you can combine delegate instances, you can remove one from another with the `Delegate.Remove` method, and C# uses the shorthand of the `-` and the `-=` operators in the obvious way.

- when a delegate instance is invoked, **all its actions are executed in order.** 

> **If the delegate's signature has a nonvoid return type, the value returned by `Invoke` is the value returned by the last action executed.** 

> It's rare to see a non-void delegate instance with more thatn one action in its invocation list because it means the return values of all the other actions are never dseen uncless the invoking code explicitly executes the actions one at the time, using `Delegate.GetInvocationList` to fetch the list of actions.

- if any of the actions in the invocation list throws an exception, that prevents any of the subsequent actions from being executed.

- combining and removing delegate instances is particularly useful when it comes to events.

Summary of delegates
--------------------------------

- delegates encapsulate bahaviour with a particular return type and a set of parameters, similar to a single-mthod interface
- the type signature described by a delegate type declaration determines which methods can be used to create delegate instances, and the signature for invocation
- creating a delegate instance requires a method and (for instance methods) a target to call the method on
- delegate instances are immutable
- delegate instances each contain an invocation list - a list of actions
- delegate instances can be combined with and removed from each other
