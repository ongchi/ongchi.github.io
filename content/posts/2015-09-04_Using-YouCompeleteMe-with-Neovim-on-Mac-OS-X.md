+++
title = "Using YouCompleteMe with Neovim (or vim) on Mac OS X"
date = 2015-09-04T12:18:00Z

[taxonomies]
tags = ["neovim", "vim"]
+++

## Working with Custom Python Installation

The [install guide][ycm-readme] does not always work if there are various python installation on your system.
If your YCM link against different python version from vim, may cause python crashes on start up:

```sh
Fatal Python error: PyThreadState_Get: no current thread
```

It is better to specify python paths for build with YCM plugin.

For example, I have miniconda distribution of python2.7 located in /opt/miniconda,
run the following command to generate makefiles and build for YCM:

```sh
~ $ mkdir ycmd_build
~ $ cd ycmd_build
~/ycmd_build $ cmake -G "Unix Makefiles" . \
~/.dotfiles/vim/plugged/YouCompleteMe/third_party/ycmd/cpp \
-DPYTHON_INCLUDE_DIR=/opt/miniconda/include/python2.7 \
-DPYTHON_LIBRARY=/opt/miniconda/lib/libpython2.7.dylib \
-DPYTHON_EXECUTABLE=/opt/miniconda/bin/python2.7
~/ycmd_build $ make ycm_support_libs
```

For (mac)vim, you'd better to make sure to compile with same version of python or it will crashes.

After above works, the python still crashing every time I launched nvim with no luck...😞

Look into the YCM shared libs, it is linked with relative path
even though I have specified the absolute path to cmake:

```sh
~/.d/v/p/Y/t/ycmd $ otool -L ycm_*.so
ycm_core.so:
        @rpath/ycm_core.so (compatibility version 0.0.0, current version 0.0.0)
        libpython2.7.dylib (compatibility version 2.7.0, current version 2.7.0)
        /usr/lib/libc++.1.dylib (compatibility version 1.0.0, current version 120.1.0)
        /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1225.0.0)
ycm_client_support.so:
        @rpath/ycm_client_support.so (compatibility version 0.0.0, current version 0.0.0)
        libpython2.7.dylib (compatibility version 2.7.0, current version 2.7.0)
        /usr/lib/libc++.1.dylib (compatibility version 1.0.0, current version 120.1.0)
        /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1225.0.0)
```

Finally, I realize that is the reason why pyton crashes at nvim start up.

So I just change shared library point to correct path manually by install\_name\_tool
(both ycm\_core.so and ycm\_client\_support.so).

```sh
~/.vim/plug/ycmd $ install_name_tool -change \
"libpython2.7.dylib" \
"/opt/miniconda/lib/libpython2.7.dylib" \
ycm_core.so
```

Now everything works like a charm.

## References
* <https://github.com/Valloric/YouCompleteMe/blob/master/README.md>
* <https://github.com/Valloric/YouCompleteMe/issues/18>

[ycm-readme]: https://github.com/Valloric/YouCompleteMe/blob/master/README.md
