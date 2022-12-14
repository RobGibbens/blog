---
title: "Adding a Windows Phone Project to an Existing Xamarin.Forms Solution"
date: 2015-01-09
tags: xamarin-forms
---
> <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24"><path d="M13 17.5a1 1 0 11-2 0 1 1 0 012 0zm-.25-8.25a.75.75 0 00-1.5 0v4.5a.75.75 0 001.5 0v-4.5z"></path><path fill-rule="evenodd" d="M9.836 3.244c.963-1.665 3.365-1.665 4.328 0l8.967 15.504c.963 1.667-.24 3.752-2.165 3.752H3.034c-1.926 0-3.128-2.085-2.165-3.752L9.836 3.244zm3.03.751a1 1 0 00-1.732 0L2.168 19.499A1 1 0 003.034 21h17.932a1 1 0 00.866-1.5L12.866 3.994z"></path></svg> **Note**
> This blog is _woefully_ out of date, and is here simply as an archive

So, you've fired up Xamarin Studio, created a new Xamarin.Forms app for iOS and Android and your app is super successful. Now that the two major platforms are coded and deployed to their respective app stores, you'd like to add Windows Phone as well.  

This is where Xamarin and particularly Xamarin.Forms really shines. It's possible to have your app running on a whole new platform in a matter of minutes.

Xamarin Studio does not support Windows Phone projects, so in order to create a new Windows Phone project, we will need to be using Xamarin Business Edition in Visual Studio on Windows.  The *.sln* file is exactly the same between Xamarin Studio and Visual Studio and can be opened in either IDE. Simply commit your solution to source control on your Mac, pull it down on Windows, and open it in Visual Studio.

#### Add Windows Phone Project

> Add Project -> Blank App (Windows Phone Silverlight)

At this point, we will only have iOS and Android projects so the first step is to add a Windows Phone project. 

![Initial solution explorer](/blog/docs/assets/InitialSolutionExplorer.PNG)

Right click on the solution and choose **Add New Project**. From the dialog, choose Visual C# -> Store Apps -> Windows Phone Apps -> **Blank App (Windows Phone Silverlight)**

![Add new project](/blog/docs/assets/AddNewProject.PNG)

Create the project in the existing solution folder, and give the project the extension *.WinPhone* (this is optional, but adheres to what the Xamarin.Forms wizard would have created).

![Project name](/blog/docs/assets/ProjectName.PNG)

Next, we'll get a wizard dialog asking which version of the Windows Phone SDK we want to use. The Xamarin.Forms wizard creates a Windows Phone 8.0 project, so we'll choose that same version here.

![Target version](/blog/docs/assets/TargetVersion.PNG)

This will create the Windows Phone project for us and add it to the solution.

![New solution explorer](/blog/docs/assets/NewSolutionExplorer.PNG)

#### Add Xamarin.Forms

Now that the project is created, all we need to do is connect the Windows Phone project to our Xamarin.Forms app. This has two steps. First, we need to add the Xamarin.Forms Nuget packages to our Windows Phone project.  When the Nuget package is installed, it creates a new folder named Toolkit.Content. We need to make sure that the files in this folder are set to Content/Copy Always.

> Install-Package Xamarin.Forms

![Nuget](/blog/docs/assets/Nuget.PNG)

![Content copy always](/blog/docs/assets/ContentCopyAlways.PNG)

Next, we need to add a reference to our shared code project, which will be either a PCL or a Shared Project. This will allow us to access all of the Xamarin.Forms UI that we have already created.

![Add reference](/blog/docs/assets/AddReference.PNG)

#### Edit MainPage.xaml

The last task that we need to do is to change the XAML and the C# that was generated by default.  With Xamarin.Forms, we won't be defining our UI in the platform specific projects, but we do need the MainPage to initialize the Xamarin.Forms framework for us.

All we need in the *MainPage.xaml* file is the following XAML.

```xml
<phone:PhoneApplicationPage
  x:Class="SuperSuccessfulApp.WinPhone.MainPage"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:phone="clr-namespace:Microsoft.Phone.Controls;assembly=Microsoft.Phone"
  xmlns:shell="clr-namespace:Microsoft.Phone.Shell;assembly=Microsoft.Phone"
  xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
  xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
  mc:Ignorable="d"
  FontFamily="{StaticResource PhoneFontFamilyNormal}"
  FontSize="{StaticResource PhoneFontSizeNormal}"
  Foreground="{StaticResource PhoneForegroundBrush}"
  SupportedOrientations="Portrait" Orientation="Portrait"
  shell:SystemTray.IsVisible="True">
</phone:PhoneApplicationPage>
```

> Note : Be sure to change the x:Class attribute to your namespace

In the *MainPage.xaml.cs* file, add the following C#.

```csharp
using Microsoft.Phone.Controls;
using Xamarin.Forms;

namespace SuperSuccessfulApp.WinPhone
{
  public partial class MainPage : PhoneApplicationPage
  {
    public MainPage()
    {
      InitializeComponent();

      Forms.Init();
      Content = SuperSuccessfulApp.App.GetMainPage().ConvertPageToUIElement(this);
    }
  }
}
```

> Note : Be sure to change the relevant namespaces

At this point, we have a functioning Windows Phone app, leveraging all of the work that we previously did for iOS and Android. The only remaining tasks would be to implement any custom renderers and/or platform specific interfaces, and add the appropriate icons and images.
