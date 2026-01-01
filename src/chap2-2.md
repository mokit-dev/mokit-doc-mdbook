# 2.2 Conda Installation and Other Pre-build

This section goes over several installing approaches that, unlike [Section 2.3](./chap2-3.md), does not require building from source manually. 

1. Install from conda (for Linux and MacOS). Binary package is provided in this approach.
2. Use homebrew build (for MacOS only). This approach will build it automatically.
3. Manually download pre-built Linux binaries (it's not a conda package). 
    > Unlike conda, this approach will not take care of dependency versions (like numpy) for you, and you need to set environment variables manually. So this approach is usually not recommended. 
    >
    > However, this approach does not require network, which may be a reason for choosing it.

4. Even simpler way to use pre-built Linux binaries. 

**Setups after installation**

Like [Section 2.3](./chap2-3.md), there is still something to do after "installation".

1. Setup the environment variables of MOKIT itself. Except conda users, all other users need to set that, which is mentioned at the end of each approaches.
2. Setup the environment variables of dependencies, like Gaussian, GAMESS, PySCF, etc. Please read [Section 2.5](./chap2-5.md) to determine which dependencies are necessary for you and read [Section 2.3.4](./chap2-3.md#234-environment-variables) to set up them.

## 2.2.1 Online Installation
You can choose option 1 or 2 below. After mokit is successfully installed, if you want GAMESS to be called by `automr`, you need to [install GAMESS properly](./chap4-4.md#4410-gvb_prog) and write related environment variables.

### Option 1: Install from conda (for Linux and MacOS)
This is the easiest way, but network is required to auto-download the requirements. 

For Linux x86-64 platform, 
we used to ship MOKIT at two channels, the `mokit` channel (used with anaconda default main channel) and the `mokit/label/cf` channel (used with conda-forge).
Both the default and conda-forge channel require `glibc >= 2.17`.
Starting from v1.2.7rc15, these two channels are merged into one channel `mokit` (used with conda-forge). `mokit/label/cf` can still be used as an alias for `mokit`.

For MacOS arm64 platform, there's only one `mokit` channel. Other platforms are not supported yet, like Linux aarch64 or MacOS x86-64.

<!--Replace conda with mamba/micromamba is ok in principle.-->

If you have no access to network, but still do not want to compile MOKIT manually, you can try options in [Section 2.2.2](#222-pre-built-linux-executables-and-libraries).

#### Use MOKIT with conda-forge channel

Creating a new environment before installing is highly recommended, to avoid changing your base environment. You can create the environment in one go like
```
conda create -n mokit-py311 python=3.11 mokit -c mokit -c conda-forge
conda activate mokit-py311
```

For Linux x86-64, you can use any version of Python 3.9-3.12. For MacOS arm64, only 3.11 is available. 
If you have enabled conda-forge (by `conda config --add channels conda-forge` or modifying condarc), you can
omit the `-c conda-forge` option.

Each time when you login onto the machine, you need to activate the virtual environment by `conda activate mokit-py311` and then you can use MOKIT on the current computer/node. But if you want to submit MOKIT jobs to a queue on Cluster（集群）, things are somewhat different and please read [Section 2.4.3](./chap2-4.html#243-use-mokit-on-cluster) carefully.

If you have not installed PySCF and you want to install it now, you can follow the [instruction below](#install-pyscf-with-mokit).

See [here](#update-mokit-with-conda) for updating mokit with conda.



#### Install pyscf with MOKIT

There are two options of online installation:

1. run `pip install pyscf` in your virtual environment.
1. install with conda. pyscf and MOKIT can be installed in the conda-forge channel at the same time
```
conda install mokit pyscf -c mokit -c conda-forge
```
or separately
```
conda install pyscf -c conda-forge
conda install mokit -c mokit -c conda-forge
```

#### Update MOKIT with conda

Usually the following command works. 
```
conda update mokit -c mokit -c conda-forge
```

Sometimes it may fail to find the latest version of MOKIT, especially when you haven't update mokit for a few months or use conda-forge in which dependencies evolve quite fast. 
In this case, there's a few workarounds:

(1) remove and install
```
conda uninstall mokit numpy openblas -c mokit -c conda-forge
conda install mokit -c mokit -c conda-forge
```
(2) update mokit along with some dependencies
```
conda update mokit openblas -c mokit -c conda-forge
```
(3) install a specified version (please visit [here](https://anaconda.org/mokit/mokit) to see the latest version number)
```
conda install mokit=1.2.7rc16 -c mokit -c conda-forge
```

> [!IMPORTANT]
> If you have installed MOKIT by `conda install mokit -c mokit` with the version <= 1.2.7rc14 and want to update it, please use the "remove and install" method.

### Option 2: Use homebrew-toolchains (for MacOS only)
* Prerequisites: 
    - You need to install [homebrew](https://brew.sh) on your mac 


> If you are mainland China user, follow [brew mirrors help doc](https://mirrors.ustc.edu.cn/help/brew.git.html) to install prerequisites
* A detailed brew-tap install guideline is located in [homebrew-mokit github repo](https://github.com/ansatzX/homebrew-mokit)
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### Release Installation

Assume you will use mokit in a python 3.9 environment (3.8-3.11 are all available)

```
brew install ansatzx/homebrew-mokit/mokit --with-py39
```

or

```
brew tap ansatzx/homebrew-mokit
brew install mokit --with-py39 
```

However, the release version is not up-to-date, it's highly recommended to install the latest commit.

#### Latest Commit Installation

Also assume you use mokit in a py-39 environment.

```
brew install ansatzx/homebrew-mokit/mokit --with-py39 --HEAD
```

Finally, follow caveats guides, add these environment variables in your `~/.zshrc` (or bash/fish etc. profile).

You can run `brew info mokit` to check details.

```
export MOKIT_ROOT="$(brew --prefix mokit)"
export PATH=$MOKIT_ROOT/bin:$PATH
export PYTHONPATH=$MOKIT_ROOT:$PYTHONPATH
export LD_LIBRARY_PATH=$MOKIT_ROOT/mokit/lib:$LD_LIBRARY_PATH
```

#### Upgrade Mokit

##### release upgrade 

```
brew update -f 
```

Run the command below if there's a newer mokit release.

```
brew upgrade mokit  # or `brew upgrade ` to upgrade everything

```
##### latest commit upgrade 

```
brew upgrade --fetch-HEAD mokit
```




## 2.2.2 Pre-built Linux Executables and Libraries

Unlike the conda install approach, using pre-built MOKIT in this subsection do not require network. If you want full functionality of MOKIT, you still need to have necessary dependencies: Python3 environment and NumPy, which can be achieved by anaconda/miniconda (Read [here](#how-to-choose-anaconda-version-if-installing-offline) for installing anaconda offline). If you only want part of MOKIT, especially some certain binary utilities, see [Section 2.2.3](#223-only-want-frag_guess_wfn) for a simpler instruction.

### Download

<!--[centos7_conda_py38](https://gitlab.com/jxzou/mokit/-/jobs/artifacts/master/download?job=centos7_conda_py38)  -->
[centos7_conda_py39](https://gitlab.com/jxzou/mokit/-/jobs/artifacts/master/download?job=centos7_conda_py39)   
[py39_gcc10](https://gitlab.com/jxzou/mokit/-/jobs/artifacts/master/download?job=py39_gcc10)  
[py310_gcc10](https://gitlab.com/jxzou/mokit/-/jobs/artifacts/master/download?job=py310_gcc10)

After downloading the pre-built artifacts, you need to set the following environment variables (assuming MOKIT is put in `$HOME/software/mokit`) in your `~/.bashrc`:

```bash
export MOKIT_ROOT=$HOME/software/mokit
export PATH=$MOKIT_ROOT/bin:$PATH
export PYTHONPATH=$MOKIT_ROOT:$PYTHONPATH
export LD_LIBRARY_PATH=$MOKIT_ROOT/mokit/lib:$LD_LIBRARY_PATH
export GMS=$HOME/software/gamess/rungms
```
  
The `LD_LIBRARY_PATH` is needed since the OpenBLAS dynamic library is put there.
Remember to modify the `GMS` path to suit your local environment. 


<!--The PYTHONPATH has changed since MOKIT version 1.2.5rc2.-->


Note that you need to run `source ~/.bashrc` or exit the terminal as well as re-login, in order to activate newly written environment variables.

### How to choose Anaconda Version if installing offline

Installing prebuilt MOKIT offline usually means the user cannot get NumPy with network. 
So it's important to get a certain version of anaconda which provides proper version of NumPy needed by MOKIT. 
The recommended versions of anaconda for some prebuilts are listed below (compatible miniconda version is also listed but it does not mean miniconda comes with NumPy). 
See [Anaconda release note](https://docs.anaconda.com/anaconda/release-notes/) for more information.

If your linux kernel is roughly as old as Centos7's, choose the one started with `centos7_`. 

| Artifacts | Python version | Compatible Anaconda version | Compatible Miniconda version | NumPy version |
| :---: | :---: | :---: | :---: | :---: |
| centos7_conda_py39 | 3.9 | 2022.10 | py39_22.11.1 | 1.21 |
| py39_gcc10 | 3.9 | 2022.10 | py39_22.11.1 | 1.21 |
| py310_gcc10 | 3.10 | 2023.03 | py310_23.3.1 | 1.23 |


> How to download certain version of anaconda?
> > For example, if you are using [TUNA Mirror](https://mirrors.tuna.tsinghua.edu.cn), you can go to [mirrors.tuna.tsinghua.edu.cn/anaconda/archive/](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/) and find `Anaconda3-[some version]-Linux-x86_64.sh`. The page address is similar in other mirror site, like [NJU Mirror](https://mirror.nju.edu.cn).

### Detailed Compatibility Note

| Artifacts | Compatible OS | glibc version | Python version | GCC version | NumPy version |
| :---: | :---: | :---: | :---: | :---: | :---: |
| centos7_conda_py39 | Centos 7 | 2.17 | 3.9 | 5.3 | 1.21 |
| py39_gcc10 | Debian >= 11, Ubuntu >= 20.04 | 2.31 | 3.9 | 10.2 | 1.21 |
| py310_gcc10 | Debian >= 11, Ubuntu >= 20.04 | 2.31 | 3.10 | 10.2 | 1.23 |

Tips:
* Do not extract the zip with right-click (like the one in KDE)! Use `unzip` in command line.
* The artifacts started with 'centos7_conda' need to be used with Anaconda3/Miniconda3, and the rest works with system-provided python (conda is also OK).
* We cannot list every supported linux distribution here, like Rocky Linux, OpenEuler, OpenSUSE etc. Basically the compatibility is determined by `glibc` version. You can check the [Distro compatibility table](https://github.com/mayeut/pep600_compliance?tab=readme-ov-file#distro-compatibility) for relevant information. More compatibility tests and reports are welcome.
* The GCC and NumPy version listed refer to the version used to compile the artifacts. 
  - You don't need to have the same GCC in your local machine.
  - NumPy can be sensitive to version sometimes. Try upgrade (or downgrade) numpy if your python complained about version when `import`. 
  - The NumPy version for each prebuilt is fixed, because offline anaconda users cannot update anything and have to follow the [anaconda version recommendation](#how-to-choose-anaconda-version-if-installing-offline) above. We may switch to newer miniconda image when the current ones are considered too old.



## 2.2.3 Only want `frag_guess_wfn`?

If you do not need full functionality of MOKIT and only want `frag_guess_wfn` for generating the input file of various EDA methods (or other binary utilities, like `fch2mkl`), the easiest way is to download the pre-compiled MOKIT in Section [2.2.2](./chap2-2.html#222-pre-built-linux-executables-and-libraries). There is no need to install Miniconda/Anaconda Python in this case, and no need for `conda install`.

Firstly, download a pre-compiled MOKIT package according to your OS (e.g. CentOS 7), and change the directory name
```
unzip mokit-master_linux_centos7_conda_py39.zip
rm -f mokit-master_linux_centos7_conda_py39.zip
mv mokit-master_linux_centos7_conda_py39 mokit
```

Then write proper environment variables in your `~/.bashrc` file,
```
export MOKIT_ROOT=$HOME/software/mokit
export PATH=$MOKIT_ROOT/bin:$PATH
export LD_LIBRARY_PATH=$MOKIT_ROOT/mokit/lib:$LD_LIBRARY_PATH
```

Remember to run `source ~/.bashrc` to make the environment variables valid. If you are working on a Cluster and using a script to submit any computational task, you should write the environment variables into your script.

