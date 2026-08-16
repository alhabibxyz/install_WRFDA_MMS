# WRFDA Installation with Intel Compiler

This tutorial describes the installation of **WRFDA (WRF Data Assimilation)** using the **Intel Compiler** and **Intel MPI** environment on the MMS system.

## 1. Login to the MMS Login Node

First, log in to the MMS login node.

```text
[ens_t4@mdclogin1 ~]$
```

## 2. Check the Available Compilers

Verify that the GNU C compiler is available:

```bash
which gcc
```

Expected output:

```text
/usr/bin/gcc
```

Check the Intel C Compiler:

```bash
which icc
```

Expected output:

```text
/opt/software/intel/oneapi/compiler/2022.0.2/linux/bin/intel64/icc
```

Check the Intel Fortran Compiler:

```bash
which ifort
```

Expected output:

```text
/opt/software/intel/oneapi/compiler/2022.0.2/linux/bin/intel64/ifort
```

## 3. Check Intel MPI

Check the Intel MPI C compiler wrapper:

```bash
which mpicc
```

Expected output:

```text
/opt/software/intel/oneapi/mpi/2021.5.1/bin/mpicc
```

Check the Intel MPI Fortran compiler wrapper:

```bash
which mpif90
```

Expected output:

```text
/opt/software/intel/oneapi/mpi/2021.5.1/bin/mpif90
```

The compiler environment used in this installation is therefore:

| Component              | Version  | Path                                                                   |
| ---------------------- | -------- | ---------------------------------------------------------------------- |
| GNU C                  | System   | `/usr/bin/gcc`                                                         |
| Intel C Compiler       | 2022.0.2 | `/opt/software/intel/oneapi/compiler/2022.0.2/linux/bin/intel64/icc`   |
| Intel Fortran Compiler | 2022.0.2 | `/opt/software/intel/oneapi/compiler/2022.0.2/linux/bin/intel64/ifort` |
| Intel MPI C            | 2021.5.1 | `/opt/software/intel/oneapi/mpi/2021.5.1/bin/mpicc`                    |
| Intel MPI Fortran      | 2021.5.1 | `/opt/software/intel/oneapi/mpi/2021.5.1/bin/mpif90`                   |

## 4. Check Available Modules

Check the modules available on the MMS system:

```bash
module avail
```

The Intel OneAPI module directory contains the following relevant modules:

```text
------------------------------------------ /opt/software/intel/oneapi/modulefiles -------------------------------------------

advisor/2022.0.0        compiler32/latest       dpct/latest
intel_ipp_intel64/2021.5.2
mpi/latest
advisor/latest          dal/2021.5.1            dpl/2021.6.0
intel_ipp_intel64/latest
oclpfpga/2022.0.1
ccl/2021.5.0            dal/2021.5.3            dpl/latest
intel_ippcp_ia32/2021.5.0
oclfpga/2022.0.2
ccl/2021.5.1            dal/latest              icc/2022.0.1
intel_ippcp_ia32/2021.5.1
oclfpga/latest
ccl/latest              debugger/2021.5.0       icc/2022.0.2
intel_ippcp_ia32/latest
tbb/2021.5.0
clck/2021.5.0           debugger/latest         icc/latest
intel_ippcp_intel64/2021.5.0
tbb/2021.5.1
clck/latest             dev-utilities/2021.5.1  icc32/2022.0.1
intel_ippcp_intel64/2021.5.1
tbb/latest
compiler-rt/2022.0.1     dev-utilities/2021.5.2  icc32/2022.0.2
intel_ippcp_intel64/latest
tbb32/2021.5.0
compiler-rt/2022.0.2     dev-utilities/latest    icc32/latest
itac/2021.5.0
tbb32/2021.5.1
compiler-rt/latest        dnnl-cpu-gomp/2022.0.2 init_opencl/2022.0.1
itac/latest
vpl/2022.0.0
compiler-rt32/2022.0.1    dnnl-cpu-gomp/latest    init_opencl/2022.0.2
mkl/2022.0.1
vpl/latest
compiler-rt32/2022.0.2    dnnl-cpu-iomp/2022.0.2 init_opencl/latest
mkl/2022.0.2
compiler/latest
compiler-rt32/latest      dnnl-cpu-iomp/latest   inspector/2022.0.0
mkl/latest
compiler32/2022.0.1       dnnl-cpu-tbb/2022.0.2  inspector/latest
mkl32/2022.0.1
compiler32/2022.0.2       dnnl-cpu-tbb/latest    itac/2021.5.0
mkl32/2022.0.2
compiler32/latest         dnnl/2022.0.2          mpi/2021.5.0
mkl32/latest
```

The system also provides a Conda module:

```text
-------------------------------------------------- /usr/share/modulefiles ---------------------------------------------------

conda/latest
```

## 5. Intel Software Environment

The important Intel OneAPI components for the WRFDA installation are:

* **Intel C Compiler:** `icc/2022.0.2`
* **Intel Fortran Compiler:** provided by the Intel compiler environment
* **Intel MPI:** `mpi/2021.5.1`
* **Intel MKL:** `mkl/2022.0.2`

These components will be used in the subsequent WRFDA compilation steps.

---

**Next step:** configure the required Intel compiler, Intel MPI, NetCDF, HDF5, Jasper, libpng, zlib, and other WRFDA dependencies before compiling WRFDA.

---
### Instal libraries
## 1.1. Define the Installation Directories

Create environment variables for the WRF library installation:

```bash
# Installation directory
export WRF_ROOT=$HOME/install_wrf
export DIR=$WRF_ROOT/libraries
```

The libraries will be installed under:

```text
$HOME/install_wrf/libraries
```

## 1.2. Define the Intel Compiler Environment

Set the Intel compiler paths:

```bash
# Intel compiler
export INTEL_ROOT=/opt/software/intel/oneapi/compiler/2022.0.2/linux/bin/intel64
export INTEL_LIB=/opt/software/intel/oneapi/compiler/2022.0.2/linux/compiler/lib/intel64_lin
```

Define the compilers:

```bash
# Compilers
export CC=icc
export CXX=icpc
export FC=ifort
export F77=ifort
export F90=ifort
```

Configure Intel MPI to use the Intel compilers as its backend:

```bash
# Intel MPI compiler backend
export I_MPI_CC=icc
export I_MPI_CXX=icpc
export I_MPI_F77=ifort
export I_MPI_F90=ifort
```

## 1.3. Configure the Intel Library Path

Unset `LD_LIBRARY_PATH` to avoid potential Intel `libimf`/`ar` linking problems:

```bash
# Avoid Intel libimf/ar problem
unset LD_LIBRARY_PATH
```

Set the library search path:

```bash
# Intel libraries for linking
export LIBRARY_PATH=$INTEL_LIB:$DIR/lib:$LIBRARY_PATH
export LDFLAGS="-L$INTEL_LIB -L$DIR/lib"
export CPPFLAGS="-I$DIR/include"
```

# 1. Download and Install Zlib 1.3.1

Zlib is one of the required libraries for the WRF/WRFDA build.

The following installation uses:

* **Zlib:** 1.3.1
* **Intel ICC:** 2022.0.2
* **Intel MPI:** 2021.5.1
* **System:** MMS HPC

Move to the library installation directory:

```bash
cd $DIR
wget -c https://github.com/madler/zlib/releases/download/v1.3.1/zlib-1.3.1.tar.gz
tar xvzf zlib-1.3.1.tar.gz
cd zlib-1.3.1
CC=mpicc ./configure --prefix=$DIR
make -j4
make install
```


Check that the Zlib library files have been installed:

```bash
ls -lh $DIR/lib/libz*
```

Check the Zlib header files:

```bash
ls -lh $DIR/include/zlib.h
ls -lh $DIR/include/zconf.h
```

A successful installation should provide files similar to:

```text
$DIR/lib/libz.so
$DIR/lib/libz.a
$DIR/include/zlib.h
$DIR/include/zconf.h
```
# 2. Download and Install libpng 1.6.39

```bash
cd $DIR

wget -c https://download.sourceforge.net/libpng/libpng-1.6.39.tar.gz
tar xvzf libpng-1.6.39.tar.gz
cd libpng-1.6.39

make distclean 2>/dev/null || true

CC=mpicc ./configure \
    --prefix=$DIR \
    CPPFLAGS="-I$DIR/include" \
    LDFLAGS="-L$DIR/lib -L$INTEL_LIB"

make -j4
make install
```

Check that the libpng library files have been installed:
```bash
ls -lh $DIR/lib/libpng*
ls -lh $DIR/include/png*
LD_LIBRARY_PATH="$INTEL_LIB:$DIR/lib" ldd $DIR/lib/libpng.so.16.39.0
```

