---
title: "Clean ViewModels with Xamarin.Forms"
date: 2015-01-09
---
I like to keep my view models focused on just the code needed for a particular view, and to keep as much plumbing and infrastructure code out of the view model as possible. I've already covered how to [use Fody to implement INotifyPropertyChanged](http://arteksoftware.com/fody-propertychanged-xamarin-studio/) and use auto properties in our view models. Now I want to cover how to connect the **ViewModel** to the **View** automatically.

The most straight forward way to access our view model would be to simply instantiate it in the constructor of our view.

<pre><code class="language-csharp">using Xamarin.Forms;

namespace CleanViewModels
{	
	public partial class UserPage : ContentPage
	{	
		UserViewModel _viewModel;
		public UserPage ()
		{
			InitializeComponent ();
			_viewModel = new UserViewModel ();
            BindingContext = _viewModel;
		}
	}
}
</code></pre>

This technique requires us to add this bit of code to every view that uses a view model though. I'd prefer to have that handled automatically for each view.

###View Model###

Let's begin by creating a marker interface for our view model. At its simplest, this interface is just used to constrain the generic type later.

```language-csharp
namespace CleanViewModels
{
    public interface IViewModel {}
}
```

Each of our view models will need to implement our marker interface.

<pre><code class="language-csharp">using PropertyChanged;
using System.Windows.Input;
using Xamarin.Forms;

namespace CleanViewModels
{
	[ImplementPropertyChanged]
	public class UserViewModel : IViewModel
	{
		public string FirstName { get; set; }
		public string LastName { get; set; }

		public ICommand LoadUser {
			get {
				return new Command (async () => {
					this.FirstName = "John";
					this.LastName = "Doe";
				});
			}
		}
	}
}
</code></pre>


###View Page###

Now that we have the **ViewModel** defined, we can wire it up to the View. We'll create a new class, named *ViewPage.cs*. This will be our base class for each view.  We will derive **ViewPage** from **ContentPage** and pass in the type of our view model. In the constructor, we will create a new instance of the requested **ViewModel** and set the view's BindingContext to the current **ViewModel**. We also provide a read only property to be able to access the view model in the page, if needed.

<pre><code class="language-csharp">using Xamarin.Forms;

namespace CleanViewModels
{
	public class ViewPage&lt;T&gt; : ContentPage where T:IViewModel, new()
	{
		readonly T _viewModel; 

		public T ViewModel
		{
			get {
				return _viewModel;
			}
		}

		public ViewPage ()
		{
			_viewModel = new T ();
			BindingContext = _viewModel;
		}
	}
}
</code></pre>

###View###

When using XAML in Xamarin.Forms, I was not able to set the root to use **ViewPage&lt;T&gt;** directly. Instead, we will create a wrapper class that defines the <T> parameter for us, so that we can use the wrapper class in XAML.

<pre><code class="language-csharp">namespace CleanViewModels
{	
	public class UserPageBase :  ViewPage&lt;UserViewModel&gt; {}

	public partial class UserPage : UserPageBase
	{	
		public UserPage ()
		{
			InitializeComponent ();
		}
	}
}
</code></pre>

Once the code behind is defined, we can add our xml namespace **local:** and create our view in XAML as **UserPageBase**

```language-markup
<?xml version="1.0" encoding="UTF-8"?>
<local:UserPageBase 
	xmlns="http://xamarin.com/schemas/2014/forms" 
	xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" 
	x:Class="CleanViewModels.UserPage" 
	xmlns:local="clr-namespace:CleanViewModels;assembly=CleanViewModels">

	<ContentPage.Padding>
		<OnPlatform 
			x:TypeArguments="Thickness" 
			iOS="5,20,5,5" 
			Android="5,0,5,5" 
			WinPhone="5,0,5,5" />
	</ContentPage.Padding>

	<StackLayout>
		<Entry Text="{ Binding FirstName, Mode=TwoWay }" />
		<Entry Text="{ Binding LastName, Mode=TwoWay }" />

		<Button 
	      Text="Load User"
	      Command="{Binding LoadUser}"></Button>
    </StackLayout>
</local:UserPageBase>	
```

By using a base view page, along with Fody, we are able to keep our view models clean and focused only on the properties and commands that are needed for the view, thereby increasing the readability of the code.

Check out the sample project on [my Github repo](https://github.com/RobGibbens/CleanViewModels).