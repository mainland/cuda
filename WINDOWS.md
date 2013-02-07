I have tested the cuda packages under Windows 8 x64 with GHC 7.4.2 (generating
32-bit code). Please use the MinGW shell to build the package, not cygwin.

Because autoconf does not deal well with paths containing spaces, when you
install the CUDA SDK, you must install it at a location with no spaces in its
path.

If you plan on invoking nvcc from the MingW shell or programmatically from a
Haskell program, you must make sure that cl.exe is in your path or nvcc will
complain. I did this by adding the following to my PATH environment variable:

C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\bin

Using cuda from ghci
====================

On windows, programs are linked at compile time against an import library
foo.lib. At runtime, this causes the program to load a DLL bar.dll. Knowing foo
is not enough to know bar; instead, foo.lib contains the information needed to
determine bar. Often foo and bar are the same, and ghci tries to guess bar from
foo, but this doesn't work with the CUDA SDK libraries. Therefore, to use the
cuda package from within ghci, you will need to manually modify the package
configuration registered with ghc.

If you installed the cuda package as a user package, look in
$APPDATA\Roaming\cabal for cuda-VER-HASH.conf, where VER is the version of the
cuda package and HASH is a long hex hash. Otherwise you will need to poke
through the directory where GHC is installed to find the same file.

Once you have found the file, look for the line that starts with
"extra-ghci-libraries: ". You will need to add nvcuda and cudart32_50_35 to this
line. The name of the CUDA runtime library differs between SDK versions, so find
the appropriate dll for your installed SDK; it will be in the same directory as
nvcc. By ensuring that nvcc is in your path, you will also ensure that ghci can
find cudart32_50_35.dll.

After fixing the "extra-ghci-libraries: " line, execute "ghc-pkg recache --user"
if you installed cuda as a user package, and "ghc-pkg recache" otherwise. You
should then be able to load the cuda package from ghci.
