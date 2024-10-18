---
title: "Navigation Component — Common issues we can face"
tags:
  - android
  - jetpack-compose
  - navigation
date: 2020-05-14
layout: post
---
While I was playing with Navigation Component, I’ve faced some concerns, concerns that could lead to issues for some. I will be straightforward and will start with my first concern:

## 1- The one with Shared ViewModel

Navigation Component recommends Single Activity-Multiple Fragments approach and combining this approach with Shared viewmodels raised a concern in my mind.

To be explicit, let’s say a fragment needs to fetch some data that can take time. If we use a shared viewmodel for this task,
a question comes to my mind, what if we are not in that fragment anymore, how can we cancel it? Surely, [it is a nice 
practice to cancel unneeded tasks](https://www.youtube.com/watch?v=pzfzz50W5Uo) and if we think for a second that we can clear those tasks in onClear method, 
then this concern of me will lead to an issue. Since this viewmodel is shared, it means that, 
onCleared will be called only when host activity is killed and this activity won’t be killed for a long time if we use too many fragments.

As a result, long running tasks, that initially started from fragments, won’t be cancelled even though 
when we are not inside of those fragments and it is something we’d like avoid as possible as we can **#perfmatters**

Then I started thinking. Android provides an extension function for back press dispatcher:

<script src="https://gist.github.com/melomg/57dd8d1a567549fe88b5b46e7d83ed98.js"></script>

Could I cancel those tasks when user go back from fragments by listening this dispatcher? 
This surely solves cancelling them when users go back from that fragment but it doesn’t solve the problem when users go 
to next destination fragment. Our aim was to cancel those long running tasks as soon as users leave the fragment.

Second idea was to pass `lifecycleScope` to the viewmodel methods that starts those tasks. 
If we use `lifecycleScope` to launch coroutines for those tasks, they will be cancelled when lifecycle of fragments 
are destroyed which is the thing we wished to solve. But passing `lifecycleScope` to viewmodel’s methods is not something 
we can enforce to each developer on team, it would make testing difficult and it is very likely that developers 
will forget and use `viewModelScope` instead. Our mantra is to achieve [Seiketsu](https://en.wikipedia.org/wiki/5S_(methodology)#Standardize_(seiketsu)).

Third idea came from reading [the documentation](https://developer.android.com/guide/navigation/navigation-programmatic#share_ui-related_data_between_destinations_with_viewmodel)
which is something I should’ve done first. It says we can create nested graphs and share the viewmodel within the graph 🎊. 
It might be a good idea to create nested graphs if we have huge graph, but this solution may or may not postpone
our issue hence these little nested graphs can also grow to be huge and graph scoped viewmodel objects 
will be still alive till navigation graph itself is cleared.

At the end, I stepped back and come to the decision that it might be the best practice if **each fragments 
would have their own viewmodels to handle those long running tasks and if something needs to be shared between 
those fragments, they could’ve second viewmodel which is shared(scoped) inside the navigation graph.**

## 2- The one with Custom Toolbar

There are 3 cases about how we handle toolbars.

    1- One app action bar for activity
    2- Different toolbars in each fragments as action bars
    3- Different toolbars in each fragments, not as action bars

Option 1 or 2 should be our first choice and [Android Guide page shows how to configure these options with navigation component](https://developer.android.com/guide/navigation/navigation-ui),
but if we have custom icons or custom views that we don’t have time to refactor for the time being and 
if we moved forward with 3, then we shouldn’t forget to handle system back button to go to the desired destination.

I’d suggest to add callback to `BackPressedDispatcher` by giving `viewLifecycleOwner` since 
this will ensure that the callback will only be added when the Lifecycle is started, and 
it will be removed automatically when fragment destroyed.

<script src="https://gist.github.com/melomg/57dd8d1a567549fe88b5b46e7d83ed98.js"></script>

## 3- The one with Config Changes

Sometimes in order to simplify things, we tend to move logics to base classes. 
Let’s say, we tried to abstract `navController.navigate()` method from fragments that extends base fragment, and 
we cached `navController` in base fragment. This might work for most of the fragments 
except fragments which instances are retained across Activity re-creation(via [setRetainInstance](https://developer.android.com/reference/android/app/Fragment#setRetainInstance(boolean))). 
If we have a retained fragment, it will keep same reference for `NavController`, and it will use this controller for navigation.

But the problem is that, parent of this fragment is a `NavHostFragment` which is not retained and 
when orientation change happened, a new `NavController` instance will be returned, thus, we need to use this new 
`NavController` not our cached `NavController`. **So if we need to get `NavController` to navigate, we should always 
call `findNavController()` in fragments, or whenever we’d like to navigate, we can use livedata in a shared viewmodel
which is observed in activity so that only activity will be responsible for navigation.**
I’d personally choose the second approach since livedata is lifecycle aware and I would be controlling navigation from one place.

Plus, using livedata can also save us from some exceptions like `Fragment X not attached to an activity`
(with new exception message → `Fragment X not associated with a fragment manager`) and from 
`Navigation destination X is unknown to this NavController`.

## 4- The one with Shared ViewModel again (part 2)

Another concern of mine has risen by having too many fragments with too many viewmodels. 
Having many nested graphs can lead to have many shared viewmodels. Thus, this would make it hard to 
distinguish which viewmodels are shared and which ones are not. I believe, our problem can be solved 
if we use **Shared** prefix naming for shared viewmodels.

Using base viewmodel for navigation can look simple and rather tempting, but I prefer chosing composition 
over inheritance. **Composition + Interface** delegation would be heaven. This will enable us to use 
one viewmodel in Fragments and will remove navigation logic from viewmodels by delegating these 
logics to `SharedNavigationViewModel`. Here the latest snippet shows how I prefer using navigation component:

<script src="https://gist.github.com/melomg/30fe3634cc0999df5530c71b34b3918f.js"></script>

I believe if we follow some practices and take some concerns into account, Navigation Component simplifies 
fragment stack management and visualizes our app navigation which is something I find absulately useful as
I also believe images speak louder than words.

*Thanks for reading… Please let me know your feedbacks and comments. Let’s jump to our destinations safely.*

## References

- [https://developer.android.com/guide/navigation](https://developer.android.com/guide/navigation)
- [https://proandroiddev.com/why-i-will-not-use-architecture-navigation-component-97d2ad596b36](https://proandroiddev.com/why-i-will-not-use-architecture-navigation-component-97d2ad596b36)
- [https://betterprogramming.pub/everything-about-android-jetpacks-navigation-component-b550017c7354](https://betterprogramming.pub/everything-about-android-jetpacks-navigation-component-b550017c7354)