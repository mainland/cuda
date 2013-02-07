AC_INIT([Haskell CUDA bindings], [0.5.0.0], [tmcdonell@cse.unsw.edu.au], [cuda])
AC_CONFIG_SRCDIR([Foreign/CUDA.hs])
AC_CONFIG_FILES([cuda.buildinfo])
AC_PROG_CC

# Determine the target platform
#
AC_CANONICAL_TARGET

# Search the user's PATH for the 'nvcc' compiler. If it is found, add this
# prefix to the include and library search directories. Additionally, set nvcc
# as the C preprocessor for c2hs (only, or it won't try to link cudart)
#
AC_ARG_WITH([compiler], [Haskell compiler], [GHC=$withval],  [AC_PATH_PROG(GHC, ghc)])
AC_ARG_WITH([nvcc],     [CUDA compiler],    [NVCC=$withval], [AC_PATH_PROG(NVCC, nvcc)])
AC_ARG_WITH([gcc],      [C compiler],       [CC=$withval])

if test "$NVCC" != ""; then
    case $target in
	*mingw* )
	    # We need a Windows-style path, but mingw doesn't include cygpath,
	    # thus this hack.
	    NVCC=`cd $(dirname $NVCC) && pwd -W`/$(basename nvcc)
	    ;;
    esac

    cuda_prefix="$(dirname "$(dirname "$NVCC")")"

    case $target in
	*mingw* )
	    cuda_c2hsflags="--cppopts=-E --cppopts="-D${target_os}_TARGET_OS=1" "
	    ;;
	* )
	    cuda_c2hsflags="--cpp="$NVCC" --cppopts=-E "
	    ;;
    esac

    CPPFLAGS+=" "-I${cuda_prefix}/include" "-D${target_os}_TARGET_OS=1" "
#    CC=${NVCC}

    case $target in
    *mingw32* ) cuda_libdir="${cuda_prefix}/lib/Win32" ;;
    *mingw64* ) cuda_libdir="${cuda_prefix}/lib/x64" ;;
    x86_64*)    cuda_libdir="${cuda_prefix}/lib64" ;;
    *)          cuda_libdir="${cuda_prefix}/lib"   ;;
    esac

    LDFLAGS+=" -L${cuda_libdir} "
fi

# Recent versions of Mac OS X (10.6 and later) provides a C extension for
# creating lambda-like closure expressions (blocks), the syntax for which
# confuses the c2hs preprocessor. We disable this by undefining the __BLOCKS__
# macro.
#
AC_MSG_CHECKING(for Apple Blocks extension)
if test -r "/usr/include/stdlib.h"; then
    BLOCKS=`grep __BLOCKS__ /usr/include/stdlib.h`
fi
if test "$BLOCKS" != ""; then
    cuda_c2hsflags+="--cppopts=-U__BLOCKS__ "
    AC_MSG_RESULT(yes)
else
    AC_MSG_RESULT(no)
fi

# If we are running on Mac OS add the CUDA library path to the search list. This
# option allows applications to run without requiring to set [DY]LD_LIBRARY_PATH
#
case $build in
    *darwin* ) LDFLAGS+=" -Xlinker -rpath ${cuda_prefix}/lib " ;;
    * ) ;;
esac

# Make sure both the driver and runtime are found
#
longerror='
********************************************************************************

The configuration process failed to locate your CUDA installation. Ensure that
you have installed both the developer driver and toolkit, available from:

  http://developer.nvidia.com/cuda-downloads

and make sure that "nvcc" is available in your PATH. Check the above output log
and run the command directly to ensure it can be located.

If you have a non-standard installation, you can add additional search paths
using --extra-include-dirs and --extra-lib-dirs. Note that 64-bit Linux flavours
often require both "lib64" and "lib" library paths, in that order.

********************************************************************************'

AC_CHECK_HEADERS([cuda.h cuda_runtime_api.h], [], [AC_MSG_ERROR(could not find CUDA headers${longerror})])

case $target in
    *mingw32* )
    	# AC_SEARCH_LIBS doesn't work on Win32 with functions that use the
    	# stdcall calling convention, so we use AC_CHECK_DELCS instead.
	LIBS+="-lcuda -lcudart"
	AC_CHECK_DECLS([cuDriverGetVersion],
	    [],
	    [AC_MSG_ERROR(could not find CUDA driver library${longerror})],
	    [[#include <cuda.h>]])
	AC_CHECK_DECLS([cudaRuntimeGetVersion],
	    [],
	    [AC_MSG_ERROR(could not find CUDA runtime library${longerror})],
	    [[#include <cuda_runtime_api.h>]])

	cuda_ghci_libs+="`nm ${cuda_libdir}/cuda.lib | head -2 | tail -1 | sed -e 's/://' -e 's/.dll//'` "
	cuda_ghci_libs+="`nm ${cuda_libdir}/cudart.lib | head -2 | tail -1 | sed -e 's/://' -e 's/.dll//'` "
	;;
    *)
    	AC_SEARCH_LIBS(cuDriverGetVersion,    cuda,   [], [AC_MSG_ERROR(could not find CUDA driver library${longerror})])
	AC_SEARCH_LIBS(cudaRuntimeGetVersion, cudart, [], [AC_MSG_ERROR(could not find CUDA runtime library${longerror})])
	;;	
esac

# Populate the buildinfo, with the search paths and any target specific options
#
cuda_cppflags="$CPPFLAGS "
cuda_ldflags="$LDFLAGS "

AC_SUBST([cuda_cppflags])
AC_SUBST([cuda_ldflags])
AC_SUBST([cuda_c2hsflags])
AC_SUBST([cuda_ghci_libs])
AC_OUTPUT

