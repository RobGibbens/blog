---
title: "Xamarin Insights : Unobtrusive mobile analytics"
date: 2015-01-09
---
At the [Xamarin Evolve 2014](https://evolve.xamarin.com) conference, Xamarin announced their new mobile analytics solution, [Xamarin Insights](https://insights.xamarin.com). When you add Insights to your mobile application, you can start tracking exceptions, crashes, user identities, and application events. 

One of the best features of Insights is the ability to track the user's actions through the app to be able to trace the conditions that led to an exception. All too often, users will experience an error or, even worse, an app crash. Rarely, these users will email the developer and tell them about the crash, but don't remember any useful information that would help to recreate the exception. By adding calls **Xamarin.Insights.Track()** to our methods, we can track the events leading up to a particular crash.

### Typical Usage ###

```language-csharp
public async Task GetData ()
{
	Xamarin.Insights.Track ("Enter GetData");
	
	/* Implement Method */

	Xamarin.Insights.Track ("Exit GetData");
}

private async Task GetLocalData ()
{
	Xamarin.Insights.Track ("Enter GetLocalData");
	
	/* Implement Method */

	Xamarin.Insights.Track ("Exit GetLocalData");
}

private async Task GetRemoteData ()
{
	Xamarin.Insights.Track ("Enter GetRemoteData");
	
	/* Implement Method */

	Xamarin.Insights.Track ("Exit GetRemoteData");
}
```

While this *does* work, it adds a lot of unnecessary noise to the code.  Instead of adding line after line of analytics tracking to every method, I prefer to get that boilerplate code out of the way, and let the code focus on the problem at hand.

As [I've](http://arteksoftware.com/clean-viewmodels-with-xamarin-forms) [shown](http://arteksoftware.com/end-to-end-mvvm-with-xamarin) [before](http://arteksoftware.com/fody-propertychanged-xamarin-studio), I get a lot of use out of [Fody](https://github.com/Fody/Fody) in my mobile apps. Fody allows us to control the build time compilation and change the outputted assembly.  In this case, we can use Fody's MethodDecorator package to move the Insights tracking logic into a method attribute.

### Adding Fody ###

We'll base our custom attribute on Fody's MethodDecorator attribute, which we can add to our project from [Nuget](http://www.nuget.org/packages/MethodDecoratorEx.Fody).

> Install-Package MethodDecoratorEx.Fody

> NOTE : There are two Fody MethodDecorators on Nuget. I'm using MethodDecoratorEx

You'll need to add the declaration to the FodyWeavers.xml file as well.

```language-markup
<?xml version="1.0" encoding="utf-8" ?>  
<Weavers>
	<MethodDecoratorEx />
</Weavers>
```

[I wrote up instructions on using Fody with Xamarin Studio](http://arteksoftware.com/fody-propertychanged-xamarin-studio)

### Create Attribute ###

Next, we'll create a custom attribute to wrap up the Insights code. Each time we enter or leave a method, we'll make a call to **Xamarin.Insights.Track()**.  

```language-csharp
using System;
using System.Reflection;
using MethodDecoratorInterfaces;
using ArtekSoftware.Demos;
using Xamarin;

[module: Insights]

namespace ArtekSoftware.Demos
{
	[AttributeUsage (
    		AttributeTargets.Method 
            | AttributeTargets.Constructor 
            | AttributeTargets.Assembly 
            | AttributeTargets.Module)]
	public class InsightsAttribute : Attribute, IMethodDecorator
	{
		private string _methodName;

		public void Init (object instance, MethodBase method, object[] args)
		{
			_methodName = method.DeclaringType.FullName + "." + method.Name;
		}

		public void OnEntry ()
		{
			var message = string.Format ("OnEntry: {0}", _methodName);
			Insights.Track (message);
		}

		public void OnExit ()
		{
			var message = string.Format ("OnExit: {0}", _methodName);
			Insights.Track (message);
		}

		public void OnException (Exception exception)
		{
			Insights.Report (exception);
		}
	}
}
```

### Add Attribute to Methods ###

Once the attribute is defined, all we need to do is decorate the methods that we want to track.  Notice that the implementation of each method is focused simply on the method's logic and not Insights tracking.

```language-csharp
[Insights]
public async Task GetData ()
{
	/* Implement Method */
}

[Insights]
private async Task GetLocalData ()
{
	/* Implement Method */
}

[Insights]
private async Task GetRemoteData ()
{
	/* Implement Method */
}
```

### Performance ###

This may seem like a lot of extra overhead, especially since we're calling out to a remote server. I asked the Xamarin Insights team about this, and got the following answers.

> - When does Insights send its data?
    - If Insights detects a wifi connection then generally we feel free to send data as often as we like, if we are on a Cellular connection we wait a very long time before sending data
- Does Insights.Track immediately call the server, or are the calls batched up?
    - All data is batched up for a few seconds before sending out to the server
- Do Insights.Track and Insights.Report call the server asynchronously, or are these blocking calls?
	- All API calls are essentially async, any Insights activity happens in a background thread
- If queued, does the queue persist across restarts of the app? Restarts of the device?
    - All insights data is journaled to disk, this means track/identify/report/crashes/everything is persistent across restarts. We send out the old data whenever we have a good opportunity to do so, usually after we send out some new data successfully.

### Results ###

The end result of this is that we get really detailed tracking of the events that lead to an exception. This will make finding and fixing errors in our apps faster and more efficient.  We get the details of constant tracking without littering our code with tracking calls.

{<1>}![](/content/images/2014/Oct/Insights-Events.png)
