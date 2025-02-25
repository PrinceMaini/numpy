project(
  'NumPy',
  'c', 'cpp', 'cython',
  version: run_command(
    # This should become `numpy/_version.py` in NumPy 2.0
    ['numpy/_build_utils/gitversion.py'],
    check: true).stdout().strip(),
  license: 'BSD-3',
  meson_version: '>=1.2.99',  # version in vendored-meson is 1.2.99
  default_options: [
    'buildtype=debugoptimized',
    'b_ndebug=if-release',
    'c_std=c99',
    'cpp_std=c++17',
    'blas=openblas',
    'lapack=openblas',
    'pkgconfig.relocatable=true',
  ],
)

fs = import('fs')

cc = meson.get_compiler('c')
cpp = meson.get_compiler('cpp')
cy = meson.get_compiler('cython')

# Check compiler is recent enough (see the SciPy Toolchain Roadmap for details)
if cc.get_id() == 'gcc'
  if not cc.version().version_compare('>=8.4')
    error('NumPy requires GCC >= 8.4')
  endif
elif cc.get_id() == 'msvc'
  if not cc.version().version_compare('>=19.20')
    error('NumPy requires at least vc142 (default with Visual Studio 2019) ' + \
          'when building with MSVC')
  endif
endif
if not cy.version().version_compare('>=0.29.34')
  error('NumPy requires Cython >= 0.29.34')
endif

py = import('python').find_installation(pure: false)
py_dep = py.dependency()

if not cc.has_header('Python.h', dependencies: py_dep)
  error('Cannot compile `Python.h`. Perhaps you need to install python-dev|python-devel')
endif

# Add default compile flags for any compiler that supports them.
# Note that MSVC does not support strict aliasing at all, and neither do the
# Intel compilers on Windows, so the `-fno` flavor of the flag should be fine.
add_project_arguments(
  cc.get_supported_arguments( '-fno-strict-aliasing'), language : 'c'
)
#
# Clang defaults to a non-strict floating error point model, but we need strict
# behavior. `-ftrapping-math` is equivalent to `-ffp-exception-behavior=strict`.
# Note that this is only supported on macOS arm64 as of XCode 14.3
if cc.get_id() == 'clang'
  add_project_arguments(
    cc.get_supported_arguments('-ftrapping-math'), language: ['c', 'cpp'],
  )
endif

subdir('meson_cpu')
subdir('numpy')
