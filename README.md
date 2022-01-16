# Preface
While most people would stick with python and Qt to build modern user interfaces (GUIs), I recently found an ancient language, tcl/tk, existed well before the birth of python, and yet still used today in the industry, quietly. It's royalty-free, available in all python distributions, as well as Linux and mingw64. Despite its availability, it's not popular, partly because it's superceded by python, which is much more beginner-friendly. Though, with its speed, small foot-print (less than 10Mb for the barebone core), I decide to turn into tcl/tk for building user interfaces and interactive graphical programs for my own scientific projects. I already knew there exist vtk for scientific plotting and 3D displays, but it dropped support for tcl/tk since version 8, again due to the unpopularity of tcl/tk language. Luckily, vtk still host their 7.1 source code, and that version appear to be the last to support tcl/tk without python. so, I set out to build vtk 7.1 to call its functions directly in tcl, and of course, encountered many roadblocks. Lucky for me, I was able to punch through all the roadblocks, one by one, until I was able to do it. This git repository is intended to host the fixed codes of vtk-7.1 and instruction of how to build vtk-7.1 from scratch with tcl/tk support, in DYNAMIC library form, using Windows's msys2-mingw64 and Linux's gcc.


# Instructions
Enough of my saga, here is the instruction of how to build vtk 7.1 from source with tcl/tk support, in dynamic library form:
1. Download and extract the vtk 7.1.1 source code from vtk official website:
https://www.vtk.org/files/release/7.1/VTK-7.1.1.zip
2. Download and extract all the files and folders in this git repository (except this readme, mind you).
3. Copy and replace all the extracted files and folders from this git repository into the extracted vtk 7.1.1 source directory.
4. In Windows, install msys2 from here, and follow all the steps to install core packages for coding (e.g. base-devel, mingw-w64-x86_64-toolchain)
https://www.msys2.org/
5. Download and install cmake (actually, cmake-gui interface).
https://cmake.org/download/
Special notes for Linux build-from-source: need to install extra package to fix cmake gui configure error:
- mesa-libGL-devel.x86_64
- libXt-devel.x86_64
(need mesa-dri-drivers.x86_64 for 3D/OpenGL display across X-forwarding anyways)
6. Start Cmake.
Special notes for Windows: When using cmake the first time, make sure to configure for MSYS (but still run "make" under mingw64 afterward. There are several msys window, one is msys2-msys, and one is msys2-mingw64. Make sure to use msys2-mingw64 to run "make" command)
7. Set CMAKE_BUILD_TYPE to Release (of course...)
8. Make a new directory for installation instead of the default C:/Program Files (x86)/VTK under "CMAKE_INSTALL_PREFIX"
9. To compile for TCLTK, tick "VTK_Group_Tk" and "VTK_WRAP_TCL". Skip python. Then hit Configure.
10. Make sure to build tcl/tk from source: https://www.tcl.tk/software/tcltk/download.html, with
make install-private-headers
after make;make install
Alternative: download and install pre-built tcl/tk with tcl3d and a bunch of good extras from bawt website and point the TCL/TK include path there (in cmakegui).
http://www.bawt.tcl3d.org/preview.html
11. Next, specify TCL and TK include/library path (should be in the <tcl/tk_install_directyr>/include and /lib). VTK_USE_TK is automatically set to ON. Tick "Advanced" to show and set the definition of TK and TCL include/lib directory.
Disclaimer: Do NOT use the one bundled with mingw64 (stupid mingw64 tcl/tk include separation into "generic" and "win", and then some outside - we need them in THE SAME PLACE!!!). Else the vtk application will crash/quit for no reason, especially right after a vtkTkRenderWidget initialization - which is the WHOLE POINT of vtk integration to tcl/tk.
12. Change TK_INTERNAL_PATH to be the same with TK_INCLUDE_PATH (after make install-private-headers)
13. Finally, click "Generate"
14. Cd to the build directory:

<code>cd <build_directory></code>

For Windows, make sure to run msys2-mingw64 instead of msys2-msys console.
To speed up, type:

<code>make -j8</code>
	
for 8 thread parallel build.
Type:
	
<code>make <specific_vtk_module></code>
	
e.g.
	
<code>make vtkRenderingLabel</code>
	
To make a specific module only (in case something went wrong and you don't want the output to be flooded with other module's building logs.
15. Once it's done, type:
	
<code>make install</code>
	
The vtk modules for tcl are located in <install_dir>/lib/tcltk/vtk-7.1. The compiled dynamic libraries are in <install_dir>/bin. Simply copy this whole vtk-7.1 folder into the tcl or tk lib directory, where all other tcl/tk module directories are located, and make sure the dynamic libraries are in PATH and LD_LIBRARY_PATH. The easiest way is to copy those dynamic libraries into the same directory where tclsh.exe and/or wish.exe are located.
To use vtk in wish/tcl, type
package require vtk


# Full Saga
Below is my notes of all the pains and sufferings I encountered during vtk 7.1 build for tcl, and how to fix them.

20211208, 12:03pm:

- Need to change a macro (again) to make sure linux understand it's a dynamic library export:
File VTKGenerateExportHeader.cmake, in C:\msys64\mingw64\vtk\VTK-7.1.1\CMake: Change line:

<code>elseif(COMPILER_HAS_HIDDEN_VISIBILITY AND USE_COMPILER_HIDDEN_VISIBILITY)</code>

to:

<code>else()</code>

Most modern compiler has this flag (visibility), but somehow cmake failed to recognize this my linux system's gcc.


20211208: finalized guide for vtk rebuild using msys2 mingw64 Windows 10 1809. At first, I have to modify some file after cmake, but now I know where to modify before cmake config. Also, I was able to fix some weird compilation error for 3rd party libs that I couldn't before, and also figured out why vtk broke in tcl vtkTkRenderWidget initialization. It was tough, but totally worth it. I'm sure there are more compiler warning that I didn't catch, but I'll leave that for the audience.

- Modify C:\msys64\mingw64\vtk\VTK-7.1.1\Common\Core\vtkAOSDataArrayTemplate.h:
Add "VTKCOMMONCORE_EXPORT" to FastDownCast definition:

<code>VTKCOMMONCORE_EXPORT static vtkAOSDataArrayTemplate<ValueType>* FastDownCast(vtkAbstractArray *source){</code>


- Put these lines in C:\msys64\mingw64\vtk\VTK-7.1.1\Rendering\OpenGL2\vtkWin32RenderWindowInteractor.cxx inside a "#ifndef _WIN32" macro check, like this:

<code>#ifndef _WIN32</code>
	
<code>typedef struct _TOUCHINPUT {</code>

<code>  LONG      x;</code>

<code>  LONG      y;</code>

<code>  HANDLE    hSource;</code>

<code>  DWORD     dwID;</code>
	
<code>  DWORD     dwFlags;</code>
	
<code>  DWORD     dwMask;</code>
	
<code>  DWORD     dwTime;</code>
	
<code>  ULONG_PTR dwExtraInfo;</code>
	
<code>  DWORD     cxContact;</code>
	
<code>  DWORD     cyContact;</code>
	
<code>} TOUCHINPUT, *PTOUCHINPUT;</code>
	
<code>DECLARE_HANDLE(HTOUCHINPUT);</code>
	
<code>#define HTOUCHINPUT ULONG</code>
	
<code>#endif</code>

These are already defined in winuser.h of mingw. Possibly a Linux remnant.


- In C:\msys64\mingw64\vtk\VTK-7.1.1\Rendering\Label\vtkLabelHierarchyPrivate.h: add "const" to this line like this:
	
<code>bool operator () ( const vtkIdType& a, const vtkIdType& b ) const</code>

and in C:\msys64\mingw64\vtk\VTK-7.1.1\Rendering\Label\vtkLabelHierarchy.cxx: add "const" to this line like this:
	
<code>bool operator()(const vtkHierarchyNode & a,</code>
	
<code>                    const vtkHierarchyNode & b) const</code>
					
Both comparators' bool operator() definition must be set to const. Damn, this one is tough to trace...
Source: https://stackoverflow.com/questions/51235355/comparison-object-being-invocable-as-const


- Modify C:\msys64\mingw64\vtk\VTK-7.1.1\ThirdParty\tiff\vtktiff\CMakeLists.txt, line 493 (or find FILE_OFFSET_BITS). Change:

<code>if (UNIX)</code>
	
to:
	
<code>if (UNIX OR MINGW)</code>
	
This will remove the compile error "left hand of == is empty" for FILE_OFFSET_BITS in stdio.h


- Another tough-to-track bug is in C:\msys64\mingw64\vtk\VTK-7.1.1\ThirdParty\libxml2\vtklibxml2\include\libxml\xmlexports.h:
within the section of:
	
<code>#if defined(_WIN32) && defined(__MINGW32__)</code>
	
Change this line:
	
<code>    #define XMLPUBVAR __declspec(dllexport)</code>
	
To:
	
<code>#define XMLPUBVAR __declspec(dllexport) extern</code>
	
This will fix "multiple definition" compilation error caused by, again, mismanagement of C-C++ linkage.
Source: https://mail.gnome.org/archives/xml/2010-May/msg00026.html


- In C:\msys64\mingw64\vtk\VTK-7.1.1\Common\DataModel\vtkPolyData.h, move the whole section of:
	
<code>inline unsigned char vtkPolyData::GetCellPoints(){}</code>
	
up, above where the first time it was used.
Simply search for <code>GetCellPoints(</code> and find the first line of code where it was first used (yes, USED, not just declared)
Hint: it's in <code>inline int vtkPolyData::IsTriangle(int v1, int v2, int v3)</code> for vtk 7.1:

<code>this->GetCellPoints(cells[j], n2, tVerts2);</code>
	
So, move the definition of <code>inline unsigned char vtkPolyData::GetCellPoints(){}</code> above <code>IsTriangle</code> function
This will suppress the annoying warning of redefined without dllimport shit...


- Modify vtkexportheader.cmake.in under C:\msys64\mingw64\vtk\VTK-7.1.1\CMake:
Basically we need to remove the reference to EXMPORT_IMPORT_CONDITION and DEFINE_IMPORT. I really hate when EXPORT macros means dllimport. If it's static, just leave it, don't dllexport or dllimport it!!!
	
<code>#  ifndef @EXPORT_MACRO_NAME@</code>
	
<code>#      define @EXPORT_MACRO_NAME@ @DEFINE_EXPORT@</code>

<code>#  endif</code>
	
This will ensure everything is built as DLL.


- Need to change pkgIndex.tcl.in under C:\msys64\mingw64\vtk\VTK-7.1.1\Wrapping\Tcl\ to make it load a dll library in the PATH:
	
<code>proc load_library_package {libName libPath {libPrefix {@VTK_TCL_LIBNAME_PREFIX@}}} {</code>
	
to
	
<code>proc load_library_package {libName {libPrefix {@VTK_TCL_LIBNAME_PREFIX@}}} {</code>

Change:
	
<code>set libFile [file join $libPath "$libPrefix$libName-@VTK_MAJOR_VERSION@.@VTK_MINOR_VERSION@$libExt"]</code>
	
to
	
<code>set libFile "$libPrefix$libName-@VTK_MAJOR_VERSION@.@VTK_MINOR_VERSION@$libExt"</code>
	
Change:
	
<code>if {[catch "cd {$libPath}; load {$libFile}" errorMessage]} {</code>
	
to
	
<code>if {[catch "load {$libFile}" errorMessage]} {</code>

Change
	
<code>::vtk::init::load_library_package {vtk${kit}TCL} {@VTK_TCL_LIBRARY_DIR@}</code>
	
to
	
<code>::vtk::init::load_library_package {vtk${kit}TCL}</code>

This will skip the stupid library search path and instead use the default "load" function with simple pure library file name without path. Always work.


