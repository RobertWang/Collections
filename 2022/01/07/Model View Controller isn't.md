> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [beza1e1.tuxen.de](http://beza1e1.tuxen.de/model_view_controller.html)

> Maybe the most misunderstood design pattern.

The [Model-View-Controller (MVC) pattern](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) is everywhere. Web frameworks use. Apps use it. GUIs use it. Well, actually they don't use MVC. It is just a marketing lie. They use something else like MVA.

The History of MVC
------------------

MVC was invented in the 70s, when Alan Kay with his group worked on the Dynabook and Smalltalk at Xerox Parc. [Trygve M. H. Reenskaug remembers it](http://heim.ifi.uio.no/~trygver/themes/mvc/mvc-index.html):

> I created the Model-View-Controller pattern as an obvious solution to the general problem of giving users control over their information as seen from multiple perspectives. MVC has created a surprising amount of interest. Some texts even use perverted variants for the opposite purpose of making the computer control the user.

Let us try to understand what they had mind, when they use the terms model, view, and controller. An example from [the original paper](http://heim.ifi.uio.no/~trygver/1979/mvc-1/1979-05-MVC.pdf) is project management, where we have activities with order relation between them. The model is identical to todays meaning. In today's UML we would design the model like this:

Now think about ways to represent a model like that. Here are three from the paper:

![](http://beza1e1.tuxen.de/img/three_views.png)

Reenskaug considered views to be subclasses of more general UI elements like today. For example, the first view is an "ActivityList, which is a subclass of ListView."

There is also a definition for views:

> To any given Model there is attached one or more Views, each View being capable of showing one or more pictorial representations of the Model on the screen and on hardcopy. A View is also able to perform such operations upon the Model that is reasonabely associated with that View.

This definition is different to the ones in the current literature. It says that the view can modify the model without any controller.

This first paper also features an "editor", which composes multiple views on the screen. There is no "controller" yet. That appears in a [second paper](http://heim.ifi.uio.no/~trygver/1979/mvc-2/1979-12-MVC.pdf):

> A controller is the link between a user and the system. It provides the user with input by arranging for relevant views to present themselves in appropriate places on the screen. It provides means for user output by presenting the user with menus or other means of giving commands and data. The controller receives such user output, translates it into the appropriate messages and pass these messages on to one or more of the views.
> 
> A controller should never supplement the views, it should for example never connect the views of nodes by drawing arrows between them.
> 
> Conversely, a view should never know about user input, such as mouse operations and keystrokes. It should always be possible to write a method in a controller that sends messages to views which exactly reproduce any sequence of user commands.

The term "editor" also appears but with a different meaning than in the first paper. Here, it acts as an adapter which you can temporarily put between view and controller for an edit mode.

Jim Althoff then implemented MVC in the Smalltalk-80 class library but again slightly different. The paper [A cookbook for using the model-view controller user interface paradigm in Smalltalk-80](http://dl.acm.org/citation.cfm?id=50757.50759) documents that in 1988. What we call "observers" today, was "dependents" in that publication.

![](http://beza1e1.tuxen.de/img/mvc_smalltalk.png)

MVC was not part of the popular [Design Patterns](https://en.wikipedia.org/wiki/Design_Patterns) book by the Gang of Four in 1994. I consider the Smalltalk implementation as the MVC reference because it was the first instance which multiple projects and developers used. In todays UML, it would be this:

In 2003, Reenskaug revisits his design and splits it into multiple patterns. He calls that [MVC Pattern Language](http://heim.ifi.uio.no/~trygver/2003/javazone-jaoo/MVC_pattern.pdf).

*   The "Model/Editor Separation" describes the decoupling of the model from the interface to interact with it. This enables the model to be close to the mental model of users, which is considered good object oriented design. The interface needs not be considered in the model.
*   The "Input/Output Separation" is about the split of handling user input via a controller and presenting the model to the user via a view. This basic idea is credited to the Smalltalk-80 implementation.
*   The "Tools for Tasks" pattern is "the original MVC". The problem it solves is "to give the user a Tool for performing one or more tasks. The Tool shall give the user an illusion of interacting directly with the model."

The paper also describes the controller and view parts as a [facade](https://en.wikipedia.org/wiki/Facade_pattern) for the model according to the Gang of Four terminology.

A future vision Reenskaug describes in the paper is to generate the editor from the model. So he might find [Naked Objects](https://en.wikipedia.org/wiki/Naked_objects) a better MVC according to his original overall intent.

MVC from Principles
-------------------

We can also derive MVC from architectural principles and thanks to design patterns we can describe that tersely in two steps, which maps directly to the "Model/Editor Separation" and the "Input/Output Separation" of Reenskaug.

1.  In a layered architecture, we want to decouple UI from the lower-level model. Lower levels must not know of higher levels. They use abstractions for looser coupling. Thus, we must use observers to call from the model to the UI.
2.  We also want to decouple the user interaction from the graphical representation to increase cohesion, thus we split into controller and view on the UI layer. Since they both must directly reference the model, they stay on the same level and are free to interact without observer-decoupling.

This derivation is why I draw the model lower than controller and view. The depiction from Smalltalk-80 seems to have the same idea.

The second step is not well motivated in todays environments. Often there is no need to actually split controller and view. John Gossmann [remarked in 2005](https://blogs.msdn.microsoft.com/johngossman/2005/10/08/introduction-to-modelviewviewmodel-pattern-for-building-wpf-apps/):

> what exactly happened to Controller in modern GUI development is a long digression...I tend to think it just faded into the background. It is still there, but we don't have to think about it as much as we did in 1979

However, as a framework you want to claim the MVC term because it gives reviewers a nice feeling of familiarity. Unfortunately, sometimes a reviewer is not fooled. In that case, just pick two components in your UI layer call one view and the other whatever. Now you can claim that you made a variation.

Variations of MVC
-----------------

[Martin Fowler](https://www.martinfowler.com/eaaDev/uiArchs.html) and [Derek Greer](http://aspiringcraftsman.com/2007/08/25/interactive-application-architecture/) both made overviews. The differences are sometimes subtle.

For example, there is **Model-View-Presenter** (MVP). The class diagram Greer draws looks exactly like his MVC class diagram except that he changed "Controller" to "Presenter". This only hints at the actual difference: A controller has the primary job to handle inputs and can modify the model accordingly. A presenter has the primary job to modify the model while the view also handles the inputs.

**Model-View-ViewModel** (MVVM) also merges the input handling into the view and it decouples model and view completely. In MVC the view is directly connected to the model, while the reverse direction is abstract via observers. MVVM disconnects the view from the model and introduces the ViewModel as indirection. What is commonly in the ViewModel for example, is selection logic which usually affects multiple GUI elements at once. The primary motivation for MVVM was to enable the view to be defined by XML without any logic, so UI designers without programming skills can build that. The context is not Smalltalk, but Microsofts WPF apps. John Gossmann [introduced the MVVM pattern](https://blogs.msdn.microsoft.com/johngossman/2005/10/08/introduction-to-modelviewviewmodel-pattern-for-building-wpf-apps/). I'm not sure about the class diagram of MVVM. What I found is [descriptions like this one from Josh Smith](https://msdn.microsoft.com/en-us/magazine/dd419663.aspx):

> The view classes have no idea that the model classes exist, while the ViewModel and model are unaware of the view.

MVVM also is an instance of Fowlers [Passive View](https://www.martinfowler.com/eaaDev/PassiveScreen.html) idea which is motivated by improved testability: GUIs cannot be unittested well, so make the GUI part small and put as much code as possible elsewhere.

**Model-View-Adapter** (MVA) also decouples model and view completely, but without the intention to not have logic in the views. The input also comes through the view, so this turns the controller into a mere adapter. The adapter is valuable for example, when it decouples data base schema from the GUI.

Since the class diagrams of MVA and MVVM are isomorphic, I consider MVVM a WPF-specific MVA.

Conclusions
-----------

MVC is not a well-defined concept, so do not assume it to be one in conversations. Treat it more like a buzzword like "cloud". MVC might be something precise within specific contexts (e.g. Android app development), but is only a fuzzy concept in general.

If you design a system, it is not helpful to consider options like MVC vs MVVM vs MVP. Instead, identify potential problems and if necessary solve them. For example, coupling between UI and model is potentially a problem because it makes changes to either of them more costly as you have to adapt the other part as well. You can proactively decouple them by using observers instead of direct calls. You can also use something other than observers or not decouple it at all.

![](http://beza1e1.tuxen.de/img/mvc_again.jpg)

* * *

I made the UML diagrams with [draw.io](https://www.draw.io/) which is nice for such little drawings.

Maybe the most misunderstood design pattern.