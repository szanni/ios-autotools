Autotools helpers for iOS
=========================

Helper tools for cross-compilation of autotools libraries for iOS (and simulator) on OS X.
All libraries are static, since iOS does not support linking of custom dynamic libraries.

You will have to install `automake`, `autoconf` and `libtool` via your favorite mechanism (compile from source, homebrew, macports, ...).


Usage: iconfigure
-----------------

To configure a library for a specific architecture run `iconfigure` in the directory containing the
*configure* script.

    iconfigure armv7|armv7s|arm64|i686|x86_64

This will set up the environment for cross-compiling for the specified architecture.
You can then proceed as usual by running `make` and `make install`. `make install`
will default to install your libraries system-wide in */opt/* so that you can use them
in multiple projects.

All this can be configured by setting the following variables:

     SDKVERSION   e.g.: 7.1
     PREFIX       e.g.: /User/Home/project/staticlib

And the typical compiler flags:

    CFLAGS CPPFLAGS CXXFLAGS LDFLAGS PKG_CONFIG_PATH

For more variables and their functionality see the usage instructions and/or read
the source code.

For example to build a library locally in your project targeting the iOS SDK version 7.1 on armv7 run:

    SDKVERSION=7.1 PREFIX=/User/Home/project/staticlib iconfigure armv7
    make
    make install


Usage: autoframework
--------------------

To build an iOS framework for all architectures run `autoframework` in the directory containing
the *configure* script.

This will create two directories *Static* and *Frameworks*. *Static* will contain one
subdirectory for each architecture, each containing all installation targets for that
architecture. *Frameworks* will contain the resulting multiarch framework.

    autoframework MyLib libmylib.a

As with `iconfigure` there are a few optional settings:

    ARCHS    e.g. armv7 armv7s
    PREFIX   e.g. /User/Home/project

For example for building your framework locally for armv7 and armv7s you could run:

    ARCHS="armv7 armv7s" PREFIX=/User/Home/project autoframework MyLib libmylib.a


External build system
---------------------

While *Xcode* does not directly support *autotools* it has support for plain *Makefiles*.
You can use this to build all your libraries within your project automatically.

Add a new *External Build System* target to your project (*File > New > Target > OS X > Other > External Build System*).
Enter a name like *StaticLibs* and create a *Makefile* in the top level of your project:

    ARCHS="armv7 armv7s arm64 i686 x86_64"
    FRAMEWORKS = Frameworks/MyLib.framework

    all: $(FRAMEWORKS)

    Frameworks/MyLib.framework: mylib
      cd $< && ./autogen.sh
      # $(PROJECT_DIR)  set by Xcode
      # $(ARCHS)        set by Xcode, but sadly does not include all target architectures
      cd $< && PREFIX=$(PROJECT_DIR) ARCHS=$(ARCHS) autoframework MyLib libmylib.a

Be aware that Xcode itself does set the ARCHS variable to an incorrect value. We therefore
have to override the value by manually specifying the desired architectures in the
Makefile.

Now add the new target to the dependencies of your main project and build.
You should now be able to either add your libraries either by adding the
desired frameworks in *Frameworks/* or by adding *Static/{include,lib}* to the
paths in your main project and setting the appropriate linker flags.

Depending on where you installed your autotools suite you might have to adjust the
*PATH* variable in your new target. Go to: *Build Settings > + > Add User-Defined Setting*
and enter *PATH* with the value *$PATH:/usr/local/bin* (or wherever your tools are
located).


License
-------

ISC license, see the LICENSE file for more details.


Origins
-------

Inspired by the initial *iOS-autoconf* scripts of intlres1 and sbooth.

