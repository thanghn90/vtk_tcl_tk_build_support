# VTK 8 DOES NOT support tcl/tk directly
This note is for vtk 8 build. The instruction should be similar to the one in the main readme.md file, but I don't recommend building, since vtk 8 does not generate tcl library modules. I guess vtk only support tcl/tk through python from version 8 onward.
This only serves in case you want to build vtk 8 module for tkinter within python. I did not test run it nor linking python directory during vtk 8 cmake build, so you may encounter additional roadblocks.


20211208: Same as VTK 7 note, except:
- Need to modify tiff.h under C:\msys64\mingw64\vtk\VTK-8.2.0\ThirdParty\tiff\vtktiff\libtiff to remove macro condition vtktiff_EXPORTS and to add "extern" to dllexport.
Kind of similar to libxml2 AND the vtkexportheader.cmake.in:

<code>#ifdef _WIN32</code>

<code>#if TIFF_SHARED_LIBS</code>

<code>#define vtktiff_EXPORT __declspec(dllexport) extern</code>

<code>#else</code>

<code>#define vtktiff_EXPORT</code>

<code>#endif</code>

<code>#else</code>

<code>#define vtktiff_EXPORT extern</code>

<code>#endif</code>

Not sure for linux, whether I have to declare __attribute__((visibility("default"))) before the last "extern". It's up to Linux user to figure out.


- Modify exodusII.h under C:\msys64\mingw64\vtk\VTK-8.2.0\ThirdParty\exodusII\vtkexodusII\include, section that defines EXODUS_EXPORT:
From:

<code>#ifndef EXODUS_EXPORT</code>

<code>#define EXODUS_EXPORT extern</code>

<code>#endif /* EXODUS_EXPORT */</code>

To:

<code>#ifdef _WIN32</code>

<code>#ifndef EXODUS_EXPORT</code>

<code>#define EXODUS_EXPORT __declspec(dllexport) extern</code>

<code>#endif /* EXODUS_EXPORT for windows */</code>

<code>#else</code>

<code>#ifndef EXODUS_EXPORT</code>

<code>#define EXODUS_EXPORT extern</code>

<code>#endif /* EXODUS_EXPORT for unix */</code>

<code>#endif</code>

Definitely the people who wrote this exodus code doesn't think about Windows.
Also, on the same error, there is ONE constant that is defined in TWO places but not declared as exported
In ex_create_par.c and ex_open_par.c, under C:\msys64\mingw64\vtk\VTK-8.2.0\ThirdParty\exodusII\vtkexodusII\src:
Change:

<code>const char exodus_unused_symbol_dummy_1;</code>

To:

<code>EXODUS_EXPORT const char exodus_unused_symbol_dummy_1;</code>

Ironically, the one that's not used is the one that cause so much trouble...


- No need to make change to vtkLabelHierarchyPrivate.h and vtkLabelHierarchy.cxx under C:\msys64\mingw64\vtk\VTK-7.1.1\Rendering\Label\. The "const" was added to the bool operator ().
Good to know someone already fixed this.


- No need to make change to ThirdParty tiff cmake list file. Again, someone already fixed this.


- What the... There is no wrapping TCL??? How am I suppose to use VTK with tcl/tk?
I guess VTK only support TK THROUGH python. Time to compile vtk for python then.
