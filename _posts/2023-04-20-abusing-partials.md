---
title: "The partial keywords in C# and ways to make use of it"
date: 2023-04-20 01:00:00
layout: post
---

Even though most developers I've talked to know about the `partial` keyword I have rarely seen it used in projects. 
For those who don't know about it, the `partial` keyword allows you to split the functionality across multiple files with some restrictions:
- all the partial pieces need to be in the same assembly
- all the partial pieces need to be in the same namespace


The keyword was introduced back in `2005` as part of [C# language version `2.0`](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-version-history#c-version-20)
and was mostly seen when relying on designer functionality for your projects. The main use was to auto-generate functionality for the classes that represented your UI like ASCX, ASPX, WPF, WinForms.
You would write some code in your ASCX file and it would automatically run a tool that created <filename>.ascx.cs which contained a partial class with all the server-side component pre-written for you as fields.
  
Fast forward to 2020 and [C# language version `9.0`](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#support-for-code-generators) when the `source generator` feature is implemented and `partial` starts to be more popular.
The source generator functionality assist with build-time code generation as an alternative to reflection based functionalities. A curated list of source generators can be found [here](https://github.com/amis92/csharp-source-generators#source-generators)
  
Let's take a look at libraries like AutoMapper, MediatR or Autofac.
They heavily rely on `reflection` at runtime to figure out the classes that implement certain marker interfaces so it knows which classes expose the functionality it needs like `Profile` or `IRequestHandler` or just scans entire assemblies to figure out the mappings.
  
The downsides of these libraries is the autoscan, and, while most of them do support manually specifying the mappings, it makes them less interesting for developers.
  
However, with the introduction of code generators, alternatives to these libraries are being developed. Libraries like [Mapperly](https://github.com/riok/mapperly), [Mediator](https://github.com/martinothamar/Mediator), [PureDI](https://github.com/DevTeam/Pure.DI) take advantage of Rosyn's capabilities to generate the necessary pieces of code at build time, while still keeping your code clean. The generated code is actually hidden from you in a separate set of files that are included for you in the project at build time - read more [here](https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview)

The requirement for some of these libraries to work, though, is the use of `partial` classes and attributes to let them know where to add the code for you.

For example, the snippet below along with the AutoCtor library, makes sure to generate the constructor for you, where the dependency are injected. That way you don't have to write it yourself
  
{% highlight csharp %}
[AutoConstruct]
public partial class ExampleClass
{
    private readonly ICustomService _customService;
}
{% endhighlight %}
  
gets built into
  
{% highlight csharp %}
partial class ExampleClass
{
    public ExampleClass(ICustomService customService)
    {
        _customService = customService;
    }
}
{% endhighlight %}
  
## How can `partial` actually help organize your code
  
There are several usages for partials. You can find an example in my video [here](https://www.youtube.com/watch?v=35VgRuw_vJw&t=1918s) but I will go into more details in here.
  
> **_NOTE:_**  Did you know the `partial` keyword works also for `methods` and not just `classes` ?
  
### Usage 1: Easier navigation inside your controllers
  
By relying on 2 features from VS Code or Visual Studio we can increase the readability of our controller classes.
First, we mark the class as partial so we can spread each method in its own file. Second, we rely on `file nesting` to group the files together into 1 nice hierarchy

The end result looks like this:

![File nested partial classes](/assets/image_filenesting_partial_classes.png)
  
### Usage 2: Separating your contract attributes from the rest of the code
  
This is similar to the `header` functionality from C++. 
For those who are not familiar with it, in C++ you define a "contract" which is the `header` file and it only contains the definition of your class (methods and their parameters), and then, in your `cpp` file, you define the actual code for each method
  
While this is annoying to do for every single class, it can be beneficial for things like `controller` classes where the attributes for routing, authentication and swagger code generation polute the class.
  
Again, we rely on `file nesting` and create a file called `MyController.cs` where we define a partial controller class with its attributes and partial methods and their attributes and then add another class file called `MyController.Implementation.cs` where we keep the actual implementation of the class & methods

The end result looks like this:

![Partial class contract separation](/assets/image_contractseparating_partial_classes.png)

As you can see this makes it a lot easier to:
  - focus on the actual implementation of the method
  - see the method routes without having to go to each partial class where a method is defined (in case you also separate your class by method)
  - read the routes & configuration of your routes
  - modify the contract separately from the implementation - less conflicts during merge
  - keep the code clean by ensuring each file has a purpose - one defines the contract while the other defines the actual code. Similar to `interfaces`
  
Let me know in the comment section below if you find these usages useful or if you have any concerns about applying them to your project 👍
