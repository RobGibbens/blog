---
title: "IoC Containers with Xamarin"
date: 2015-01-09
---
When writing cross platform apps with Xamarin, our goal is share as close to 100% of our code across all the platforms. While this is an admirable goal to aim for, it is not realistic. Very often, we find ourselves needing to access a platform specific feature from our shared code.  We have multiple options in this case. We could use a Shared Project with compiler directives, class mirroring, or partial classes and access the platform code alongside our shared code. Alternatively we could use an abstraction for the functionality in our shared code, and pass an implementation of the abstraction into our shared code. In this way, our shared code only needs to know about the abstraction, typically an interface. This strategy, known as Inversion of Control or IoC, works especially well when using Portable Class Libraries (PCL) for our shared code. 

Even when using IoC, manually managing all of our dependencies, including instantiating the dependency tree for an object, can be tedious at best. This is where we turn to a host of existing IoC containers. The .net ecosystem has long had a wealth of choices in IoC containers. Some of the more popular ones include [StructureMap](https://github.com/structuremap/structuremap), [Castle Windsor](http://docs.castleproject.org/Default.aspx?Page=MainPage&NS=Windsor&AspxAutoDetectCookieSupport=1), [Ninject](http://www.ninject.org/), [Unity](https://unity.codeplex.com/), and [Autofac](http://autofac.org/). These are by no means the only choices, up to and including rolling our own. 

Not all of these containers are able to run in limitations imposed by mobile devices though. Phones and tablets have constrained cpu and memory, and iOS devices forbid JIT compiling and certain uses of reflection. Additionally, the library authors have to specifically compile for Xamarin.iOS and Xamarin.Android, either individually or as part of a PCL.

I decided to put together some sample code showing how the Xamarin-compatible IoC containers work. As of this writing (July, 2014) I have limited the comparison only to IoC containers that 1) I could get to work successfully and 2) had an available Nuget package for easy installation. I did not want to go through the process of pulling individual repositories and building the source from scratch, although that is a valid option. This excluded some options that I do want to mention as alternatives.

- [Xamarin.Forms Dependency Service](http://developer.xamarin.com/guides/cross-platform/xamarin-forms/dependency-service/). This is really more of a Service Locator than it is an IoC container. Also, it is only available as part of Xamarin.Forms.
- [OpenNetCF](http://ioc.codeplex.com/). There is no nuget package for this library. Also, it requires custom attributes be added to the shared code, diminishing the usefulness.
- [XPlatUtils](https://github.com/jonathanpeppers/XPlatUtils). There is no nuget package for this library.

The libraries that I focused on were [Autofac](http://www.nuget.org/packages/Autofac/), [MvvmCross](http://www.nuget.org/packages/MvvmCross.HotTuna.CrossCore), [Ninject](http://www.nuget.org/packages/Portable.Ninject), [TinyIoc](http://www.nuget.org/packages/TinyIoC/), and [Unity](http://www.nuget.org/packages/Unity).

All of the code is available from my [Github Repo](https://github.com/RobGibbens/Xamarin.IoC)

##Sample Project##

In the sample project, we have a single IoCDemo.Core project. This project contains the interface abstractions for our platform specific projects (ISettings and IPlatform) and a concrete ViewModel (MainViewModel) which takes the two interfaces as constructor dependencies. For each library, I created an iOS and an Android project to demonstrate wiring up the dependencies to platform specific implementations and creating the view model. Each container will be wired up in an App.cs file in each platform.

> Some of the IoC containers have the ability to scan your assemblies and automatically wire up your dependecies. I chose **not** to use this ability. In a mobile app, every bit of cpu power is precious. I would rather spend the extra few seconds to write the code to wire up the dependency once at development time than have the app scan the assemblies every single time it is started.

###Autofac###
> Install-Package Autofac
####Wiring up the container####

```language-csharp
using Autofac;
using IoCDemo.Core;

namespace AutoFacDemo.iOS
{
	public class App
	{
		public static IContainer Container { get; set; }

		public static void Initialize()
		{
			var builder = new ContainerBuilder();

			builder.RegisterInstance(new ApplePlatform()).As<IPlatform>();
			builder.RegisterInstance(new AppleSettings()).As<ISettings>();
			builder.RegisterType<MainViewModel> ();

			App.Container = builder.Build ();
		}
	}
}
```

####Resolving the view model####

```language-csharp
MainViewModel viewModel = null;

using (var scope = App.Container.BeginLifetimeScope ()) {
	viewModel = App.Container.Resolve<MainViewModel> ();
}
```
{<1>}![](/content/images/2014/Jul/Autofac.png)

###MvvmCross###
> Install-Package MvvmCross.HotTuna.CrossCore
####Wiring up the container####

```language-csharp
using Cirrious.CrossCore;
using IoCDemo.Core;
using Cirrious.CrossCore.IoC;

namespace MvvmCrossDemo.iOS
{
	public static class App
	{
		public static void Initialize ()
		{
			MvxSimpleIoCContainer.Initialize ();
			Mvx.RegisterType<IPlatform, ApplePlatform> ();
			Mvx.RegisterType<ISettings, AppleSettings> ();
		}
	}
}
```

####Resolving the view model###

```language-csharp
var viewModel = Mvx.IocConstruct<MainViewModel> ();
```
{<2>}![](/content/images/2014/Jul/MvvmCross.png)

###Ninject###
> Install-Package Portable.Ninject
####Wiring up the container####
```language-csharp
using Ninject;

namespace NinjectDemo.iOS
{
	public static class App
	{
		public static StandardKernel Container { get; set; }

		public static void Initialize()
		{
			var kernel = new Ninject.StandardKernel(new NinjectDemoModule());			
			
			App.Container = kernel;
		}
	}
}
```

####Resolving the view model###

```language-csharp
var viewModel = App.Container.Get<MainViewModel> ();
```
{<3>}![](/content/images/2014/Aug/Ninject.png)

###TinyIoc###
> Install-Package TinyIoc
####Wiring up the container####

```language-csharp
using TinyIoC;
using IoCDemo.Core;

namespace TinyIoCDemo.iOS
{
	public static class App
	{
		public static void Initialize ()
		{
			var container = TinyIoCContainer.Current;

			container.Register<IPlatform, ApplePlatform> ();
			container.Register<ISettings, AppleSettings> ();
		}
	}
}
```

####Resolving the view model###

```language-csharp
var viewModel = TinyIoC.TinyIoCContainer.Current.Resolve<MainViewModel> ();
```
{<4>}![](/content/images/2014/Jul/TinyIoC.png)

###Unity###
> Install-Package Unity
####Wiring up the container####

```language-csharp
using Microsoft.Practices.Unity;
using IoCDemo.Core;

namespace UnityDemo.iOS
{
	public class App
	{
		public static UnityContainer Container { get; set; }

		public static void Initialize()
		{
			App.Container = new UnityContainer();
			App.Container.RegisterType<IPlatform, ApplePlatform> ();
			App.Container.RegisterType<ISettings, AppleSettings> ();
		}
	}
}
```

####Resolving the view model###

```language-csharp
var viewModel = App.Container.Resolve (typeof(MainViewModel), "mainViewModel") as MainViewModel;
```
{<5>}![](/content/images/2014/Jul/Unity.png)

Again, checkout the sample app on my [Github repo](https://github.com/RobGibbens/Xamarin.IoC) to compare our IoC container choices for Xamarin.