---
title: "The `partial` keywords in `C#` and ways to make use of it"
date: 2023-04-20 01:00:00
layout: post
---

Even those most of the developers I've talked to know about the `partial` keyword I have rarely seen it used in projects. 
For those who don't know about it, the `partial` keyword allows you to split the functionality across multiple files with some restrictions:
- all the partial pieces need to be in the same assembly
- all the partial pieces need to be in the same namespace


The keyword was introduced back in `2005` as part of [C# language version `2.0`](https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-version-history#c-version-20)
and was mostly seen when relying on designer functionality for your projects. The main use was to auto-generate functionality for the classes that represented your UI like ASCX, ASPX, WPF, WinForms.
You would write some code in your ASCX file and it would automatically run a tool that created <filename>.ascx.cs which contained a partial class with all the server-side component pre-written for you as fields.
  
Fast forward to 2020 and `C# language version 9.0` when the `source generator` feature is implemented and `partial` starts to be more popular.
The source generator functionality assist with build-time code generation as an alternative to reflection based functionalities. A curated list of source generators can be found [here](https://github.com/amis92/csharp-source-generators#source-generators)
  
Let's take a look at libraries like AutoMapper, MediatR or Autofac which rely on `reflection` at runtime to figure out the classes that implement certain marker interfaces so it knows which classes expose the functionality it needs like `Profile` or `IRequestHandler` or just scans entire assemblies to figure out the mappings.
  
The downsides of these libraries is the autoscan and while most of them do support manually specifying the mappings it makes them less interesting for developers.
With code generators, alternatives to these libraries are being developed like [Mapperly](https://github.com/riok/mapperly), [Mediator](https://github.com/martinothamar/Mediator), [PureDI](https://github.com/DevTeam/Pure.DI) that take advantage of Rosyn's capabilities to generate the necessary pieces of code at build time for you while still keeping your code clean (the generated code is actually hidden from you in a separate set of files that are included for you in the project at build time)

The requirement for some of these libraries to work is the use of `partial` classes and attributes to let them know where to add the code for you.
For example the snippet below + the AutoCtor library makes sure to generate the constructor for you where the dependency are injected so you don't have to write it yourself
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
  
This is similar to the `header` functionality from C++. For those who are not familiar in C++ you define a "contract" which is the header file where you specify the definition of your class (including the methods and their parameters) and then in your `cpp` file you define the actual code for each method
While this is annoying to do for every single class, it can be beneficial for things like `controller` classes where the attributes for routing, authentication and swagger code generation polute the class.
Again we rely on `file nesting` and create a file called `MyController.cs` and one called `MyController.Implementation.cs` and we define partial methods in the main class with their attributes and we keep the code in the implementation class

The end result looks like this:

![Partial class contract separation](/assets/image_contractseparating_partial_classes.png)
