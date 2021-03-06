Overview
========

This is a collection of scripts to make downloading and building mpv, ffmpeg
and libass easier. ffmpeg and libass get special treatment, because they are
essential, and distribution packages are often too old or too broken.

If you are running Mac OSX and using homebrew we provide homebrew-mpv_, an up
to date formula that compiles mpv with sensible dependencies and defaults for
OSX.

Generic Instructions
====================

(For Debian/Ubuntu see `Instructions for Debian and Ubuntu`_)

Make sure git is installed. Also check that the dependencies listed in
the next section are installed.

Checkout the build repo:

    git clone https://github.com/mpv-player/mpv-build.git

    cd mpv-build

To get the ffmpeg, libass and mpv sources and build them, run the command:

    ./rebuild -j4

The ``-j4`` asks it to use 4 parallel processes.

Note that this command implicitly does an update followed by a full cleanup
(even if nothing changes), which is supposed to reduce possible problems with
incremental builds. You can do incremental builds by explicitly calling
``./build``. This can be faster on minor updates, but breaks sometimes, e.g.
the FFmpeg build system can sometimes be a bit glitchy.

Install mpv with:

    sudo ./install

mpv doesn't need to be installed. The binary ./mpv/build/mpv can be used as-is.
You can copy it to /usr/local/bin manually. Note that libass and ffmpeg will be
statically linked with mpv when using the provided scripts, and no ffmpeg or
libass libraries are/need to be installed. There are no required config or
data files either.

Dependencies
============

Essential dependencies (incomplete list):

- gcc or clang, yasm, git
- autoconf/autotools (for libass)
- X development headers (xlib, X extensions, vdpau, GL, Xv, ...)
- Audio output development headers (libasound, pulseaudio)
- fribidi, freetype, fontconfig development headers (for libass)
- libjpeg
- OpenSSL or GnuTLS development headers if you want to open https links
  (this is also needed to make youtube-dl interaction work)
- youtube-dl (at runtime) if you want to play Youtube videos directly
  (a builtin mpv script will call it)
- libx264/libmp3lame/libfdk-aac if you want to use encoding (you have to
  add these options explicitly to the ffmpeg options, as ffmpeg won't
  autodetect these libraries; see next section)

Note: most dependencies are optional and autodetected. If they're missing,
these features will be disabled silently. This includes some dependencies
which could be considered essential.

Enabling optional ffmpeg dependencies
=====================================

ffmpeg doesn't autodetect many dependencies. Instead, it requires you to
enable them explicitly at configuration time. (And it will simply fail
if the dependencies are not available.)

You can put additional ffmpeg configure flags into ffmpeg_options. For
example, to enable some dependencies needed for encoding:

    echo --enable-libx264    >> ffmpeg_options

    echo --enable-libmp3lame >> ffmpeg_options

    echo --enable-libfdk-aac >> ffmpeg_options

Do this in the mpv-build top-level directory (the same that contains
the build scripts and this readme file). It must be done prior running
./build or ./rebuild.

Instructions for Debian and Ubuntu
==================================

**Warning**: DO NOT USE THESE INSTRUCTIONS UNLESS YOU KNOW WHAT YOU ARE DOING.
    These scripts are fragile (probably rather Debian's fault than ours). They
    will often fail with weird errors that are hard to solve. They only exist
    for users who want to hurt themselves. Use the generic instructions, unless
    you really insist on having a Debian package as result.

Install some fundamental packages with the command:

    apt-get install git devscripts equivs

Now, to checkout the build repo, run:

    git clone https://github.com/mpv-player/mpv-build.git

    cd mpv-build

To make sure that everything is up to date:

    ./update

Install the dependencies with:

    rm -f mpv-build-deps_*_*.deb

    mk-build-deps -s sudo -i

This will generate and install a dummy package that depends on the
required packages. The rm -f command makes sure there are no previous
versions of the build-deps package lying around that may otherwise
confuse mk-build-deps.

If you don't want to use sudo, you can also try:

    mk-build-deps

    su -c 'dpkg -i mpv-build-deps_*_*.deb || apt-get install -f'

You can now build a full mpv Debian package with the following command:

    dpkg-buildpackage -uc -us -b -j4

"4" is the number of jobs running in parallel - change it to match
your number of processors. The file mpv_<version>_<architecture>.deb
will be created in the parent directory. Install it with

    sudo dpkg -i ../mpv_<version>_<architecture>.deb

where you must replace <version> with the version of mpv you just
built (as indicated in debian/changelog) and <architecture> with your
architecture. As with the mpv-build-deps package, you should ensure
you are installing the correct package and not one you previously
compiled.

To recompile a new package based on the latest sources, repeat the
above steps starting with and including ``./update``.

For further information regarding the Debian package, see
debian/README.Debian.

Local changes to the git repositories
=====================================

Making local changes to the created git repositories is generally discouraged.
Updating might remove local changes or conflict with them. Sometimes the
repositories might be wiped entirely. If you make local changes, always keep
them in a separate repository and merge them after updating.

In general, changes to the mpv-build repository itself are relatively safe,
keeping branches in sub-repositories might be ok, and making local, uncommitted
changes in sub-repositories will break.

Forcing master versions for some parts
======================================

The following command can be used to delete all local changes, and to checkout
the current master version for mpv:

    ./use-mpv-master

And run ``./rebuild`` or similar. Use this to switch back to the latest release:

    ./use-mpv-release

Likewise, you can use ``./use-ffmpeg-master`` and ``./use-ffmpeg-release`` to
switch between git master and the latest FFmpeg release.

Use on your own risk.

mpv configure options
=====================

Just like ``ffmpeg_options``, the file ``mpv_options`` in the
mpv-build top-level directory can be used to set custom mpv configure
options prior to compiling. Like with ffmpeg_option, it expects one
switch per line (e.g. ``--enable-something``).

But normally, you shouldn't need this.

Building libmpv
---------------

You can enable building libmpv by enabling the configure option:

    echo --enable-libmpv-shared > mpv_options

Note that this will make the mpv-build scripts also enable PIC for all used
libraries. For this reason, be sure to run ``./clean`` before rebuilding.

The Debian packaging scripts do not support libmpv yet.

Contact
=======

You can find us on IRC in ``#mpv`` on ``irc.freenode.net``

Report bugs to the `issues tracker`_ provided by GitHub to send us bug
reports or feature requests.

.. _issues tracker: https://github.com/mpv-player/mpv/issues
.. _homebrew-mpv: https://github.com/mpv-player/homebrew-mpv
