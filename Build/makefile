# Makefile for Fire Dynamics Simulator (FDS)

# To use this makefile, select the appropriate compiler and OS from the list below. For example, to
# compile the parallel version (MPI) of FDS under Linux using the Intel compilers type:
#
# make impi_intel_linux

# A make utility for windows may be downloaded from http://gnuwin32.sourceforge.net/packages/make.htm

# The variable VPATH defines where the source code is relative to the current directory

VPATH = ../Source

# set compiler type

I_IFORT=ifort
ifdef INTEL_IFORT
   I_IFORT=$(INTEL_IFORT)
endif

ifeq ($(shell echo "check_quotes"),"check_quotes")
# windows
  GIT_HASH := $(shell ..\Scripts\githash)
  GIT_STAT := $(shell ..\Scripts\gitstat)
  GIT_DATE := $(shell ..\Scripts\gitlog)
  GIT_BRANCH := $(shell ..\Scripts\gitbranch)
  BUILD_DATE := $(shell date /t) $(shell time /t)
else
# linux/osx
  SHELL:=/bin/bash
  GIT_HASH := $(shell git describe --long --abbrev=7)
  ifeq ("$(shell git diff --shortstat $(VPATH)/*.f90)","")
    GIT_STAT :=
  else
    GIT_STAT := -dirty
  endif
  GIT_DATE := $(shell git log -1 --format=%cd)
  GIT_BRANCH := $(shell git rev-parse --abbrev-ref HEAD)
  BUILD_DATE := $(shell date "+%b %d, %Y  %T")
# The preprocessor used by xlf uses comma as separation between preprocessing commands.
# Commas needs to be escaped if they are to be used as part of a value in a preprocessing variable.
  BUILD_DATE_XLF := $(shell date "+%b %d\, %Y  %T")
# For OSX get CLT version:
  CLT_VERSION := $(shell ../Scripts/osx_clt.sh)
endif

GITINFO_BASE=-DGITHASH_PP=\"$(GIT_HASH)$(GIT_STAT)-$(GIT_BRANCH)\" -DGITDATE_PP=\""$(GIT_DATE)\""
GITINFO=-fpp $(GITINFO_BASE) -DBUILDDATE_PP=\""$(BUILD_DATE)\""
GITINFOGNU=-cpp $(GITINFO_BASE) -DBUILDDATE_PP=\""$(BUILD_DATE)\""
GITINFOXLF=-qsuffix=cpp=f90 '-WF,-DGITHASH_PP=\"$(GIT_HASH)\",-DGITDATE_PP=\"$(GIT_DATE)\",-DBUILDDATE_PP=\"$(BUILD_DATE_XLF)\"'
GITINFOPGI=-Mpreprocess $(GITINFO_BASE) -DBUILDDATE_PP=\""$(BUILD_DATE)\""
GFORTRAN_OPTIONS=

FFLAGSMKL_INTEL =
LFLAGSMKL_INTEL =
FFLAGSMKL_OPENMPI =
LFLAGSMKL_OPENMPI =
FFLAGSMKL_CUSTOM =
LFLAGSMKL_CUSTOM =
FFLAGS_SUNDIALS =
LFLAGS_SUNDIALS =
ifdef MKLROOT # This assumes the MKL library is available.
ifeq ($(shell echo "check_quotes"),"check_quotes")
# windows
# mkl libraries has changed location in Intel oneAPI version 2024 and later
# MKLLIBDIR location is defined in havemkl.bat script
ifeq ($(shell ..\Scripts\havemkl),1)
# Use cluster_sparse_solver with intelmpi blacs
  FFLAGSMKL_INTEL = -DWITH_MKL /I"$(MKLROOT)"\include
  LFLAGSMKL_INTEL = "$(MKLLIBDIR)"\mkl_intel_lp64.lib "$(MKLLIBDIR)"\mkl_sequential.lib "$(MKLLIBDIR)"\mkl_core.lib "$(MKLLIBDIR)"\mkl_blacs_intelmpi_lp64.lib
  LFLAGSMKL_INTEL_OPENMP = "$(MKLLIBDIR)"\mkl_intel_lp64.lib "$(MKLLIBDIR)"\mkl_intel_thread.lib "$(MKLLIBDIR)"\mkl_core.lib "$(MKLLIBDIR)"\mkl_blacs_intelmpi_lp64.lib
endif
else
# linux
ifneq ("$(wildcard ${MKLROOT}/lib/intel64/libmkl_blacs_intelmpi_lp64.a)","")
# Use cluster_sparse_solver with intelmpi blacs
  FFLAGSMKL_INTEL = -DWITH_MKL -I"${MKLROOT}/include"
  LFLAGSMKL_INTEL = -Wl,--start-group ${MKLROOT}/lib/intel64/libmkl_intel_lp64.a ${MKLROOT}/lib/intel64/libmkl_sequential.a ${MKLROOT}/lib/intel64/libmkl_core.a ${MKLROOT}/lib/intel64/libmkl_blacs_intelmpi_lp64.a -Wl,--end-group -lpthread -lm -ldl
  LFLAGSMKL_INTEL_OPENMP = -Wl,--start-group ${MKLROOT}/lib/intel64/libmkl_intel_lp64.a ${MKLROOT}/lib/intel64/libmkl_intel_thread.a ${MKLROOT}/lib/intel64/libmkl_core.a ${MKLROOT}/lib/intel64/libmkl_blacs_intelmpi_lp64.a -Wl,--end-group -liomp5 -lpthread -lm -ldl
endif
ifneq ("$(wildcard ${MKLROOT}/lib/intel64/libmkl_blacs_openmpi_lp64.a)","")
# Use cluster_sparse_solver with openmpi blacs
  FFLAGSMKL_OPENMPI = -DWITH_MKL -I$(MKLROOT)/include
  LFLAGSMKL_OPENMPI = -Wl,--start-group ${MKLROOT}/lib/intel64/libmkl_intel_lp64.a ${MKLROOT}/lib/intel64/libmkl_core.a ${MKLROOT}/lib/intel64/libmkl_intel_thread.a ${MKLROOT}/lib/intel64/libmkl_blacs_openmpi_lp64.a -Wl,--end-group -lpthread -lm -ldl
  FFLAGSMKL_GNU_OPENMPI = -DWITH_MKL -I$(MKLROOT)/include
  LFLAGSMKL_GNU_OPENMPI =  -Wl,--start-group ${MKLROOT}/lib/intel64/libmkl_gf_lp64.a ${MKLROOT}/lib/intel64/libmkl_gnu_thread.a ${MKLROOT}/lib/intel64/libmkl_core.a ${MKLROOT}/lib/intel64/libmkl_blacs_openmpi_lp64.a -Wl,--end-group -lgomp -lpthread -lm -ldl
endif
ifneq ("$(wildcard ${MKLROOT}/lib/libmkl_blacs_custom_lp64.a)","")
# Use cluster_sparse_solver with custom blacs
  FFLAGSMKL_CUSTOM = -DWITH_MKL -I$(MKLROOT)/include
# MKL_LIBS needs to occur 3 times in order to avoid unresolved external reference warnings
  MKL_LIBS_OPENMP = ${MKLROOT}/lib/libmkl_intel_lp64.a ${MKLROOT}/lib/libmkl_core.a ${MKLROOT}/lib/libmkl_intel_thread.a ${MKLROOT}/lib/libmkl_blacs_custom_lp64.a
  MKL_LIBS = ${MKLROOT}/lib/libmkl_intel_lp64.a ${MKLROOT}/lib/libmkl_core.a ${MKLROOT}/lib/libmkl_sequential.a ${MKLROOT}/lib/libmkl_blacs_custom_lp64.a
  LFLAGSMKL_CUSTOM_OPENMP = ${MKL_LIBS_OPENMP} ${MKL_LIBS_OPENMP} ${MKL_LIBS_OPENMP} -lpthread -lm -ldl
  LFLAGSMKL_CUSTOM = ${MKL_LIBS} ${MKL_LIBS} ${MKL_LIBS} -lpthread -lm -ldl
  FFLAGSMKL_GNU_CUSTOM = -DWITH_MKL -I$(MKLROOT)/include
  LFLAGSMKL_GNU_CUSTOM =  ${MKLROOT}/lib/libmkl_intel_lp64.a ${MKLROOT}/lib/libmkl_sequential.a ${MKLROOT}/lib/libmkl_core.a ${MKLROOT}/lib/libmkl_blacs_custom_lp64.a -lpthread -lm -ldl
endif
endif # OS test
endif # MKLROOT test


ifdef SUNDIALS_HOME # This assumes the SUNDIALS library is available.
   FFLAGS_SUNDIALS = -DWITH_SUNDIALS -I"${SUNDIALS_HOME}/fortran"
   FFLAGS_SUNDIALS_WIN = -DWITH_SUNDIALS /I"${SUNDIALS_HOME}\fortran"
   LFLAGS_SUNDIALS = -L${SUNDIALS_HOME}/lib64 -l:libsundials_fcvode_mod.a -l:libsundials_fnvecserial_mod.a -l:libsundials_cvode.a
   LFLAGS_SUNDIALS_INTEL = ${SUNDIALS_HOME}/lib64/libsundials_fcvode_mod.a ${SUNDIALS_HOME}/lib64/libsundials_fnvecserial_mod.a ${SUNDIALS_HOME}/lib64/libsundials_cvode.a
   LFLAGS_SUNDIALS_OSX = ${SUNDIALS_HOME}/lib/libsundials_fcvode_mod.a ${SUNDIALS_HOME}/lib/libsundials_fnvecserial_mod.a ${SUNDIALS_HOME}/lib/libsundials_cvode.a
   LFLAGS_SUNDIALS_WIN = ${SUNDIALS_HOME}/lib/sundials_fcvode_mod.lib ${SUNDIALS_HOME}/lib/sundials_fnvecserial_mod.lib ${SUNDIALS_HOME}/lib/sundials_cvode.lib  /link /NODEFAULTLIB:MSVCRTD /NODEFAULTLIB:libcmtd.lib
endif

obj_mpi = prec.o cons.o chem.o prop.o devc.o type.o data.o mesh.o func.o gsmv.o smvv.o rcal.o turb.o soot.o \
          pois.o geom.o ccib.o radi.o part.o vege.o ctrl.o hvac.o mass.o \
          wall.o fire.o velo.o pres.o init.o dump.o read.o divg.o main.o

objwin_mpi = $(obj_mpi:.o=.obj)

# General Purpose Rules

no_target:
	@echo \******** You did not specify a make target \********
	@echo Please read the comments at the top of the makefile

setup:
%.o : %.mod

setup_win:
%.obj : %.mod

.SUFFIXES: .f90 .o .obj

.f90.o:
	$(FCOMPL) -c $(FFLAGS) $<
.f90.obj:
	$(FCOMPL) -c $(FFLAGS) $<

# Files which have OpenMP compiler directives are compiled using the FOPENMPFLAG and can be enabled/disabled on a per-file basis.

divg.o: FFLAGS += $(FOPENMPFLAGS)
func.o: FFLAGS += $(FOPENMPFLAGS)
mass.o: FFLAGS += $(FOPENMPFLAGS)
pres.o: FFLAGS += $(FOPENMPFLAGS)
turb.o: FFLAGS += $(FOPENMPFLAGS)
velo.o: FFLAGS += $(FOPENMPFLAGS)
radi.o: FFLAGS += $(FOPENMPFLAGS)
wall.o: FFLAGS += $(FOPENMPFLAGS)
main.o: FFLAGS += $(FOPENMPFLAGS)

divg.obj: FFLAGS += $(FOPENMPFLAGS)
func.obj: FFLAGS += $(FOPENMPFLAGS)
mass.obj: FFLAGS += $(FOPENMPFLAGS)
pres.obj: FFLAGS += $(FOPENMPFLAGS)
turb.obj: FFLAGS += $(FOPENMPFLAGS)
velo.obj: FFLAGS += $(FOPENMPFLAGS)
radi.obj: FFLAGS += $(FOPENMPFLAGS)
wall.obj: FFLAGS += $(FOPENMPFLAGS)
main.obj: FFLAGS += $(FOPENMPFLAGS)

# Windows Intel Fortran Compiler and Intel MPI
#*** Note: the script, Build\Scripts\setup_intel_compilers.bat, must be run before running make with this target

impi_intel_win : MPILIB = "$(IMPI_RELEASE_ROOT)\impi.lib"
impi_intel_win : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win : FFLAGS = -D_WIN32 /Qipo /O2 /I$(MPIINCLUDE) /wrap-margin- $(GITINFO) $(FFLAGSMKL_INTEL) $(FFLAGS_SUNDIALS_WIN) /Qdiag-disable:10448
impi_intel_win : FCOMPL = $(I_IFORT)
impi_intel_win : obj = fds_impi_intel_win
impi_intel_win : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL) $(LFLAGS_SUNDIALS_WIN)

impi_intel_win_vt : MPILIB = "$(IMPI_RELEASE_ROOT)\impi.lib"
impi_intel_win_vt : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win_vt : FFLAGS = -D_WIN32 /Qipo /O2 /I$(MPIINCLUDE) /wrap-margin- $(GITINFO) $(FFLAGSMKL_INTEL) $(FFLAGS_SUNDIALS_WIN) /Qdiag-disable:10448 /Zi /D "TBB_USE_THREADING_TOOLS"
impi_intel_win_vt : FCOMPL = $(I_IFORT)
impi_intel_win_vt : obj = fds_impi_intel_win_vt
impi_intel_win_vt : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL) $(LFLAGS_SUNDIALS_WIN)


impi_intel_win_openmp : MPILIB = "$(IMPI_RELEASE_ROOT)\impi.lib"
impi_intel_win_openmp : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win_openmp : FFLAGS = -D_WIN32 /Qipo /O2 /I$(MPIINCLUDE) /wrap-margin- $(GITINFO) $(FFLAGSMKL_INTEL) $(FFLAGS_SUNDIALS_WIN) /Qdiag-disable:10448
impi_intel_win_openmp : FOPENMPFLAGS = /Qopenmp
impi_intel_win_openmp : FCOMPL = $(I_IFORT)
impi_intel_win_openmp : obj = fds_impi_intel_win_openmp
impi_intel_win_openmp : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) $(FOPENMPFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL_OPENMP) $(LFLAGS_SUNDIALS_WIN)

impi_intel_win_dv : MPILIB = "$(IMPI_RELEASE_ROOT)\impi.lib"
impi_intel_win_dv : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win_dv : FFLAGS = -D_WIN32 /O1 /I$(MPIINCLUDE) /Z7 /traceback /debug:full /wrap-margin- $(GITINFO) $(FFLAGSMKL_INTEL) $(FFLAGS_SUNDIALS_WIN) /Qdiag-disable:10448
impi_intel_win_dv : FCOMPL = $(I_IFORT)
impi_intel_win_dv : obj = fds_impi_intel_win_dv
impi_intel_win_dv : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL) $(LFLAGS_SUNDIALS_WIN)

impi_intel_win_openmp_dv : MPILIB = "$(IMPI_RELEASE_ROOT)\impi.lib"
impi_intel_win_openmp_dv : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win_openmp_dv : FFLAGS = -D_WIN32 /O1 /I$(MPIINCLUDE) /Z7 /traceback /debug:full /wrap-margin- $(GITINFO) $(FFLAGSMKL_INTEL) $(FFLAGS_SUNDIALS_WIN) /Qdiag-disable:10448
impi_intel_win_openmp_dv : FOPENMPFLAGS = /Qopenmp
impi_intel_win_openmp_dv : FCOMPL = $(I_IFORT)
impi_intel_win_openmp_dv : obj = fds_impi_intel_win_openmp_dv
impi_intel_win_openmp_dv : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) $(FOPENMPFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL_OPENMP) $(LFLAGS_SUNDIALS_WIN)

impi_intel_win_db : MPILIB = "$(IMPI_DEBUG_ROOT)\impi.lib"
impi_intel_win_db : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win_db : FFLAGS = -D_WIN32 /Od /nologo /debug:all /I$(MPIINCLUDE) /Z7 /extend-source:132 /warn:unused /warn:nointerfaces /Qtrapuv /fp:strict /fp:except /traceback /check:all /stand:f18 /wrap-margin- $(GITINFO) $(FFLAGSMKL_INTEL) $(FFLAGS_SUNDIALS_WIN) /Qdiag-disable:10448
impi_intel_win_db : FCOMPL = $(I_IFORT)
impi_intel_win_db : obj = fds_impi_intel_win_db
impi_intel_win_db : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL) $(LFLAGS_SUNDIALS_WIN)

impi_intel_win_openmp_db : MPILIB = "$(IMPI_DEBUG_ROOT)\impi.lib"
impi_intel_win_openmp_db : MPIINCLUDE = "$(I_MPI_ROOT)\include\mpi"
impi_intel_win_openmp_db : FFLAGS = -D_WIN32 /Od /nologo /debug:all /I$(MPIINCLUDE) /Z7 /extend-source:132 /warn:unused /warn:nointerfaces /Qtrapuv /fp:strict /fp:except /traceback /check:all /stand:f18 /wrap-margin- $(GITINFO) $(FFLAGSMKL_INTEL) $(FFLAGS_SUNDIALS_WIN) /Qdiag-disable:10448
impi_intel_win_openmp_db : FOPENMPFLAGS = /Qopenmp_INTEL
impi_intel_win_openmp_db : FCOMPL = $(I_IFORT)
impi_intel_win_openmp_db : obj = fds_impi_intel_win_openmp_db
impi_intel_win_openmp_db : setup_win $(objwin_mpi)
	$(FCOMPL) -o $(obj) $(FFLAGS) $(FOPENMPFLAGS) /F1000000000 $(objwin_mpi) $(MPILIB) $(LFLAGSMKL_INTEL_OPENMP) $(LFLAGS_SUNDIALS_WIN)

# Linux Intel Fortran Compiler and Intel MPI

impi_intel_linux : FFLAGS = -m64 -fc=$(I_IFORT) -O2 -ipo -no-wrap-margin $(GITINFO) $(FFLAGSMKL_INTEL) $(FFLAGS_SUNDIALS) -DUSE_IFPORT  -diag-disable=10448
impi_intel_linux : LFLAGSMKL = $(LFLAGSMKL_INTEL) $(LFLAGS_SUNDIALS_INTEL)
impi_intel_linux : FCOMPL = mpiifort
impi_intel_linux : obj = fds_impi_intel_linux
impi_intel_linux : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

impi_intel_linux_openmp : FFLAGS = -m64 -fc=$(I_IFORT) -O2 -ipo -no-wrap-margin $(GITINFO) $(FFLAGSMKL_INTEL) $(FFLAGS_SUNDIALS) -DUSE_IFPORT -diag-disable=10448_INTEL
impi_intel_linux_openmp : LFLAGSMKL = $(LFLAGSMKL_INTEL_OPENMP) $(LFLAGS_SUNDIALS_INTEL)
impi_intel_linux_openmp : FCOMPL = mpiifort
impi_intel_linux_openmp : FOPENMPFLAGS = -qopenmp
impi_intel_linux_openmp : obj = fds_impi_intel_linux_openmp
impi_intel_linux_openmp : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

impi_intel_linux_db : FFLAGS = -m64 -fc=$(I_IFORT) -check all -warn all -diag-error=remark,warn,error -O0 -g -traceback -fpe0 -nofltconsistency -stand:f18 -no-wrap-margin $(GITINFO) $(FFLAGSMKL_INTEL) $(FFLAGS_SUNDIALS) -DUSE_IFPORT -diag-disable=10448_INTEL
impi_intel_linux_db : LFLAGSMKL = $(LFLAGSMKL_INTEL) $(LFLAGS_SUNDIALS_INTEL)
impi_intel_linux_db : FCOMPL = mpiifort
impi_intel_linux_db : obj = fds_impi_intel_linux_db
impi_intel_linux_db : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

impi_intel_linux_openmp_db : FFLAGS = -m64 -fc=$(I_IFORT) -check all -warn all -diag-error=remark,warn,error -O0 -g -traceback -fpe0 -nofltconsistency -stand:f18 -no-wrap-margin $(GITINFO) $(FFLAGSMKL_INTEL) $(FFLAGS_SUNDIALS) -DUSE_IFPORT  -diag-disable=10448
impi_intel_linux_openmp_db : LFLAGSMKL = $(LFLAGSMKL_INTEL_OPENMP) $(LFLAGS_SUNDIALS_INTEL)
impi_intel_linux_openmp_db : FCOMPL = mpiifort
impi_intel_linux_openmp_db : FOPENMPFLAGS = -qopenmp
impi_intel_linux_openmp_db : obj = fds_impi_intel_linux_openmp_db
impi_intel_linux_openmp_db : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

impi_intel_linux_dv : FFLAGS = -m64 -fc=$(I_IFORT) -warn unused -O1 -g -traceback -no-wrap-margin $(GITINFO) $(FFLAGSMKL_INTEL) $(FFLAGS_SUNDIALS) -DUSE_IFPORT  -diag-disable=10448
impi_intel_linux_dv : LFLAGSMKL = $(LFLAGSMKL_INTEL) $(LFLAGS_SUNDIALS_INTEL)
impi_intel_linux_dv : FCOMPL = mpiifort
impi_intel_linux_dv : obj = fds_impi_intel_linux_dv
impi_intel_linux_dv : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

impi_intel_linux_openmp_dv : FFLAGS = -m64 -fc=$(I_IFORT) -warn unused -O1 -g -traceback -no-wrap-margin $(GITINFO) $(FFLAGSMKL_INTEL) $(FFLAGS_SUNDIALS) -DUSE_IFPORT  -diag-disable=10448
impi_intel_linux_openmp_dv : LFLAGSMKL = $(LFLAGSMKL_INTEL_OPENMP) $(LFLAGS_SUNDIALS_INTEL)
impi_intel_linux_openmp_dv : FCOMPL = mpiifort
impi_intel_linux_openmp_dv : FOPENMPFLAGS = -qopenmp
impi_intel_linux_openmp_dv : obj = fds_impi_intel_linux_openmp_dv
impi_intel_linux_openmp_dv : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

# OSX Intel Fortran and Open MPI

ompi_intel_osx : FFLAGS = -m64 -O2 -stand:f18 -no-wrap-margin $(GITINFO) $(FFLAGSMKL_CUSTOM)
ompi_intel_osx : LFLAGSMKL = $(LFLAGSMKL_CUSTOM) $(CLT_VERSION)
ompi_intel_osx : LFLAGS = -static-intel
ompi_intel_osx : FCOMPL = mpifort
ompi_intel_osx : obj = fds_ompi_intel_osx
ompi_intel_osx : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(LFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_intel_osx_openmp : FFLAGS = -m64 -O2 -stand:f18 -no-wrap-margin $(GITINFO) $(FFLAGSMKL_CUSTOM)
ompi_intel_osx_openmp : LFLAGSMKL = $(LFLAGSMKL_CUSTOM_OPENMP) $(CLT_VERSION)
ompi_intel_osx_openmp : LFLAGS = -static-intel
ompi_intel_osx_openmp : FOPENMPFLAGS = -qopenmp -qopenmp-link static
ompi_intel_osx_openmp : FCOMPL = mpifort
ompi_intel_osx_openmp : obj = fds_ompi_intel_osx_openmp
ompi_intel_osx_openmp : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(LFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_intel_osx_db : FFLAGS = -m64 -check all -fp-stack-check -warn unused -O0 -auto -WB -g -traceback -fpe0 -nofltconsistency -stand:f18 -no-wrap-margin $(GITINFO) $(FFLAGSMKL_CUSTOM)
ompi_intel_osx_db : LFLAGSMKL = $(LFLAGSMKL_CUSTOM) $(CLT_VERSION)
ompi_intel_osx_db : LFLAGS = -static-intel
ompi_intel_osx_db : FCOMPL = mpifort
ompi_intel_osx_db : obj = fds_ompi_intel_osx_db
ompi_intel_osx_db : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(LFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_intel_osx_openmp_db : FFLAGS = -m64 -check all -fp-stack-check -warn unused -O0 -auto -WB -g -traceback -fpe0 -nofltconsistency -stand:f18 -no-wrap-margin $(GITINFO) $(FFLAGSMKL_CUSTOM)
ompi_intel_osx_openmp_db : LFLAGSMKL = $(LFLAGSMKL_CUSTOM_OPENMP) $(CLT_VERSION)
ompi_intel_osx_openmp_db : LFLAGS = -static-intel
ompi_intel_osx_openmp_db : FOPENMPFLAGS = -qopenmp -qopenmp-link static
ompi_intel_osx_openmp_db : FCOMPL = mpifort
ompi_intel_osx_openmp_db : obj = fds_ompi_intel_osx_openmp_db
ompi_intel_osx_openmp_db : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(LFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_intel_osx_dv : FFLAGS = -m64 -warn unused -O1 -g -traceback -stand:f18 -no-wrap-margin  $(GITINFO) $(FFLAGSMKL_CUSTOM)
ompi_intel_osx_dv : LFLAGSMKL = $(LFLAGSMKL_CUSTOM) $(CLT_VERSION)
ompi_intel_osx_dv : LFLAGS = -static-intel
ompi_intel_osx_dv : FCOMPL = mpifort
ompi_intel_osx_dv : obj = fds_ompi_intel_osx_dv
ompi_intel_osx_dv : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(LFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_intel_osx_openmp_dv : FFLAGS = -m64 -warn unused -O1 -g -traceback -stand:f18 -no-wrap-margin  $(GITINFO) $(FFLAGSMKL_CUSTOM)
ompi_intel_osx_openmp_dv : LFLAGSMKL = $(LFLAGSMKL_CUSTOM_OPENMP) $(CLT_VERSION)
ompi_intel_osx_openmp_dv : LFLAGS = -static-intel
ompi_intel_osx_openmp_dv : FOPENMPFLAGS = -qopenmp -qopenmp-link static
ompi_intel_osx_openmp_dv : FCOMPL = mpifort
ompi_intel_osx_openmp_dv : obj = fds_ompi_intel_osx_openmp_dv
ompi_intel_osx_openmp_dv : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(LFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

#*** GNU Compilers ***

ompi_gnu_linux : FFLAGS = -m64 -O2 -std=f2018 -frecursive -ffpe-summary=none -fall-intrinsics $(GITINFOGNU) $(FFLAGSMKL_GNU_OPENMPI) $(GFORTRAN_OPTIONS)
ompi_gnu_linux : LFLAGSMKL = $(LFLAGSMKL_GNU_OPENMPI)
ompi_gnu_linux : FCOMPL = mpifort
ompi_gnu_linux : FOPENMPFLAGS = -fopenmp
ompi_gnu_linux : obj = fds_ompi_gnu_linux
ompi_gnu_linux : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_gnu_linux_db : FFLAGS = -m64 -O0 -std=f2018 -ggdb -Wall -Werror -Wunused-parameter -Wcharacter-truncation -Wno-target-lifetime -fcheck=all -fbacktrace -ffpe-trap=invalid,zero,overflow -frecursive -ffpe-summary=none -fall-intrinsics -fbounds-check $(GITINFOGNU) $(FFLAGSMKL_GNU_OPENMPI) $(GFORTRAN_OPTIONS)
ompi_gnu_linux_db : LFLAGSMKL = $(LFLAGSMKL_GNU_OPENMPI)
ompi_gnu_linux_db : FCOMPL = mpifort
ompi_gnu_linux_db : FOPENMPFLAGS = -fopenmp
ompi_gnu_linux_db : obj = fds_ompi_gnu_linux_db
ompi_gnu_linux_db : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_gnu_osx : FFLAGS = -m64 -O2 -std=f2018 -frecursive -ffpe-summary=none -fall-intrinsics $(GITINFOGNU) $(FFLAGSMKL_GNU_CUSTOM) $(GFORTRAN_OPTIONS) $(FFLAGS_SUNDIALS)
ompi_gnu_osx : LFLAGSMKL = $(LFLAGSMKL_GNU_CUSTOM) $(CLT_VERSION) $(LFLAGS_SUNDIALS_OSX)
ompi_gnu_osx : FCOMPL = mpifort
ompi_gnu_osx : FOPENMPFLAGS = -fopenmp
ompi_gnu_osx : obj = fds_ompi_gnu_osx
ompi_gnu_osx : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_gnu_osx_db : FFLAGS = -m64 -O0 -std=f2018 -ggdb -Wall -Wcharacter-truncation -Wno-target-lifetime -fcheck=all -fbacktrace -ffpe-trap=invalid,zero,overflow -frecursive -ffpe-summary=none -fall-intrinsics $(GITINFOGNU) $(FFLAGSMKL_GNU_CUSTOM) $(GFORTRAN_OPTIONS) $(FFLAGS_SUNDIALS)
ompi_gnu_osx_db : LFLAGSMKL = $(LFLAGSMKL_GNU_CUSTOM) $(CLT_VERSION) $(LFLAGS_SUNDIALS_OSX)
ompi_gnu_osx_db : FCOMPL = mpifort
ompi_gnu_osx_db : FOPENMPFLAGS = -fopenmp
ompi_gnu_osx_db : obj = fds_ompi_gnu_osx_db
ompi_gnu_osx_db : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

ompi_gnu_osx_dv : FFLAGS = -m64 -O1 -fbacktrace -std=f2018 -frecursive -ffpe-summary=none -fall-intrinsics $(GITINFOGNU) $(FFLAGSMKL_GNU_CUSTOM) $(GFORTRAN_OPTIONS) $(FFLAGS_SUNDIALS)
ompi_gnu_osx_dv : LFLAGSMKL = $(LFLAGSMKL_GNU_CUSTOM) $(CLT_VERSION) $(LFLAGS_SUNDIALS_OSX)
ompi_gnu_osx_dv : FCOMPL = mpifort
ompi_gnu_osx_dv : FOPENMPFLAGS = -fopenmp
ompi_gnu_osx_dv : obj = fds_ompi_gnu_osx_dv
ompi_gnu_osx_dv : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)

mpich_gnu_polaris : FFLAGS = -m64 -O2 -fbacktrace  -std=f2018 -frecursive -ffpe-summary=none -fall-intrinsics $(GITINFOGNU) $(GNU_COMPINFO) $(FFLAGSMKL_GNU_OPENMPI) $(GFORTRAN_OPTIONS) $(FFLAGS_SUNDIALS)
#mpich_gnu_polaris : FFLAGS = -m64 -O0 -std=f2018 -ggdb -Wall -Wcharacter-truncation -Wno-target-lifetime -fcheck=all -fbacktrace -ffpe-trap=invalid,zero,overflow -frecursive -ffpe-summary=none -fall-intrinsics $(GITINFOGNU) $(GNU_COMPINFO) $(FFLAGSMKL_GNU_OPENMPI) $(GFORTRAN_OPTIONS) $(FFLAGS_SUNDIALS)
mpich_gnu_polaris : LFLAGSMKL = $(LFLAGSMKL_GNU_OPENMPI) $(LFLAGS_SUNDIALS)
mpich_gnu_polaris : FCOMPL = ftn
mpich_gnu_polaris : FOPENMPFLAGS = -fopenmp
mpich_gnu_polaris : obj = fds_mpich_gnu_polaris
mpich_gnu_polaris : setup $(obj_mpi)
	$(FCOMPL) $(FFLAGS) $(FOPENMPFLAGS) -o $(obj) $(obj_mpi) $(LFLAGSMKL)


# *** Object Dependencies ***

prec.o :
imkl.o :
cons.o : prec.o
chem.o : cons.o func.o
prop.o : prec.o
devc.o : prec.o
type.o : prec.o cons.o imkl.o
data.o : prec.o func.o cons.o prop.o
rcal.o : func.o prec.o cons.o
mesh.o : prec.o type.o
func.o : prec.o cons.o type.o mesh.o devc.o
gsmv.o : prec.o func.o cons.o mesh.o
smvv.o : geom.o prec.o func.o cons.o data.o
turb.o : prec.o func.o cons.o mesh.o
soot.o : prec.o mesh.o cons.o func.o turb.o
pois.o : prec.o cons.o
geom.o : gsmv.o func.o prec.o cons.o type.o mesh.o turb.o imkl.o data.o
ccib.o : gsmv.o func.o prec.o cons.o type.o mesh.o turb.o geom.o
radi.o : func.o prec.o cons.o type.o mesh.o rcal.o geom.o
part.o : func.o prec.o cons.o type.o mesh.o devc.o data.o geom.o soot.o ccib.o
vege.o : func.o prec.o cons.o type.o mesh.o geom.o
ctrl.o : prec.o cons.o type.o mesh.o func.o
hvac.o : func.o prec.o cons.o type.o mesh.o ctrl.o data.o geom.o
mass.o : func.o prec.o cons.o type.o mesh.o turb.o soot.o geom.o ccib.o
wall.o : func.o prec.o cons.o type.o mesh.o hvac.o mass.o soot.o geom.o ccib.o
fire.o : func.o prec.o cons.o type.o mesh.o mass.o soot.o radi.o chem.o
velo.o : func.o prec.o cons.o type.o mesh.o turb.o geom.o ccib.o
pres.o : func.o prec.o cons.o type.o mesh.o pois.o velo.o geom.o ccib.o imkl.o
init.o : func.o prec.o cons.o type.o mesh.o devc.o radi.o pois.o data.o pres.o geom.o
dump.o : func.o prec.o cons.o type.o mesh.o devc.o smvv.o turb.o data.o radi.o soot.o geom.o ccib.o fire.o
read.o : func.o prec.o cons.o type.o mesh.o devc.o ctrl.o radi.o hvac.o data.o geom.o mass.o soot.o prop.o
divg.o : func.o prec.o cons.o type.o mesh.o mass.o ccib.o
main.o : func.o prec.o cons.o type.o mesh.o devc.o smvv.o mass.o divg.o velo.o wall.o fire.o radi.o part.o vege.o dump.o read.o init.o pres.o pois.o ctrl.o turb.o hvac.o data.o geom.o ccib.o soot.o prop.o chem.o

prec.obj :
imkl.obj :
cons.obj : prec.obj
chem.obj : cons.obj func.obj
prop.obj : prec.obj
devc.obj : prec.obj
type.obj : prec.obj cons.obj imkl.obj
data.obj : prec.obj func.obj cons.obj prop.obj
rcal.obj : func.obj prec.obj cons.obj
mesh.obj : prec.obj type.obj
func.obj : prec.obj cons.obj type.obj mesh.obj devc.obj
gsmv.obj : prec.obj func.obj cons.obj mesh.obj
smvv.obj : geom.obj prec.obj func.obj cons.obj data.obj
turb.obj : prec.obj func.obj cons.obj mesh.obj
soot.obj : prec.obj mesh.obj cons.obj func.obj turb.obj
pois.obj : prec.obj cons.obj
geom.obj : gsmv.obj func.obj prec.obj cons.obj type.obj mesh.obj turb.obj imkl.obj data.obj
ccib.obj : gsmv.obj func.obj prec.obj cons.obj type.obj mesh.obj turb.obj geom.obj
radi.obj : func.obj prec.obj cons.obj type.obj mesh.obj rcal.obj geom.obj
part.obj : func.obj prec.obj cons.obj type.obj mesh.obj devc.obj data.obj geom.obj soot.obj ccib.obj
vege.obj : func.obj prec.obj cons.obj type.obj mesh.obj geom.obj
ctrl.obj : prec.obj cons.obj type.obj mesh.obj func.obj
hvac.obj : func.obj prec.obj cons.obj type.obj mesh.obj ctrl.obj data.obj geom.obj
mass.obj : func.obj prec.obj cons.obj type.obj mesh.obj turb.obj soot.obj geom.obj ccib.obj
wall.obj : func.obj prec.obj cons.obj type.obj mesh.obj hvac.obj mass.obj soot.obj geom.obj ccib.obj
fire.obj : func.obj prec.obj cons.obj type.obj mesh.obj mass.obj soot.obj radi.obj chem.obj
velo.obj : func.obj prec.obj cons.obj type.obj mesh.obj turb.obj geom.obj ccib.obj
pres.obj : func.obj prec.obj cons.obj type.obj mesh.obj pois.obj velo.obj geom.obj ccib.obj imkl.obj
init.obj : func.obj prec.obj cons.obj type.obj mesh.obj devc.obj pois.obj data.obj pres.obj geom.obj
dump.obj : func.obj prec.obj cons.obj type.obj mesh.obj devc.obj smvv.obj turb.obj data.obj radi.obj soot.obj geom.obj ccib.obj fire.obj
read.obj : func.obj prec.obj cons.obj type.obj mesh.obj devc.obj ctrl.obj radi.obj hvac.obj data.obj geom.obj mass.obj soot.obj prop.obj
divg.obj : func.obj prec.obj cons.obj type.obj mesh.obj mass.obj ccib.obj
main.obj : func.obj prec.obj cons.obj type.obj mesh.obj devc.obj smvv.obj mass.obj divg.obj velo.obj wall.obj fire.obj radi.obj part.obj vege.obj dump.obj read.obj init.obj pres.obj pois.obj ctrl.obj turb.obj hvac.obj data.obj geom.obj ccib.obj soot.obj prop.obj chem.obj

#*** Clean Target to remove Object and Module files ***

.PHONY : clean
clean:
	-rm -f *.o *.mod *.obj

.PHONY : winclean
winclean:
	-erase *.mod *.obj
