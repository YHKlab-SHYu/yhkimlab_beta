building siesta 4.1.5 with parallel and netcdf support 
====================================

연구실에서 가장 많이 사용하는 코드인 siesta의 가장 최신 release인 4.1.5버전을 netcdf와 parallel을 지원하도록 해서 빌드해보려고 한다.  
arch.make파일에 lapack, scalapack등 라이브러리를 전부 연결시켜야 잘 빌드가되기 때문에 빌드가 상당히 어렵다.  
이 튜토리얼은 siesta 빌드를 사용자가 쉽게 할 수 있도록 하는데 그 목적이 있다.

## 빌드 전에 알아두어야할 점

* 컴파일러를 지정하지 않고 빌드하면 gcc로 빌드가 된다. gcc는 병렬화를 지원하지 않는다. 따라서 병렬화를 지원하는 컴파일러인 mpiicc, mpiifort로 지정해주는 스크립트를 빌드시에 사용한다.
* siesta를 빌드할 때 netcdf를 빌드한 컴파일러와 일치하지 않으면 컴파일러가 일치하지 않는다는 에러가 뜨면서 빌드되지 않는다. 따라서 컴파일러 지정을 잘 해주어야한다.

* 사용하는 소프트웨어 버전 : zlib-1.2.7, hdf5-1.8.12, netcdf-c-4.8.0, netcdf-fortran-4.5.3
* 빌드 순서 : zlib-1.2.7 -> hdf5-1.8.12 ->  netcdf-c-4.8.0 -> netcdf-fortran-4.5.3 -> siesta-v4.1.5
* netcdf에 필요한 패키지들 중 버전을 다르게 하고 싶다면 ftp://ftp.unidata.ucar.edu/pub/netcdf를 internet explorer로 접속해 찾아보면 된다. (크롬은 ftp 지원 X)

1) zlib 설치

1. <u> wget ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-4/zlib-1.2.7.tar.gz </u> 명령어로 zlib를 다운로드
2. Build_script라는 환경변수 지정파일 하나 만든다.(이름 상관없음)

```bash
    export CC=mpiicc
    export CXX=mpiicpc
    export FC=mpiifort
    export F77=mpif77
    export F90=mpif90
    export FFLAGS='-fPIC -O3 -m64'
    export CFLAGS='-fPIC -O3 -m64'
    export MPICC=mpiicc
    export MPICXX=mpiicpc
    export MPIF77=mpif77
    export MPIF90=mpif90
    export LDFLAGS="-L/opt/intel/MKL/2018.5.274/lib -L/opt/intel/MKL/2018.5.274/lib/intel64/ -L/opt/intel/MPI/2018.4.274/impi/2018.4.274/intel64/lib"
    export LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/opt/intel/MKL/2018.5.274/lib:/opt/intel/MKL/2018.5.274/lib/intel64/:/opt/intel/MPI/2018.4.274/impi/2018.4.274/intel64/lib"
    export CPPFLAGS="-I$/opt/intel/MKL/2018.5.274/include -I$/opt/intel/MKL/2018.5.274/include/intel64/lp64 -I/opt/intel/MPI/2018.4.274/impi/2018.4.274/intel64/include"
    export LIBS="-ldl"

```

3. <u> source build_script </u> 명령어로 환경변수 업데이트
4. <u> ./configure --prefix=/설/치/경/로/</u> 명령어로 설치 준비
5. <u> make check install</u>로 설치

2) hdf5 설치

1. <u>wget ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-4/hdf5-1.8.12.tar.gz </u> 명령어로 hdf5 다운로드
2. Build_script을 다음과 같이 생성(zlib 라이브러리 추가)

```bash
export CC=mpiicc
export CXX=mpiicpc
export FC=mpiifort
export F77=mpif77
export F90=mpif90
export FFLAGS='-fPIC -O3 -m64'
export CFLAGS='-fPIC -O3 -m64'
export MPICC=mpiicc
export MPICXX=mpiicpc
export MPIF77=mpif77
export MPIF90=mpif90
export LDFLAGS="-L/opt/intel/MKL/2018.5.274/lib -L/opt/intel/MKL/2018.5.274/lib/intel64/ -L/opt/intel/MPI/2018.4.274/impi/2018.4.274/intel64/lib -L/zlib/설/치/된/경/로"
export LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/opt/intel/MKL/2018.5.274/lib:/opt/intel/MKL/2018.5.274/lib/intel64/:/opt/intel/MPI/2018.4.274/impi/2018.4.274/intel64/lib:/zlib/설/치/된/경/로"
export CPPFLAGS="-I$/opt/intel/MKL/2018.5.274/include -I$/opt/intel/MKL/2018.5.274/include/intel64/lp64 -I/opt/intel/MPI/2018.4.274/impi/2018.4.274/intel64/include -I/zlib/설/치/된/경/로"
export LIBS="-ldl"
```

3. <u> source build_script </u> 명령어로 환경변수 업데이트
4. <u>./configure --with-zlib=/설/치/된/경/로/zlib --prefix=/설/치/경/로/ --enable-parallel --enable-fortran --enable-shared</u> 명령어로 설치 준비
    * parallel 옵션을 넣어야 netcdf도 parallel 버전으로 설치된다.
5. <u> make check install</u> 로 설치

3) netcdf-c 설치

1. <u>wget ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-c-4.8.0.tar.gz</u> 명령어로 netcdf-c 다운로드
2. Build_script을 다음과 같이 생성(zlib, hdf5 라이브러리 추가)

```bash
export CC=mpiicc
export CXX=mpiicpc
export FC=mpiifort
export F77=mpif77
export F90=mpif90
export FFLAGS='-fPIC -O3 -m64'
export CFLAGS='-fPIC -O3 -m64'
export MPICC=mpiicc
export MPICXX=mpiicpc
export MPIF77=mpif77
export MPIF90=mpif90
export LDFLAGS="-L/opt/intel/MKL/2018.5.274/lib -L/opt/intel/MKL/2018.5.274/lib/intel64/ -L/opt/intel/MPI/2018.4.274/impi/2018.4.274/intel64/lib -L/zlib/설/치/된/경/로 -L/hdf5/설/치/된/경/로"
export LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/opt/intel/MKL/2018.5.274/lib:/opt/intel/MKL/2018.5.274/lib/intel64/:/opt/intel/MPI/2018.4.274/impi/2018.4.274/intel64/lib:/zlib/설/치/된/경/로:/hdf5/설/치/된/경/로"
export CPPFLAGS="-I$/opt/intel/MKL/2018.5.274/include -I$/opt/intel/MKL/2018.5.274/include/intel64/lp64 -I/opt/intel/MPI/2018.4.274/impi/2018.4.274/intel64/include -I/zlib/설/치/된/경/로 -I/hdf5/설/치/된/경/로"
export LIBS="-ldl"

```

3. <u> source build_script </u> 명령어로 환경변수 업데이트
4. <u>./configure --prefix=/설/치/경/로/</u> 명령어로 설치 준비
5. <u> make check install</u> 로 설치

4) netcdf-fortran 설치

1. <u>wget	ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-fortran-4.5.3.tar.gz</u> 명령어로 netcdf-fortran 다운로드
2. Build_script을 다음과 같이 생성(zlib, hdf5, netcdf-c 라이브러리 추가)

```bash
export CC=mpiicc
export CXX=mpiicpc
export FC=mpiifort
export F77=mpif77
export F90=mpif90
export FFLAGS='-fPIC -O3 -m64'
export CFLAGS='-fPIC -O3 -m64'
export MPICC=mpiicc
export MPICXX=mpiicpc
export MPIF77=mpif77
export MPIF90=mpif90
export LDFLAGS="-L/opt/intel/MKL/2018.5.274/lib -L/opt/intel/MKL/2018.5.274/lib/intel64/ -L/opt/intel/MPI/2018.4.274/impi/2018.4.274/intel64/lib -L/zlib/설/치/된/경/로 -L/hdf5/설/치/된/경/로 -L/netcdf/설/치/된/경/로"
export LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/opt/intel/MKL/2018.5.274/lib:/opt/intel/MKL/2018.5.274/lib/intel64/:/opt/intel/MPI/2018.4.274/impi/2018.4.274/intel64/lib:/home2/littleyu/.local/zlib/lib:/zlib/설/치/된/경/로:/hdf5/설/치/된/경/로:/netcdf/설/치/된/경/로"
export CPPFLAGS="-I$/opt/intel/MKL/2018.5.274/include -I$/opt/intel/MKL/2018.5.274/include/intel64/lp64 -I/opt/intel/MPI/2018.4.274/impi/2018.4.274/intel64/include -I/zlib/설/치/된/경/로 -I/hdf5/설/치/된/경/로 -I/netcdf/설/치/된/경/로"
export LIBS="-ldl -lnetcdf -lhdf5_hl -lhdf5 -lz"
```

3. <u> source build_script </u> 명령어로 환경변수 업데이트
4. <u>./configure --prefix=/netcdf/설/치/된/경/로/</u> 명령어로 설치 준비
5. <u> make install</u> 로 설치(check 옵션 사용시 에러)

5) siesta 빌드
1. Obj 폴더에서 sh ../Src/obj_setup.sh 명령어로 파일 복사
2. arch.make 파일을 다음과 같이 생성

```bash
# 
# Copyright (C) 1996-2016	The SIESTA group
#  This file is distributed under the terms of the
#  GNU General Public License: see COPYING in the top directory
#  or http://www.gnu.org/copyleft/gpl.txt.
# See Docs/Contributors.txt for a list of contributors.
#
#-------------------------------------------------------------------
# arch.make file for gfortran compiler.
# To use this arch.make file you should rename it to
#   arch.make
# or make a sym-link.
# For an explanation of the flags see DOCUMENTED-TEMPLATE.make

.SUFFIXES:
.SUFFIXES: .f .F .o .c .a .f90 .F90

SIESTA_ARCH = unknown

CC = mpiicc
FPP = $(FC) -E -P
FC = mpiifort
FC_SERIAL = ifort

FFLAGS = -O2 -fPIC -xSSE4.2 -fp-model source

MKLROOT=/opt/intel/MKL/2018.5.274
MPIROOT=/opt/intel/MPI/2018.4.274/impi/2018.4.274/intel64
MKL_LIB=/opt/intel/MKL/2018.5.274/lib/intel64

AR = /usr/bin/ar
RANLIB = ranlib

SYS = nag

SP_KIND = 4
DP_KIND = 8
KINDS = $(SP_KIND) $(DP_KIND)

INCFLAGS=-I$(MKLROOT)/include -I$(MKLROOT)/include/intel64/lp64 -I$(MPIROOT)/include
LDFLAGS=-I$(MKLROOT)/lib -I$(MKLROOT)/lib/intel64/ -I$(MPIROOT)/lib

BLAS_LIBS= -Wl,--start-group $(MKL_LIB)/libmkl_blas95_lp64.a $(MKL_LIB)/libmkl_intel_lp64.a $(MKL_LIB)/libmkl_sequential.a $(MKL_LIB)/libmkl_core.a -Wl,--end-group
LAPACK_LIBS= -Wl,--start-group $(MKL_LIB)/libmkl_lapack95_lp64.a $(MKL_LIB)/libmkl_blacs_intelmpi_lp64.a $(MKL_LIB)/libmkl_intel_lp64.a $(MKL_LIB)/libmkl_sequential.a $(MKL_LIB)/libmkl_core.a -Wl,--end-group
BLACS_LIBS= -Wl,--start-group $(MKL_LIB)/libmkl_blacs_intelmpi_lp64.a $(MKL_LIB)/libmkl_intel_lp64.a $(MKL_LIB)/libmkl_sequential.a $(MKL_LIB)/libmkl_core.a -Wl,--end-group
SCALAPACK_LIBS= -Wl,--start-group $(MKL_LIB)/libmkl_scalapack_lp64.a $(MKL_LIB)/libmkl_intel_lp64.a $(MKL_LIB)/libmkl_core.a $(MKL_LIB)/libmkl_sequential.a  -Wl,--end-group -lpthread
#COMP_LIBS = libsiestaLAPACK.a libsiestaBLAS.a

FPPFLAGS = $(DEFS_PREFIX)-DFC_HAVE_ABORT

MPI_INTERFACE=libmpi_f90.a
MPI_INCLUDE=.

FPPFLAGS+=-DMPI

LIBS = $(SCALAPACK_LIBS) $(BLACS_LIBS) $(LAPACK_LIBS) $(BLAS_LIBS) 

# Dependency rules ---------

FFLAGS_DEBUG = -g -O1 -fp-model source   # your appropriate flags here...

# The atom.f code is very vulnerable. Particularly the Intel compiler
# will make an erroneous compilation of atom.f with high optimization
# levels.
atom.o: atom.F
	$(FC) -c $(FFLAGS_DEBUG) $(INCFLAGS) $(FPPFLAGS) $(FPPFLAGS_fixed_F) $< 
state_analysis.o: state_analysis.F
	$(FC) -c $(FFLAGS_DEBUG) $(INCFLAGS) $(FPPFLAGS) $(FPPFLAGS_fixed_F) $< 

.c.o:
	$(CC) -c $(CFLAGS) $(INCFLAGS) $(CPPFLAGS) $< 
.F.o:
	$(FC) -c $(FFLAGS) $(INCFLAGS) $(FPPFLAGS) $(FPPFLAGS_fixed_F)  $< 
.F90.o:
	$(FC) -c $(FFLAGS) $(INCFLAGS) $(FPPFLAGS) $(FPPFLAGS_free_F90) $< 
.f.o:
	$(FC) -c $(FFLAGS) $(INCFLAGS) $(FCFLAGS_fixed_f)  $<
.f90.o:
	$(FC) -c $(FFLAGS) $(INCFLAGS) $(FCFLAGS_free_f90)  $<

#use for netCDF(netcdf, zlib, hdf5를 자신이 설치한 경로에 맞게 편집)
NETCDF_ROOT = /home2/littleyu/.local/netcdf/
INCFLAGS += -I/home2/littleyu/.local/netcdf/include -I/home2/littleyu/.local/hdf5/include
LDFLAGS += -L/home2/littleyu/.local/zlib/lib -Wl,-rpath,/home2/littleyu/.local/zlib/lib
LDFLAGS += -L/home2/littleyu/.local/hdf5/lib -Wl,-rpath,/home2/littleyu/.local/hdf5/lib
LDFLAGS += -L/home2/littleyu/.local/netcdf/lib -Wl,-rpath,/home2/littleyu/.local/netcdf/lib
LIBS += -L$(NETCDF_ROOT)/lib -lnetcdff -lnetcdf -lhdf5_hl -lhdf5 -lz
COMP_LIBS += libncdf.a libfdict.a
FPPFLAGS += -DCDF -DNCDF -DNCDF_4
```

3. <u>make</u> 명령어로 siesta 빌드
4. <u>cd ../Util</u> 명령어로 Util 폴더로 이동
5. <u>sh build_all.sh</u>명령어로 유틸 전부 빌드

이렇게 전부 따라오게 되면 ./Grid, ./Gen-basis, ./DensityMatrix, ./WFS, ./ON 5개 외에 다른 유틸들을 전부 빌드 가능하게 된다.

참고자료

1. siesta manual  
2. netcdf INSTALL.md  
3. 국가 슈퍼 컴퓨팅센터 https://www.ksc.re.kr/cmnt/jrs/sft/view?jrskey=76#a  

작성자 : 유승현