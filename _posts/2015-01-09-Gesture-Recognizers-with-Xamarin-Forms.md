---
title: "Gesture Recognizers with Xamarin.Forms"
date: 2015-01-09
---
iOS and Android apps both provide a way to wire up a UI control to listen for specific touch events. These events are called gestures. Typically, we would use this to listen for a tap (click) gesture, or maybe a swipe or fling gesture.

In a native Xamarin.iOS or Xamarin.Android app, wiring up these events is fairly straight forward. As of this writing though (July, 2014), [Xamarin.Forms only has cross platform support for the tap gesture](http://developer.xamarin.com/guides/cross-platform/xamarin-forms/working-with/gestures/). Just because all of the gesture recognizers aren't wrapped for Xamarin.Forms does not mean that we can not use them though. By using [custom renderers](http://developer.xamarin.com/guides/cross-platform/xamarin-forms/custom-renderer/), we are able to get access to the native controls and wire up our gesture recognizer just as we would in a native Xamarin app.

I have created a sample app in my [Github repo](https://github.com/RobGibbens/XamarinFormsGestureRecognizers) to demonstrate how we can use the simplicity of Xamarin.Forms and still have the power of built in gesture recognizers.

###Sample App###
We'll start with a new Xamarin.Forms app. This can be a Shared Project or a Portable project (the sample app is a Shared Project). For this sample, we're going to wire up a **Label** control to listen for the touch events.

To be able to use a custom renderer, we'll need to subclass **Label** in our shared code.

```language-csharp
using Xamarin.Forms;

namespace XamarinFormsGestureRecognizers
{
	public class FancyLabel : Label {}
}
```

In our *App.cs* we'll set the Content of our **Page** to a new **FancyLabel**. Since the **TapGestureRecognizer** is already built in to Xamarin.Forms, we'll wire that one up here in our shared code.

```language-csharp
using Xamarin.Forms;
using System;

namespace XamarinFormsGestureRecognizers
{
	public static class App
	{
		public static Page GetMainPage ()
		{	
			var fancyLabel = new FancyLabel {
				Text = "Hello, Forms!",
				VerticalOptions = LayoutOptions.CenterAndExpand,
				HorizontalOptions = LayoutOptions.CenterAndExpand,
			};

			var tapGestureRecognizer = new TapGestureRecognizer ();
			tapGestureRecognizer.Tapped += (sender, e) => Console.WriteLine ("Tapped");
			fancyLabel.GestureRecognizers.Add (tapGestureRecognizer);

			return new ContentPage { 
				Content = fancyLabel
			};
		}
	}
}
```

Now we need to implement our custom renderers for each platform.

####iOS####
We'll add a new file named *FancyIosLabelRenderer.cs* to the iOS project and add a class derived from Xamarin.Forms' **LabelRenderer**.

```language-csharp
using Xamarin.Forms;
using Xamarin.Forms.Platform.iOS;
using MonoTouch.UIKit;
using System;
using XamarinFormsGestureRecognizers;
using XamarinFormsGestureRecognizers.iOS;

namespace XamarinFormsGestureRecognizers.iOS
{
	public class FancyIosLabelRenderer : LabelRenderer
	{
		protected override void OnElementChanged (ElementChangedEventArgs<Label> e)
		{
			base.OnElementChanged (e);
		}
	}
}
```

The renderer class itself is a view that wraps the native control. While we could wire up the recognizer to the control itself, it's better to simply wire it up to the renderer's view. Since the renderers can be reused, we also need to be sure to only create the gesture recognizers if the OldElement is null (when we're not reusing the control).

```language-csharp
public class FancyIosLabelRenderer : LabelRenderer
	{
		UILongPressGestureRecognizer longPressGestureRecognizer;
		UIPinchGestureRecognizer pinchGestureRecognizer;
		UIPanGestureRecognizer panGestureRecognizer;
		UISwipeGestureRecognizer swipeGestureRecognizer;
		UIRotationGestureRecognizer rotationGestureRecognizer;

		protected override void OnElementChanged (ElementChangedEventArgs<Label> e)
		{
			base.OnElementChanged (e);

			longPressGestureRecognizer = new UILongPressGestureRecognizer (() => Console.WriteLine ("Long Press"));
			pinchGestureRecognizer = new UIPinchGestureRecognizer (() => Console.WriteLine ("Pinch"));
			panGestureRecognizer = new UIPanGestureRecognizer (() => Console.WriteLine ("Pan"));
			swipeGestureRecognizer = new UISwipeGestureRecognizer (() => Console.WriteLine ("Swipe"));
			rotationGestureRecognizer = new UIRotationGestureRecognizer (() => Console.WriteLine ("Rotation"));

			if (e.NewElement == null) {
				if (longPressGestureRecognizer != null) {
					this.RemoveGestureRecognizer (longPressGestureRecognizer);
				}
				if (pinchGestureRecognizer != null) {
					this.RemoveGestureRecognizer (pinchGestureRecognizer);
				}
				if (panGestureRecognizer != null) {
					this.RemoveGestureRecognizer (panGestureRecognizer);
				}
				if (swipeGestureRecognizer != null) {
					this.RemoveGestureRecognizer (swipeGestureRecognizer);
				}
				if (rotationGestureRecognizer != null) {
					this.RemoveGestureRecognizer (rotationGestureRecognizer);
				}
			}

			if (e.OldElement == null) {
				this.AddGestureRecognizer (longPressGestureRecognizer);
				this.AddGestureRecognizer (pinchGestureRecognizer);
				this.AddGestureRecognizer (panGestureRecognizer);
				this.AddGestureRecognizer (swipeGestureRecognizer);
				this.AddGestureRecognizer (rotationGestureRecognizer);
			}
		}
	}
```

Lastly, we need to be sure that we wire up the custom renderer to look for instances of the **FancyLabel** class.

```language-csharp
[assembly: ExportRenderer (typeof(FancyLabel), typeof(FancyIosLabelRenderer))]
```

####Android####
Using the gesture recognizers in Android will be similar, but with a bit more work. [Xamarin's documentation](http://developer.xamarin.com/recipes/android/other_ux/gestures/create_a_gesture_listener/) explains how to use a **GestureDetector** in an Android app, which is what we'll do here.

Once again, we'll add a new file called *FancyAndroidLabelRenderer.cs* to hold our custom renderer.
```language-csharp
using Xamarin.Forms.Platform.Android;
using Xamarin.Forms;
using XamarinFormsGestureRecognizers;
using XamarinFormsGestureRecognizers.Droid;
using Android.Views;

namespace XamarinFormsGestureRecognizers.Droid
{
	public class FancyAndroidLabelRenderer : LabelRenderer
	{
		public FancyAndroidLabelRenderer ()
		{
		}

		protected override void OnElementChanged (ElementChangedEventArgs<Label> e)
		{
			base.OnElementChanged (e);
		}
	}
}
```

We'll also add a class named *FancyGestureListener.cs* and we'll add gesture listener. Here we'll subclass the built in **SimpleOnGestureListener** to provide most of the functionality.

```language-csharp
using Android.Views;
using System;

namespace XamarinFormsGestureRecognizers.Droid
{
	public class FancyGestureListener : GestureDetector.SimpleOnGestureListener
	{
		public override void OnLongPress (MotionEvent e)
		{
			Console.WriteLine ("OnLongPress");
			base.OnLongPress (e);
		}

		public override bool OnDoubleTap (MotionEvent e)
		{
			Console.WriteLine ("OnDoubleTap");
			return base.OnDoubleTap (e);
		}

		public override bool OnDoubleTapEvent (MotionEvent e)
		{
			Console.WriteLine ("OnDoubleTapEvent");
			return base.OnDoubleTapEvent (e);
		}

		public override bool OnSingleTapUp (MotionEvent e)
		{
			Console.WriteLine ("OnSingleTapUp");
			return base.OnSingleTapUp (e);
		}

		public override bool OnDown (MotionEvent e)
		{
			Console.WriteLine ("OnDown");
			return base.OnDown (e);
		}

		public override bool OnFling (MotionEvent e1, MotionEvent e2, float velocityX, float velocityY)
		{
			Console.WriteLine ("OnFling");
			return base.OnFling (e1, e2, velocityX, velocityY);
		}

		public override bool OnScroll (MotionEvent e1, MotionEvent e2, float distanceX, float distanceY)
		{
			Console.WriteLine ("OnScroll");
			return base.OnScroll (e1, e2, distanceX, distanceY);
		}

		public override void OnShowPress (MotionEvent e)
		{
			Console.WriteLine ("OnShowPress");
			base.OnShowPress (e);
		}

		public override bool OnSingleTapConfirmed (MotionEvent e)
		{
			Console.WriteLine ("OnSingleTapConfirmed");
			return base.OnSingleTapConfirmed (e);
		}
	}
}
```
Then, we need to wire up this listener to our renderer, and include the **ExportRenderer** attribute.

```language-csharp
using Xamarin.Forms.Platform.Android;
using Xamarin.Forms;
using XamarinFormsGestureRecognizers;
using XamarinFormsGestureRecognizers.Droid;
using Android.Views;

[assembly: ExportRenderer (typeof(FancyLabel), typeof(FancyAndroidLabelRenderer))]

namespace XamarinFormsGestureRecognizers.Droid
{
	public class FancyAndroidLabelRenderer : LabelRenderer
	{
		private readonly FancyGestureListener _listener;
		private readonly GestureDetector _detector;

		public FancyAndroidLabelRenderer ()
		{
			_listener = new FancyGestureListener ();
			_detector = new GestureDetector (_listener);

		}

		protected override void OnElementChanged (ElementChangedEventArgs<Label> e)
		{
			base.OnElementChanged (e);

			if (e.NewElement == null) {
				if (this.GenericMotion != null) {
					this.GenericMotion -= HandleGenericMotion;
				}
				if (this.Touch != null) {
					this.Touch -= HandleTouch;
				}
			}

			if (e.OldElement == null) {
				this.GenericMotion += HandleGenericMotion;
				this.Touch += HandleTouch;
			}
		}

		void HandleTouch (object sender, TouchEventArgs e)
		{
			_detector.OnTouchEvent (e.Event);
		}

		void HandleGenericMotion (object sender, GenericMotionEventArgs e)
		{
			_detector.OnTouchEvent (e.Event);
		}
	}	
}
```

Checkout the sample app on my [Github repo](https://github.com/RobGibbens/XamarinFormsGestureRecognizers) to get started with Xamarin.Forms and gestures.
