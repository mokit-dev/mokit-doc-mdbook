# 2.3 Linux: Build from Source

The latest version of MOKIT source code can be downloaded via [mokit-master.zip](https://gitlab.com/jxzou/mokit/-/archive/master/mokit-master.zip) or [mokit-master.tar.gz](https://gitlab.com/jxzou/mokit/-/archive/master/mokit-master.tar.gz).


## 2.3.1 Prerequisite
1. Fortran compiler: `ifort`(>=2017) or `gfortran`(>=4.8.5)
2. Intel MKL(recommended) or [OpenBLAS](https://github.com/xianyi/OpenBLAS)
3. `f2py`: Anaconda Python3(recommended) or Miniconda + Numpy

It is recommended to install the Intel compiler and Anaconda Python3 on your computer/node. Although these two packages may be large, they meet all prerequisites of compiling MOKIT. Note that the Intel compiler is free of charge for academic use. You can download Anaconda Python3 from the [NJU mirror website](http://mirrors.nju.edu.cn/anaconda/archive), e.g the package `Anaconda3-2024.02-1-Linux-x86_64.sh`.
<!--
Currently python >= 3.12 is not supported due to the new f2py backend has not been fully supported yet. Please use python <= 3.11 (create such an environment with network, or download an old Anaconda3 (2024.02 and earlier)).
-->

If you do not have the `ifort` compiler or Intel MKL on your computer, you may want to download and install them. There are several versions recommended, you can choose any one of them:  
(1) Intel Parallel Studio XE 2017~2020 are all OK (2019 or 2020 preferred).  
(2) Since 2021, there is no Parallel Studio XE, but only the Intel OneAPI. You should download and install both HPC Toolkit and MKL if you want to use 2021 version.

If you want to use other compilers, please read [Section 2.3.6](#236-compiler-notes).


## 2.3.2 Using make to compile
Assuming you have already downloaded the MOKIT .zip package, just run
```
unzip mokit-master.zip
```
to uncompress it. Note that DO NOT unzip the package in Windows OS, but unzip it in Linux OS. Then enter the source code directory
```
mv mokit-master mokit
cd mokit/src
```

and run
```
make all
```
to compile MOKIT. This will take about 2 minutes. There is no `make install` step. If you are using Intel 2025 or higher, where there is `ifx` but no `ifort`, you can run
```
make all -f Makefile.intel_ifx
```
instead.

If you do not need `automr` for automatic multireference calculations and only want to compile one or several modules, e.g. `fch2inp` (for Gaussian -> GAMESS orbital transferring), then you simply need to run
```
make fch2inp
```
Be careful with hints on the screen, some modules depend on other modules, thus a compilation of two or three modules is necessary sometimes.


## 2.3.3 Using fpm to compile
If you prefer to use `fpm` rather than `make`, you are supposed to run

```
unzip mokit-master.zip
mv mokit-master mokit
cd mokit/
fpm build --compiler=ifort --flag="-O2 -fpp -fPIC -qopenmp"
fpm install --compiler=ifort --flag="-O2 -fpp -fPIC -qopenmp" --prefix .
```

This will use ifort+MKL to compile all binary executables of MOKIT, i.e. no Python dynamic libraries will be compiled by `fpm`. In such case you can use MOKIT utilities to transfer MOs among quantum chemistry packages. If you also want Python dynamic libraries, you should enter `mokit/src/` and run `make pymodules`.

If you prefer gfortran+MKL, you are supposed to run
```
fpm build --flag="-O2 -cpp -fPIC -fopenmp"
fpm install --flag="-O2 -cpp -fPIC -fopenmp" --prefix .
```

If you prefer gfortran+OpenBLAS, you are supposed to run
```
mv fpm.toml tpm_mkl.toml
mv fpm_openblas.toml fpm.toml
fpm build --flag="-O2 -cpp -fPIC -fopenmp" --link-flag="-L$HOME/software/openblas-0.3.29/lib64 -lopenblas"
fpm install --flag="-O2 -cpp -fPIC -fopenmp" --link-flag="-L$HOME/software/openblas-0.3.29/lib64 -lopenblas" --prefix .
```
Here the OpenBLAS is assumed to be already installed in `$HOME/software/openblas-0.3.29`.

One can use [fpm](https://github.com/fortran-lang/fpm) to compile the latest MOKIT source code under Windows OS, e.g. in msys2.


## 2.3.4 Environment variables

After successful compilation, you need to add the following environment variables into your `~/.bashrc` file:
```
export MOKIT_ROOT=$HOME/software/mokit
export PATH=$MOKIT_ROOT/bin:$PATH
export PYTHONPATH=$MOKIT_ROOT:$PYTHONPATH
export GMS=$HOME/software/gamess/rungms       # optional
export PSI4=$HOME/psi4conda/bin/psi4          # optional
export BDF=$HOME/software/bdf-pkg/sbin/run.sh # optional
```

Please modify the above paths to suit your situations. '# optional' means it is not obligatory and it depends on your usage. Since PySCF is run by `python`, OpenMolcas is run by `pymolcas`, Molpro is run by `molpro`, PSI4 is run by `psi4` and Dalton is run by `dalton`, there is no extra environment variable to be exported here. But if you define `export PSI4=...`, then this variable has priority to the path found by `which psi4`. 

For Dalton, you can install either MKL or MPI version since both are supported, and it would be automatically detected by MOKIT. The environment variable `BDF` are optional, there is no need to write it if you do not want to use BDF.

The configuration file `program.info` is no longer used since MOKIT 1.2.1. After writing environment variables, a logout and re-login on your terminal is strongly recommended. 

**Setup GAMESS**

Note that the original GAMESS source code can only deal with GVB up to 12 pairs. To go beyond that (which is routine type of calculation in `automr` of MOKIT), please read [GVB_prog](./chap4-4.md#4410-gvb_prog) carefully. After re-compiling GAMESS (followed by instructions in Section 4.4.10), an executable `gamess.01.x` will be generated. This is the executable to be called by `automr`.

**Setup scratch directory**

The scratch directory (in Chinese, 临时文件存放目录) is not determined by MOKIT or `automr`, but by each quantum chemistry package (Gaussian, OpenMolcas, GAMESS, etc). Please do not submit two computations with the same input filename (even in different work directories) at the same time, since their temporary files may be overwritten by each other. For example, GAMESS, OpenMolcas and Molpro jobs are sensitive to the files in the scratch directory, while Gaussian, PySCF and ORCA are not that sensitive.


## 2.3.5 Test
To see the version and help information, you can run
```
automr -v (or -V, --version)
automr -h (or -help, --help)
```
To test whether the automr program works, please change into the example directory, and pick up a .gjf file. For example,
```
cd $MOKIT_ROOT/examples/automr/
automr 00-h2o_cc-pVDZ_1.5.gjf >00-h2o_cc-pVDZ_1.5.out 2>&1 &
```

If you encounter any errors in output, please search your problem in [Section A1 FAQ](./chap_appdx.md#a1-frequently-asked-questions-faq) in Appendix firstly. Some common questions have been listed. For bug reporting, please read [Section A3](./chap_appdx.md#a3-bug-report).

It is strongly recommended to create a new directory and put in the input file. Because during computation many files will be generated by `automr` and those common software packages.


## 2.3.6 Compiler notes
If you want to use any Fortran compiler other than `ifort` (e.g. `gfortran`), you need to install the MKL library or OpenBLAS on your node. The recommended version is `gfortran>=4.8.5`. Note that `gfortran<=4.7` is outdated and thus not recommended. Even if you can successfully compile all utilities using `gfortran<=4.7`, they may not work normally or correctly.

### Option 1: gfortran + Intel MKL
```
make all -f Makefile.gnu_mkl
```

### Option 2: gfortran + OpenBLAS
```
make all -f Makefile.gnu_openblas
```

### Option 3: gfortran + OpenBLAS for Arch/Manjaro distributions
```
make all -f Makefile.gnu_openblas_arch
```

If you had compiled MOKIT before, and assuming now you want to use another compiler, REMEMBER to run `make distclean` before re-compiling. Note that if you change the version of Python on your node/machine, the dynamic library files `*.so` in `$MOKIT_ROOT/mokit/lib/` may become unrecognized, in which case you have to re-compile MOKIT.

If you want to use **Miniconda** instead of **Anaconda Python3**, you should install `numpy` since `numpy` is not in Miniconda by default. `f2py` will be installed along with `numpy`. If your `f2py` comes from `psi4conda/bin/f2py`, then errors may occur when compiling MOKIT. In such case you are recommended to comment environment variables of PSI4 and use your previous python, e.g. Anaconda Python 3. MOKIT can still call PSI4 even if PSI4 environment variables commented (see Section 2.2.3). If you encounter compilation errors when using Intel 2017 and Anaconda Python 3.8, you can try to use gfortran+MKL instead of Intel compiler by running `make all -f Makefile.gnu_mkl`.

For trouble shooting during compiling, please read [Section A1](./chap_appdx.md#a1-frequently-asked-questions-faq).


## 2.3.7 Notes on Quantum chemistry packages
The `automr` program in MOKIT is an integrated interface program connecting common quantum chemistry software packages. The `automr` itself does not contain any code for computing electron integrals currently, it just transfers MOs among software, sometimes read one-electron integrals from some package, and call various packages to do specified computations. Therefore, the users are assumed to have successfully installed some of these common software packages, which are [Gaussian](https://gaussian.com), [PySCF](https://github.com/pyscf/pyscf), [GAMESS](https://www.msg.chem.iastate.edu), [OpenMolcas](https://gitlab.com/Molcas/OpenMolcas), [Molpro](https://www.molpro.net), [ORCA](https://orcaforum.kofo.mpg.de), [BDF](http://182.92.69.169:7226/Introduction), [PSI4](https://github.com/psi4/psi4), [Dalton](https://gitlab.com/dalton/dalton) and [QChem](https://www.q-chem.com). Some recommended versions are shown below

| Program | Recommended Version |
| --- | --- |
| BDF | >= 0.9.8 |
| CFOUR | >= v2.1 |
| Dalton | >= 2020.0 |
| GAMESS | >= 2017 (>= 2019 preferred) |
| Gaussian | >= g09 (>=g16 preferred) |
| Molpro | >= 2015 (>= 2019 preferred) |
| OpenMolcas | >= v21.02 |
| ORCA | >= 4.2.1 (>= 5.0 preferred) |
| PSI4 | >= 1.3.2 (>= 1.4 preferred) |
| PySCF | >= 1.7.4 |
| QChem | >= 5.0 |

For DMRG related packages:

| Program | Recommended Version |
| --- | --- |
|(py)block2 | >= preview-0.5.0 |
| Block | >= 1.5.3 |
| QCMaquis | >= 3.0.3 |
| CheMPS2 | >= 1.8.9 |

The (py)block2 the most recommended one among these DMRG packages.

For MC-PDFT related packages:  
pyscf-forge latest

Older versions are not recommended, since (1) they are possibly not tested by the developers, (2) they had been tested but some functionality had not been (correctly) implemented at that time. So that they may work or may not. OpenMolcas-v18.09 has also been tested, but you need to take care of a [problem](./chap_appdx.md#q5-openmolcas-error-in-keyword).

Of course, not all packages will be called in an `automr` job. It depends on the job type and the program specified by users (see [Section 2.5](./chap2-5.md) for details). Usually three software packages (Gaussian, PySCF and GAMESS) are extensively used in routine computations. Thus you are recommended to install at least these 3 packages. 

### Installation tips and instructions

If you have any difficulty in installing software, please read their corresponding manuals carefully. Or you can find answers from their official websites, forums, and GitHub/GitLab pages:

* [Gaussian](http://gaussian.com/help)  
* [PySCF issues](https://github.com/pyscf/pyscf/issues)  
* [ORCA forum](https://orcaforum.kofo.mpg.de)  
* [Molcas forum](https://molcasforum.univie.ac.at), [OpenMolcas issues](https://gitlab.com/Molcas/OpenMolcas/-/issues)  
* [Molpro group](https://groups.google.com/d/forum/molpro-user)  
* [GAMESS issues](https://github.com/gms-bbg/gamess-issues/issues)  
* [PSI4](http://forum.psicode.org)  
* [Dalton](https://gitlab.com/dalton/dalton)

If you can read Chinese, the following installation instructions or tutorials of common software packages on the WeChat Official Accounts 'quantumchemistry'(微信公众号"量子化学") are strongly recommended:

[Linux下安装Intel oneAPI](https://mp.weixin.qq.com/s/7pQETkrDO1C83vQjKQqI4w)  
[Linux下Gaussian 16安装教程](https://mp.weixin.qq.com/s/ffGo6eOEacfgqg3sYbrLJA)  
[ORCA 5.0安装及运行](https://mp.weixin.qq.com/s/yeCOMhothZeL-V7veAcbuw)  
[GAMESS简易编译教程](https://mp.weixin.qq.com/s/SF5BEfKsGwdKSlZdAe1t4A)  
[PSI4程序安装及运行](https://mp.weixin.qq.com/s/I7Q1YXX5oSsXDe3oMo2jPw)  
[离线编译OpenMolcas+QCMaquis](https://mp.weixin.qq.com/s/Gb1Lzv1bcQmvuHMZjAQLxQ)  
[安装基于openmpi的mpi4py](https://mp.weixin.qq.com/s/f5bqgJYG5uAK1Zubngg65g)  
[自动做多参考态计算的程序MOKIT](https://mp.weixin.qq.com/s/bM244EiyhsYKwW5i8wq0TQ)  

also these installation instructions on GitLab  
[离线安装PySCF-2.x](https://gitlab.com/jxzou/qcinstall/-/blob/main/%E7%A6%BB%E7%BA%BF%E5%AE%89%E8%A3%85PySCF-2.x.md)  
[离线安装PySCF-2.x-extensions](https://gitlab.com/jxzou/qcinstall/-/blob/main/%E7%A6%BB%E7%BA%BF%E5%AE%89%E8%A3%85PySCF-2.x-extensions.md?ref_type=heads) (for pyscf[dmrgscf], etc.)  
[离线安装OpenMolcas-v22.06](https://gitlab.com/jxzou/qcinstall/-/blob/main/%E7%A6%BB%E7%BA%BF%E5%AE%89%E8%A3%85OpenMolcas-v22.06.md)  
[编译MPI并行版OpenMolcas](https://gitlab.com/jxzou/qcinstall/-/blob/main/%E7%BC%96%E8%AF%91MPI%E5%B9%B6%E8%A1%8C%E7%89%88OpenMolcas.md)  
[block2的编译和安装](https://gitlab.com/jxzou/qcinstall/-/blob/main/block2%E7%9A%84%E7%BC%96%E8%AF%91%E5%92%8C%E5%AE%89%E8%A3%85.md)  
[离线安装量子化学软件Dalton](https://gitlab.com/jxzou/qcinstall/-/blob/main/%E7%A6%BB%E7%BA%BF%E5%AE%89%E8%A3%85%E9%87%8F%E5%AD%90%E5%8C%96%E5%AD%A6%E8%BD%AF%E4%BB%B6Dalton.md)

