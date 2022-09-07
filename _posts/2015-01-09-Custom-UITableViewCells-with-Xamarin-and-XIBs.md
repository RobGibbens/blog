---
title: "Custom UITableViewCells with Xamarin and XIBs"
date: 2015-01-09
---

Android's [List View](http://developer.xamarin.com/guides/android/user_interface/working_with_listviews_and_adapters/) allows us to iterate over an enumerable collection and display each piece of data in a list item. The list view works in conjunction with an adapter to loop over the data and display the data in a layout. We could, and often do, create our own layouts for this purpose. Customizing the layout allows us to match the list view's look and feel to the rest of our app, and to tailor the fields and controls that are shown.

For the times where we just want to display some simple data on the screen though, Android does include some [built-in list item layouts](http://developer.android.com/reference/android/R.layout.html).  I was having a hard time finding any good documentation of these built-in layouts, so I have created a [sample app](https://github.com/RobGibbens/ListViewDemo) displaying as many of the layouts as I could figure out.  The app is written using [Xamarin.Android](http://android.xamarin.com).

All of the code is available from my [Github repo](https://github.com/RobGibbens/ListViewDemo)


###ArrayAdapter with SimpleListItem1###

The simplest adapter to use is the built-in ArrayAdapter, using the built-in Android.Resource.Layout.SimpleListItem1 layout. This layout contains a single TextView, allowing display of a single piece of text.

```language-csharp
using System;
using Android.App;
using Android.OS;
using Android.Widget;

namespace ListViewDemo
{
	[Activity (Label = "ExampleActivity")]
	public class ExampleActivity : ListActivity
	{
		protected override void OnCreate (Bundle savedInstanceState)
		{
			base.OnCreate (savedInstanceState);

			var kittens = new [] { "Fluffy", "Muffy", "Tuffy" };

			var adapter = new ArrayAdapter (
								this, //Context, typically the Activity
								Android.Resource.Layout.SimpleListItem1, //The layout. How the data will be presented 
								kittens //The enumerable data
							);

			this.ListAdapter = adapter;
		}
	}
}
```

The other layouts include (with the controls available)

###Android.Resource.Layout.ActivityListItem###
  - 1 ImageView (Android.Resource.Id.Icon)
  - 1 TextView (Android.Resource.Id.Text1)
{<1>}![](/content/images/2014/Jul/ActivityListItem.png)
  
###Android.Resource.Layout.SimpleListItem1###
  - 1 TextView (Android.Resource.Id.Text1)
{<2>}![](/content/images/2014/Jul/SimpleListItem1.png)

###Android.Resource.Layout.SimpleListItem2###
  - 1 TextView/Title (Android.Resource.Id.Text1)
  - 1 TextView/Subtitle (Android.Resource.Id.Text2)
{<3>}![](/content/images/2014/Jul/SimpleListItem2.png)

###Android.Resource.Layout.SimpleListItemActivated1###
  - 1 TextView (Android.Resource.Id.Text1)
  - Note : Set choice mode to multiple or single
  
```language-csharp
  this.ListView.ChoiceMode = ChoiceMode.Multiple;
```

{<4>}![](/content/images/2014/Jul/SimpleListItemActivated1.png)

###Android.Resource.Layout.SimpleListItemActivated2###
  - 1 TextView (Android.Resource.Id.Text1)
  - 1 TextView/Subtitle (Android.Resource.Id.Text2)
  - Note : Set choice mode to multiple or single
  
```language-csharp
  this.ListView.ChoiceMode = ChoiceMode.Multiple;
```
{<5>}![](/content/images/2014/Jul/SimpleListItemActivated2.png)

###Android.Resource.Layout.SimpleListItemChecked###
  - 1 TextView (Android.Resource.Id.Text1)
  - Note : Set choice mode to multiple or single
  
```language-csharp
  this.ListView.ChoiceMode = ChoiceMode.Multiple;
```
{<6>}![](/content/images/2014/Jul/SimpleListItemChecked.png)

###Android.Resource.Layout.SimpleListItemMultipleChoice###
  - 1 TextView (Android.Resource.Id.Text1)
  - Note : Set choice mode to multiple or single
  
```language-csharp
  this.ListView.ChoiceMode = ChoiceMode.Multiple;
```
{<7>}![](/content/images/2014/Jul/SimpleListItemMultipleChoice.png)

###Android.Resource.Layout.SimpleListItemSingleChoice###
  - 1 TextView (Android.Resource.Id.Text1)
  - Note : Set choice mode to single
  
```language-csharp
  this.ListView.ChoiceMode = ChoiceMode.Single;
```
{<8>}![](/content/images/2014/Jul/SimpleListItemSingleChoice.png)

###Android.Resource.Layout.TestListItem###
  - 1 TextView (Android.Resource.Id.Text1)
{<9>}![](/content/images/2014/Jul/TestListItem.png)
 
###Android.Resource.Layout.TwoLineListItem###
  - 1 TextView/Title (Android.Resource.Id.Text1)
  - 1 TextView/Subtitle (Android.Resource.Id.Text2)
{<10>}![](/content/images/2014/Jul/TwoLineListItem.png)

Again, checkout the Xamarin.Android sample app on my [Github repo](http://github.com/RobGibbens/ListViewDemo) to see these list view layouts in action.

