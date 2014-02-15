# How to create a framework for iOS


In the last tutorial I wrote you learnt how to create a reusable knob control,
but it wasn't very clear exactly how you should reuse it. The simplest technique
is to provide the source code files for developers to drop into their Xcode
projects, but 

WRITE THIS LATER


## Getting Started

The main purpose of this tutorial is to explain how to create a framework which
can be used in iOS, and therefore there will only be a small amount of objective-C
code described in the tutorial. In order to get started you should download the
source files for the `RWKnobControl` available in the zip file here. You'll see
how to use them as you go through the process of creating the first project in
the section __Creating a Static Library Project__.

## What is a Framework?

A framework collects a static library and the header files associated with it
together in a well-defined structure in such a way that Xcode understands how
to find the constituent parts. On OSX it is possible to create a dynamically
linked framework - i.e. one which is shared between multiple applications and is
linked at runtime. On iOS dynamically linked frameworks are not allowed - other
than the system ones provided by Apple. There are a few reasons behind this -
principally that apps on iOS are not allowed to share code - i.e. exactly the
purpose of dynamically linked frameworks. However, this doesn't mean that 
frameworks are completely off the table - statically linked frameworks can be
of great use - collecting the static library with the the header files required.

Since this is actually what a framework is comprised of, you're first going to
learn how to create and use a static library, so that when the tutorial moves on
to building a framework then it doesn't come across as smoke and mirrors magic.


## Creating a Static Library project

Open Xcode and create a new static library project by clicking __File > New > Project__
and selecting __iOS > Framework & Library > Cocoa Touch Static Library__.

![Creating a static library](img/creating_static_lib.png)

Name the product `RWUIControls` and save the project in an empty directory.

![Naming the static library](img/options_for_static_lib.png)

A static library project is made up of header files and `.m` files, which are
compiled to make the library itself.

To make life easier for developers using your library (and later framework) you're 
going to make it so that the only need to import one header file in order to access
all the classes you wish to be available to them. When creating the static
library, Xcode created __RWUIControls.h__ and __RWUIControls.m__ - you don't
need the implementation file, so right click on __RWUIControls.m__ and select
delete. You don't need the file, so move it to the trash when asked. Open up
__RWUIControls.h__ and replace the content with the following:

    #import <UIKit/UIKit.h>

This represents an import which the entire library needs, and as you create the
different component classes you'll add them to this file, ensuring that they
become accessible for users.

Because the project you're building today relies on __UIKit__, and the static
library project doesn't link against it by default, add this as a dependency.
Select the project in the navigator, and then choose the __RWUIControls__
target in then central pane. Click on __Build Phases__ and then expand the
__Link Binary with Libraries__ section. Click the __+__ to add a new framework
and navigate to find __UIKit__ before clicking add.

INSERT GIF HERE

A static library is of no use unless it is combined with a selection of header
files which can be used by the compiler as a manifest of what classes (and
methods on those classes) exist within the binary. Some of the classes you
create in your library will be publicly accessible, but some will be for
internal use only. Next you need to add an operation in the build which will
collect together the public header files and put them somewhere accessible.
Later, these will be copied into the framework.

Whilst still looking at the same __Build Phases__ screen in Xcode as before,
find __Editor > Add Build Phase > Add Copy Headers Build Phase__.

Note: If this option is grayed out in the menu, then try clicking in the white
area below the existing build phases to get the correct focus.

INSERT GIF HERE

You can now add the __RWUIControls.h__ file to the public section of this new
build phase by dragging it from the navigator to the public part of the panel.
This will ensure that this header file is available for users of your library.

Note: All the header files that are included in any of your public headers must
also be made public. Otherwise you'll get compiler errors whilst attempting to
use the library.

### Creating a UI Control

Now that you've got your project set up, it's time to start adding some
functionality to your library - otherwise why would anybody want to use it?
Since the point of this tutorial is to describe how to build a framework, not
how to build a UI control, we'll borrow the code from the last tutorial. In the
zip file you downloaded there is a directory __RWKnobControl__. Drag it from the
finder into the __RWUIControls__ group in Xcode.

![Drag to import RWKnobControl](img/drop_rwuiknobcontrol_from_finder.png)

Choose to __Copy items into destination group's folder__ and ensure that the
new files are being added to the __RWUIControls__ static library target.

![Import settings for RWKnobControl](img/import_settings_for_rwknobcontrol.png)

This will add the `.m` files to the compilation list and, by default, the `.h`
files to the __Project__ group. This means that they will not be shared - i.e.
private.

![Default target membership](img/default_header_membership.png)

Note: The 3 target names are somewhat confusing: _public_ is shared as expected,
but _private_ is not quite the same as _project_. _Private_ headers can be shared as
private headers - not exactly what you want. Therefore, your headers should
either be in the _public_ group (shared) or the _project_ group (not-shared).

You want to share the main control header - __RWKnobControl.h__, and there are
several ways you can do this. The first is to simply drag the file from the
_project_ group to the _public_ group in the __Copy Headers__ panel.

INSERT_GIF_HERE

Alternatively, you might find it easier to change the membership in the
__Target Membership__ panel when editing the file. This is likely to be more
convenient as you continue to add and develop the library.

![Selecting target membership](header_membership.png)

Note: as you continue to add new classes to your library, don't forget to keep
the membership up-to-date. Make as few headers public as possible, and ensure
that the remainder are in the _project_ group.

### Configuring Build Settings

You are now very close to being able to build this project and create your
first ever static library, however, there are a few settings that you need to
configure in order to make the library as user-friendly as possible.

Firstly you need to provide a directory name for the public headers to be
copied to. This will ensure that you can locate the relevant headers when you
come to use the static library.

Click on the project in the Project Navigator, and then select the
__RWUIControls__ static library target. Select the __Build Settings__ tab and
then search for __"public headers"__. Double click on the __Public Headers
Folder Path__ and enter the following:

    $(PROJECT_NAME)Headers

![Set the public header location](img/public_headers_path.png)

You'll see this directory later on.

The other settings you need to change are related to what gets left in the
binary library. The compiler gives you the option of removing code which is
never accessed (dead code) and also to remove debug symbols (i.e. function
names and other details used in debugging). Since you're creating a framework
for others to use you can disable both of these kinds of stripping, and let the
user choose when they build their dependent app. To do this, use the same
search field again, this time to update the following settings:

- "Dead Code Stripping". Set this to `NO`
- "Strip Debug Symbols During Copy". Set this to `NO` for all configs
- "Strip Style". Set this to `Non-Global Symbols`

It has been a while coming, but you can now build the project. Unfortunately
there isn't a lot to see yet, but at least you can confirm it builds.

To build, select the target as __iOS Device__ and press __⌘ + B__ to perform
the build. Once this has completed then the __libRWUIControls.a__ product in the
__Products__ group of the Project Navigator will turn from red to black,
signaling that it now exists. Right click on __libRWUIControls.a__ and select
__Show in Finder__.

![Successful first build](img/successful_first_build.png)

In this directory you can see the static library itself (__libRWUIControls.a__)
and the directory you specified for the public header -
__RWUIControlsHeaders__. Notice that the headers you made public exist in the
folder as you would expect them to.

### Creating a dependent development project


## Building a Framework


## How to use a framework


## Using a bundle for resources


### Importing the bundle into the dependent project



## Where To Go From Here?
