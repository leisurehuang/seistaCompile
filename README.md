# Siesta编译教程

## 1. 安装 Ubuntu 11.10 中文版
  在 Ubuntu 图形界面下，运行 `terminal`。（不要 `Ctrl-Alt-F1`，否则中文显示有问题，并且无法使用 gedit 等软件来编辑文本）
  默认是进入自己的家目录（`~/`）下，我的是`/home/leisure`
  先建立两个子目录:
  		
  		cd ~
  		mkdir sourcecode
  		mkdir lib

分别存放源码，以及将来编译好的数学库文件。数学库文件也可以使用系统默认的路径，比如 `/usr/lib/`。

所有需要编译的库的源文件如果大家不好找，来我的网盘下载：<https://pan.baidu.com/s/1mhM0mUk>

## 2. 安装` g++` 以及 `gfortran`

    	sudo apt-get update
    	sudo apt-get install build-essential
    	sudo apt-get install gfortran

## 3. 安装 `mpich2`

到 <http://www.mcs.anl.gov/research/projects/mpich2/downloads/index.php?s=downloads> 下载最新版的 `mpich2`
解压缩到 `~/sourcecode/mpich2`

### 3.1获取当前电脑的CPU类型  
怎样获得自己 CPU 合适的 `C/Fortran FLAGS`呢？
运行:

	echo "" | gcc -march=native -v -E - 2>&1 | grep cc1

我这里的结果是：

		/usr/lib/gcc/x86_64-linux-gnu/4.6.1/cc1 -E -quiet -v -imultilib . -imultiarch x86_64-linux-gnu - -march=corei7-avx -mcx16 -msahf -mno-movbe -maes -mpclmul -mpopcnt -mno-abm -mno-lwp -mno-fma -mno-fma4 -mno-xop -mno-bmi -mno-tbm -mavx -msse4.2 -msse4.1 --param l1-cache-size=32 --param l1-cache-line-size=64 --param l2-cache-size=8192 -mtune=corei7-avx -fstack-protector

一大串，关键词是：`corei7-avx`


### 3.2编译mpich2：

		cd ~/sourcecode/mpich2/
        
		./configure --prefix=/opt/mpich2 --enable-fc CC=/usr/bin/gcc CXX=/usr/bin/g++ FC=/usr/bin/gfortran F77=/usr/bin/gfortran FCFLAGS="-O2 -march=corei7-avx" CFLAGS="-O2 -march=corei7-avx"
        
		make
		
		sudo make install
        
		export PATH=/opt/mpich2/bin:$PATH
        
		export LD_LIBRARY_PATH=/opt/mpich2/lib:$LD_LIBRARY_PATH

**参数说明**：`--prefix` 就是欲安装的路径，这里装到 `/opt/mpich2/` 下面。`--enable-fc` 要求生成 `fortran` 编译器，但其实不加它也一样。CC，CXX，FC，F77等指定了 C，C++ 和 `Fortran` 的编译器的绝对地址。

**FLAGS**：这里只是指明 CPU 为 `Intel Core i7`。  

按照以上脚本，用 gfortran 和 gcc/g++ 编译好的 mpich2 的可执行文件位于`/opt/mpich2/bin`中。

## 4. 安装 blas 和 lapack

下载 lapack 3.4.0
<http://www.netlib.org/lapack/lapack-3.4.0.tgz>

解压缩到 `~/sourcecode/lapack-3.4.0`  

		cd ~/sourcecode/lapack-3.4.0
		
		cp INSTALL/make.inc.gfortran ./make.inc
		
		gedit make.inc

在头部关于编译器选择的一大块，用以下代换：

	FORTRAN  = /usr/bin/gfortran
	OPTS     = -O2 -march=corei7-avx
	DRVOPTS  = $(OPTS)
	NOOPT    = -O2 -march=corei7-avx
	LOADER   = /usr/bin/gfortran
	LOADOPTS =

保存关闭。然后执行  

	make blaslib
	
	make lapacklib

`blas` 比较容易编译。`lapack` 则一定要设置正确，如上所述。

编译好以后，将 `librefblas.a` 和 `liblapack.a` 拷贝到 ~/lib 下面，然后

	export LD_LIBRARY_PATH=/home/leisure/lib:$LD_LIBRARY_PATH

## 5. 安装 BLACS

下载 `BLACS` 的 `tarball` 并解压缩到 `~/sourcecode/BLACS`
要注意选择 `mpi` 版本的 `BLACS`
比如： <http://www.netlib.org/blacs/mpiblacs.tgz>

	cd ~/sourcecode/BLACS/
	gedit Bmake.inc

复制以下内容并保存。其中的 `FLAGS` (有关 Intel Core i7 的部分)和`home`路径（我的用户名为leisure） 应该根据自己的修改。

<pre><code>
#=========================================================
#============ SECTION 1: PATHS AND LIBRARIES =============
#=========================================================
#  The following macros specify the name and location of libraries required by
#  the BLACS and its tester.
#=========================================================

#  --------------------------------------
#  Make sure we've got a consistent shell
#  --------------------------------------
  SHELL = /bin/sh

#  -----------------------------
#  The top level BLACS directory
#  -----------------------------
  BTOPdir = /home/leisure/sourcecode/BLACS

#  -----------------------------------------------------------------
#  The communication library your BLACS have been written for.
#  Known choices (and the machines they run on) are:
#
#     COMMLIB   MACHINE
#     .......   ..............................................................
#     CMMD      Thinking Machine's CM-5
#     MPI       Wide variety of systems
#     MPL       IBM's SP series (SP1 and SP2)
#     NX        Intel's supercomputer series (iPSC2, iPSC/860, DELTA, PARAGON)
#     PVM       Most unix machines; See PVM User's Guide for details
#  -----------------------------------------------------------------
  COMMLIB = MPI

#  -------------------------------------------------------------
#  The platform identifier to suffix to the end of library names
#  -------------------------------------------------------------
  PLAT = LINUX

#  ----------------------------------------------------------
#  Name and location of the BLACS library.  See section 2 for
#  details on BLACS debug level (BLACSDBGLVL).
#  ----------------------------------------------------------
  BLACSdir    = $(BTOPdir)/LIB
  BLACSDBGLVL = 0
  BLACSFINIT  = $(BLACSdir)/blacsF77init_$(COMMLIB)-$(PLAT)-$(BLACSDBGLVL).a
  BLACSCINIT  = $(BLACSdir)/blacsCinit_$(COMMLIB)-$(PLAT)-$(BLACSDBGLVL).a
  BLACSLIB    = $(BLACSdir)/blacs_$(COMMLIB)-$(PLAT)-$(BLACSDBGLVL).a

#  -------------------------------------
#  Name and location of the MPI library.
#  -------------------------------------
  MPIdir = /opt/mpich2/
  MPILIBdir = $(MPIdir)/lib/
  MPIINCdir = $(MPIdir)/include
  MPILIB = $(MPILIBdir)/libmpichf90.a

#  -------------------------------------
#  All libraries required by the tester.
#  -------------------------------------
  BTLIBS = $(BLACSFINIT) $(BLACSLIB) $(BLACSFINIT) $(MPILIB)

#  ----------------------------------------------------------------
#  The directory to put the installation help routines' executables
#  ----------------------------------------------------------------
  INSTdir = $(BTOPdir)/INSTALL/EXE

#  ------------------------------------------------
#  The name and location of the tester's executable
#  ------------------------------------------------
  TESTdir = $(BTOPdir)/TESTING/EXE
  FTESTexe = $(TESTdir)/xFbtest_$(COMMLIB)-$(PLAT)-$(BLACSDBGLVL)
  CTESTexe = $(TESTdir)/xCbtest_$(COMMLIB)-$(PLAT)-$(BLACSDBGLVL)
#=========================================================
#===================== End SECTION 1 =====================
#=========================================================


#=========================================================
#=============== SECTION 2: BLACS INTERNALS ==============
#=========================================================
#  The following macro definitions set preprocessor values for the BLACS.
#  The file Bconfig.h sets these values if they are not set by the makefile.
#  User's compiling only the tester can skip this entire section.
#  NOTE: The MPI defaults have been set for MPICH.
#=========================================================

#  -------------------------------------------------------------
#  The directory to find the required communication library include files,
#  if they are required by your system.
#  -------------------------------------------------------------
  SYSINC = -I$(MPIINCdir)

#  -----------------------------------------------------------------
#  The Fortran 77 to C interface to be used.  If you are unsure of the correct
#  setting for your platform, compile and run BLACS/INSTALL/xintface.
#  Choices are: Add_, NoChange, UpCase, or f77IsF2C.
#  -----------------------------------------------------------------
  INTFACE = -Df77IsF2C

#  ------------------------------------------------------------------------
#  Allows the user to vary the topologies that the BLACS default topologies
#  (TOP = ' ') correspond to.  If you wish to use a particular topology
#  (as opposed to letting the BLACS make the choice), uncomment the
#  following macros, and replace the character in single quotes with the
#  topology of your choice.
#  ------------------------------------------------------------------------
#  DEFBSTOP   = -DDefBSTop="'1'"
#  DEFCOMBTOP = -DDefCombTop="'1'"

#  -------------------------------------------------------------------
#  If your MPI_Send is locally-blocking, substitute the following line
#  for the empty macro definition below.
#  SENDIS = -DSndIsLocBlk
#  -------------------------------------------------------------------
  SENDIS =

#  --------------------------------------------------------------------
#  If your MPI handles packing of non-contiguous messages by copying to
#  another buffer or sending extra bytes, better performance may be
#  obtained by replacing the empty macro definition below with the
#  macro definition on the following line.
#  BUFF = -DNoMpiBuff
#  --------------------------------------------------------------------
  BUFF =

#  -----------------------------------------------------------------------
#  If you know something about your system, you may make it easier for the
#  BLACS to translate between C and fortran communicators.  If the empty
#  macro defininition is left alone, this translation will cause the C
#  BLACS to globally block for MPI_COMM_WORLD on calls to BLACS_GRIDINIT
#  and BLACS_GRIDMAP.  If you choose one of the options for translating
#  the context, neither the C or fortran calls will globally block.
#  If you are using MPICH, or a derivitive system, you can replace the
#  empty macro definition below with the following (note that if you let
#  MPICH do the translation between C and fortran, you must also indicate
#  here if your system has pointers that are longer than integers.  If so,
#  define -DPOINTER_64_BITS=1.)  For help on setting TRANSCOMM, you can
#  run BLACS/INSTALL/xtc_CsameF77 and BLACS/INSTALL/xtc_UseMpich as
#  explained in BLACS/INSTALL/README.
#   TRANSCOMM = -DUseMpich
#
#  If you know that your MPI uses the same handles for fortran and C
#  communicators, you can replace the empty macro definition below with
#  the macro definition on the following line.
 TRANSCOMM = -DCSameF77
#  -----------------------------------------------------------------------
#  TRANSCOMM =

#  --------------------------------------------------------------------------
#  You may choose to have the BLACS internally call either the C or Fortran77
#  interface to MPI by varying the following macro.  If TRANSCOMM is left
#  empty, the C interface BLACS_GRIDMAP/BLACS_GRIDINIT will globally-block if
#  you choose to use the fortran internals, and the fortran interface will
#  block if you choose to use the C internals.  It is recommended that the
#  user leave this macro definition blank, unless there is a strong reason
#  to prefer one MPI interface over the other.
#  WHATMPI = -DUseF77Mpi
#  WHATMPI = -DUseCMpi
#  --------------------------------------------------------------------------
  WHATMPI =

#  ---------------------------------------------------------------------------
#  Some early versions of MPICH and its derivatives cannot handle user defined
#  zero byte data types.  If your system has this problem (compile and run
#  BLACS/INSTALL/xsyserrors to check if unsure), replace the empty macro
#  definition below with the macro definition on the following line.
#  SYSERRORS = -DZeroByteTypeBug
#  ---------------------------------------------------------------------------
  SYSERRORS =

#  ------------------------------------------------------------------
#  These macros set the debug level for the BLACS.  The fastest
#  code is produced by BlacsDebugLvl 0.  Higher levels provide
#  more debug information at the cost of performance.  Present levels
#  of debug are:
#  0 : No debug information
#  1 : Mainly parameter checking.
#  --------------------------------------------------------------
  DEBUGLVL = -DBlacsDebugLvl=$(BLACSDBGLVL)

#  ---------------------------------------------------------------
#  All BLACS definitions needed for compile (DEFS1 contains definitions used
#  by all BLACS versions).
#  ---------------------------------------------------------------
  DEFS1 = -DSYSINC $(SYSINC) $(INTFACE) $(DEFBSTOP) $(DEFCOMBTOP) $(DEBUGLVL)
  BLACSDEFS = $(DEFS1) $(SENDIS) $(BUFF) $(TRANSCOMM) $(WHATMPI) $(SYSERRORS)
#=========================================================
#===================== End SECTION 2 =====================
#=========================================================


#=========================================================
#================= SECTION 3: COMPILERS ==================
#=========================================================
#  The following macros specify compilers, linker/loaders, the archiver,
#  and their options.  Some of the fortran files need to be compiled with no
#  optimization.  This is the F77NO_OPTFLAG.  The usage of the remaining
#  macros should be obvious from the names.
#=========================================================
  F77            = /opt/mpich2/bin/mpif77
  F77NO_OPTFLAGS = -O0
  F77FLAGS       = -O2 -march=corei7-avx
  F77LOADER      = $(F77)
  F77LOADFLAGS   =
  CC             = /opt/mpich2/bin/mpicc
  CCFLAGS        = -O2 -march=corei7-avx
  CCLOADER       = $(CC)
  CCLOADFLAGS    =

#  ----------------------------------------------------------------
#  The archiver and the flag(s) to use when building an archive (library).
#  Also the ranlib routine.  If your system has no ranlib, set RANLIB = echo.
#  ----------------------------------------------------------------
  ARCH      = ar
  ARCHFLAGS = r
  RANLIB    = ranlib

#=========================================================
#===================== End SECTION 3 =====================
#=========================================================
</code></pre>

然后执行：

	make mpi

最后把 `./LIB` 子目录下面产生的3个 `.a` 文件拷贝到 `~/lib/` 下面

## 6. 安装 scalapack

下载 scalapack 2.0.1
<http://www.netlib.org/scalapack/scalapack-2.0.1.tgz>

解压缩到`~/sourcecode/scalapack-2.0.1`

	cd ~/sourcecode/scalapack-2.0.1
	gedit SLmake.inc

复制以下内容并保存:

<pre><code>
##################################################################
#
#  Program:         ScaLAPACK
#
#  Module:          SLmake.inc
#
#  Purpose:         Top-level Definitions
#
#  Creation date:   February 15, 2000
#
#  Modified:        October 13, 2011
#
#  Send bug reports, comments or suggestions to scalapack@cs.utk.edu
#
##################################################################
#
#  C preprocessor definitions:  set CDEFS to one of the following:
#
#     -DNoChange (fortran subprogram names are lower case without any suffix)
#     -DUpCase   (fortran subprogram names are upper case without any suffix)
#     -DAdd_     (fortran subprogram names are lower case with "_" appended)

CDEFS         = -DAdd_

#
#  The fortran and C compilers, loaders, and their flags
#

FC            = /opt/mpich2/bin/mpif77
CC            = /opt/mpich2/bin/mpicc
NOOPT         = -O0
FCFLAGS       = -O2 -march=corei7-avx
CCFLAGS       = -O2 -march=corei7-avx
FCLOADER      = $(FC)
CCLOADER      = $(CC)
FCLOADFLAGS   = $(FCFLAGS)
CCLOADFLAGS   = $(CCFLAGS)

#
#  The archiver and the flag(s) to use when building archive (library)
#  Also the ranlib routine.  If your system has no ranlib, set RANLIB = echo
#

ARCH          = ar
ARCHFLAGS     = cr
RANLIB        = ranlib

#
#  The name of the ScaLAPACK library to be created
#

SCALAPACKLIB  = libscalapack.a

#
#  BLAS, LAPACK (and possibly other) libraries needed for linking test programs
#

BLASLIB       = /home/leisure/lib/librefblas.a
LAPACKLIB     = /home/leisure/lib/liblapack.a
LIBS          = $(LAPACKLIB) $(BLASLIB)
</code></pre>

其中注意根据自己的 CPU 修改 `-march=corei7-avx` 以及home目录的路径(我这里是用户名为leisure，请修改为自己的用户名)。各个编译器以及库文件都要指定绝对路径，以免编译的时候找不到。

然后:

	make

将编译好的 `libscalapack.a` 拷贝到 `~/lib/` 下面。

## 7. 安装 Siesta

到 <http://www.icmab.es/dmmis/leem/siesta/CodeAccess/selector.html> 下载 Siesta的源代码，解压缩到 ~/sourcecode/siesta-xx.xx.xx


第一步，cd到siesta中`Obj`目录下面

 	sh ../Src/obj_setup.sh

之后../Src目录里面会生成configure文件，继续Obj目录下执行:

	../Src/configure --enable-mpi FC=/opt/mpich2/bin/mpif90


配置成功后，根据提示编辑`arch.make`文件 

	gedit arch.make

修改一些编译器的路径，改为绝对路径。具体来说，我的 arch.make 修改为：


<pre><code>
#
# This file is part of the SIESTA package.
#
# Copyright (c) Fundacion General Universidad Autonoma de Madrid:
# E.Artacho, J.Gale, A.Garcia, J.Junquera, P.Ordejon, D.Sanchez-Portal
# and J.M.Soler, 1996-2006.
#
# Use of this software constitutes agreement with the full conditions
# given in the SIESTA license, as signed by all legitimate users.
#
.SUFFIXES:
.SUFFIXES: .f .F .o .a .f90 .F90

SIESTA_ARCH=x86_64-unknown-linux-gnu--unknown

FPP=
FPP_OUTPUT=
FC=/opt/mpich2/bin/mpif90
RANLIB=ranlib

SYS=nag

SP_KIND=4
DP_KIND=8
KINDS=$(SP_KIND) $(DP_KIND)

FFLAGS=-g -O2
FPPFLAGS= -DMPI -DFC_HAVE_FLUSH -DFC_HAVE_ABORT
LDFLAGS=

ARFLAGS_EXTRA=

FCFLAGS_fixed_f=
FCFLAGS_free_f90=
FPPFLAGS_fixed_F=
FPPFLAGS_free_F90=

HOME_LIB=/home/leisure/lib
BLAS_LIBS=$(HOME_LIB)/librefblas.a
LAPACK_LIBS=$(HOME_LIB)/liblapack.a
BLACS_LIBS=$(HOME_LIB)/blacsCinit_MPI-LINUX-0.a $(HOME_LIB)/blacsF77init_MPI-LINUX-0.a $(HOME_LIB)/blacs_MPI-LINUX-0.a
SCALAPACK_LIBS=$(HOME_LIB)/libscalapack.a

COMP_LIBS=dc_lapack.a liblapack.a libblas.a

NETCDF_LIBS=
NETCDF_INTERFACE=

LIBS=$(SCALAPACK_LIBS) $(BLACS_LIBS) $(LAPACK_LIBS) $(BLAS_LIBS) $(NETCDF_LIBS)

#SIESTA needs an F90 interface to MPI
#This will give you SIESTA's own implementation
#If your compiler vendor offers an alternative, you may change
#to it here.
MPI_INTERFACE=libmpi_f90.a
MPI_INCLUDE=.
MPI_LIB=/opt/mpich2/lib
DEFS_MPI=-DMPI

#Dependency rules are created by autoconf according to whether
#discrete preprocessing is necessary or not.
.F.o:
   $(FC) -c $(FFLAGS) $(INCFLAGS) $(FPPFLAGS) $(FPPFLAGS_fixed_F)  $<
.F90.o:
   $(FC) -c $(FFLAGS) $(INCFLAGS) $(FPPFLAGS) $(FPPFLAGS_free_F90) $<
.f.o:
   $(FC) -c $(FFLAGS) $(INCFLAGS) $(FCFLAGS_fixed_f)  $<
.f90.o:
   $(FC) -c $(FFLAGS) $(INCFLAGS) $(FCFLAGS_free_f90)  $<
</code></pre>

这里注意:

	MPI_INTERFACE=libmpi_f90.a
	MPI_INCLUDE=.

这两个 MPI interface 的设置。不要使用自己生成的 mpich2 路径下的文件，而要采用 Siesta 自带的库文件。但所有的数学库都采用刚刚编译好的 `~/lib/` 下面的那些文件。

configure完成后，继续在`Obj`目录里面:

	make
编译完成后会在`Obj`目录里面找到siesta的可执行文件

尝试:

	./siesta

如果显示类似：

	Siesta Version: siesta-2.0.2
	Architecture  : x86_64-unknown-linux-gnu--unknown
	Compiler flags: /opt/mpich2/bin/mpif90 -g -O2
	PARALLEL version

	* Running in serial mode with MPI

其中含有 `PARALLEL version`，则说明已经并行化。按 `Ctrl-C` 退出运行。

	sudo cp ./siesta /usr/local/bin/

将编译好的二进制文件拷贝到自己想要安装的路径下，比如 `/usr/local/bin/`

运行 Siesta 的时候，比如在 `~/` 下面建立一个 sample 目录，并将 `fdf` 文件命名为 in.fdf，将赝势文件 (`.psf`) 也拷贝进来，则

	cd ~/sample/
	/opt/mpich2/bin/mpiexec -np 8 siesta < in.fdf | tee out.out

则以 8 个线程运行，屏幕内容输出到 `out.out` 文件。

为了下次登陆的时候有合适的环境变量，还需要执行

	gedit ~/.bashrc

加入两行：

	export PATH=/opt/mpich2/bin:$PATH
	export LD_LIBRARY_PATH=/opt/mpich2/lib:/home/leisure/lib:$LD_LIBRARY_PATH

保存即可。


大功告成！