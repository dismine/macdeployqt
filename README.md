Fixed macdeployqt tool for Qt 6.6
=================================

Ths repository contains a fork of the macdeployqt utility, which can be used to copy required Qt libraries and plug-ins
into applications bundles and then codesign the whole bundle for notarization.

The codesign feature is a recent addition to the tool, and doesn't work properly since it only signs binaries like
shared libraries, framework libraries and executables, but not the enclosing bundles (e.g. the `.app` or `.framework`
directories). This will lead to codesign complaining about the Qt frameworks being "not signed at all" and refusing to
sign the app bundle executable(s).

This repository contains a brute-force fix for this issue. For every binary signed, the tool now traverses the path of
that binary and looks for any directory names ending with `.framework`. If one is found, it is also signed. For
frameworks containing multiple binaries, this can result in some codesign invocations to fail, but the last try should
succeed (since all binaries inside are signed then).

In addition to signing frameworks, the enclosing application bundle is now also signed. Note this is done with the same
settings as all files inside the bundle, so if you need additional parameters, e.g. for specific entitlements, the
bundle needs to be properly signed again after running macdeployqt.

## Compatibility with brew

Original macdeployqt utility is not compatible with brew. Brew packages Qt library differently. Install name path for
the library absolute instead of relative with `@rpath` which macdeployqt expects. To avoid this issue we propose to add
a symbolic link to place where macdeployqt expects Qt library from brew.

```shell
ln -s $(brew --prefix qt)/lib ./lib
```

While solution with symbolic link helps to find all dependecies it doesn't fix codesigning issue. For some reasons
macdeployqt doesn't follow `@loader_path` to find binary dependecies. Which is critical because by default it adds
`@loader_path` to all install names.

## Building macdeployqt

The repository is organized to build a standalone executable, which, after installation, is set tup with an RPATH in a
way that it can replace the original tool in a Qt build.

### Requirements

The code was taken from the Qt 6.6 version, so it is recommended to build it against this version. It should run
with future versions as well though. Other than that, the build requirements are basically the same as for building Qt
6.6 on macOS.

It is recommended to use the Ninja build generator, as this is Qt's preferred build tool. It should build with the Unix
Makefiles or Xcode generators as well.

### Running the build

Configure the build using cmake, then build and install the executable:

```shell
cmake -GNinja \
    -S /path/to/macdeployqt-sources \
    -B /path/to/build-dir
    -DCMAKE_INSTALL_PREFIX=/path/to/install-dir
    -DCMAKE_BUILD_TYPE=Release
cmake --build /path/to/build-dir --target install
```

If you want to sign with certificate pass `-DCODE_SIGN_IDENTITY="Developer ID Application: YOUR NAME \(TEAM_ID\)"`.

After running the above commands, you should have a `macdeployqt` executable in the install dir. Copy it to your Qt
installation's `bin` dir and replace the original executable located there.

### Providing a different RPATH

If you don't want to replace the original executable or use the program from a different location, set
the `MACDEPLOQT_RPATH` variable to the required value, e.g. via ` -DMACDEPLOQT_RPATH=/some/other/path` in the initial
cmake configure command.

By default, it is set to `@executable_path/../lib`.
