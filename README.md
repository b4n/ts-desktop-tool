ts-desktop-tool
===============

`ts-desktop-tool` is a a small and hackish tool to help translate *.desktop*
files using Qt's *.ts* files.

Genesis
-------

It is the result of not finding any way of externalizing desktop file
translations in the Qt world.  The closest I could find was in the context of
KDE, but it seemed to be buried far too deep inside their tool set for it to
be usable outside of KDE.
At any rate, coming from a GetText world with `intltool`, I couldn't accept
having to manually translate the desktop file directly in it, with all
languages mixed.

So this tool started to form, basically as a poor man's `intltool` dealing
with *.ts* files.

This tool was written to only depend on a POSIX shell and basic POSIX
utilities like `sed`, `grep` & such.  As a result, although I tried to make it
as robust as it was easy enough to do, it is not proof against weird or
invalid input.  Expect the unexpected on bizarre input, with a wide definition
of bizarre.  This said, it should still work well on any common case.

Features
--------

This tool can do two things:

* Create a C++ source file parseable by `lupdate` from a desktop file

        ts-desktop-tool extract

* Merge translations from a series of *.ts* file into a *.desktop* file

        ts-desktop-tool merge TS_FILE...

All commands read the template *.desktop* file from stdin, and write their
output to stdout.
The input format for desktop files is the same as the one used by `intltool`,
that is keys to be translated should be prefixed by an underscore (`_`).

Makefile.am example
-------------------

    translations = \
    	myapp_de.ts \
    	myapp_es.ts \
    	myapp_fr.ts \
        ...

    desktopdir = $(datadir)/applications
    desktop_in_files = myapp.desktop.in
    nodist_desktop_DATA = $(desktop_in_files:.desktop.in=.desktop)

    $(desktop_in_files:.desktop.in=.desktop): $(translations:%=$(srcdir)/%)

    SUFFIXES = .desktop.in .desktop.cpp .desktop
    .desktop.in.desktop.cpp:
    	ts-desktop-tool extract <$< >$@
    .desktop.in.desktop:
    	ts-desktop-tool merge $(translations:%=$(srcdir)/%) <$< >$@
    CLEANFILES = \
    	$(desktop_in_files:.desktop.in=.desktop.cpp) \
    	$(desktop_in_files:.desktop.in=.desktop)
    EXTRA_DIST = \
    	$(desktop_in_files)

    # Assuming the update-ts rule runs lupdate on all its prerequisites
    update-ts: $(desktop_in_files:.desktop.in=.desktop.cpp)

Limitations
-----------

This tool is fairly hackish in its implementation, and doesn't use robust
parsers -- far from it.  It should work in all usual cases, but can likely
get fooled by valid yet uncommon input, and will surely be fooled by invalid
input.

License
-------

    Copyright 2018 Colomban Wendling <ban@herbesfolles.org>

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
