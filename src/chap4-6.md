# 4.6 APIs in MOKIT
Currently only Python APIs are provided. C/C++ APIs may be provided in the future. 

Table of Content

* [4.6.1 Frequently used APIs in MOKIT workflow](#461-frequently-used-apis-in-mokit-workflow)
* [4.6.2 APIs from module `rwgeom`](#462-apis-from-module-rwgeom)
* [4.6.3 Other wavefunction APIs](#463-other-wavefunction-apis)
* [4.6.4 Other functions in rwwfn](#464-other-functions-in-rwwfn)
* [4.6.5 The py2xxx modules](#465-the-py2xxx-modules)
* [4.6.6 The qchem module](#466-the-qchem-module)
* [4.6.7 The mirror_wfn module](#467-the-mirror_wfn-module)


## 4.6.1 Frequently used APIs in MOKIT workflow

`gaussian` module provides some APIs for manipulating .fch(k) files while `rwwfn` provides some APIs for read/write .fch(k) or .log files. 
Some commonly used APIs in these modules are shown below.

1. load_mol_from_fch(fchname)
2. loc(fchname, method, idx)
3. uno(fchname)
4. nio(n_fch, n_1_fch)
5. permute_orb(fchname, orb1, orb2)
6. gen_fcidump(fchname, nacto, nacte, mem=4000, np=None)
7. get_1e_exp_and_sort_pair(mo_fch, no_fch, npair)
8. read_mo_from_fch(fchname, nbf, nif, ab)
9. read_density_from_gau_log(logname, itype, nbf)
10. read_dm_from_fch(fchname, itype, nbf)
11. write_pyscf_dm_into_fch(fchname, nbf, dm, itype, force)
12. export_mat_into_txt(txtname, n, mat, lower, label)
13. pairing_open_with_vir

Meanings of frequently appeared arguments are  
`fchname`: filename of .fch(k) file  
`logname`: filename of .log/.out file  
`nbf`: the number of basis functions  
`nif`: the number of (linear-independent) MOs

Meanings of other arguments are shown in below subsections.

### 4.6.1.1 load_mol_from_fch
Load a PySCF mol object from a Gaussian .fch(k) file. For example, run python in Shell
```python
from pyscf import scf
from mokit.lib.gaussian import load_mol_from_fch
mol = load_mol_from_fch(fchname='00-h2o_cc-pVDZ_0.96_rhf.fchk')
mf = scf.RHF(mol).run()
```
where the module gaussian is actually the file `$MOKIT_ROOT/mokit/lib/gaussian.py`.

### 4.6.1.2 loc
Perform orbital localization for a given .fch(k) file. Two localization methods are supported: Foster-Boys or Pipek-Mezey. For example, run python in Shell
```python
from mokit.lib.gaussian import loc
loc(fchname='benzene_5D7F_rhf.fchk', idx=range(6,21), method='pm')
```
The method option can be either 'boys' or 'pm'. You should specify the orbital indices or range in the `idx` argument. Note that the starting integer index is 0 in Python convention, and the final index cannot be reached. So range(6,21) means orbitals 7-21, which are valence occupied orbitals for benzene. The localized orbitals will be exported/written into a new file with suffix `*_LMO.fch` (in this case, it is `benzene_5D7F_rhf_LMO.fch`).

For system containing only \\( \sigma \\) bonds, these two methods make little difference. While for system containing \\( \sigma \\) and \\( \pi \\) bonds, the Boys localization method tends to mix \\( \sigma \\) and \\( \pi \\) bonds, which leads to 'banana' bonds. The PM localization method tends to keep \\( \sigma \\) and \\( \pi \\) separated. If you want to analyze localized \\( \pi \\) bonds after localization, you should choose `method='pm'`.

For PBC orbital localization the `pbc_loc` function can be used. For example, the following code performs orbital localization with a given CP2K .molden file.
```python
from mokit.lib.gaussian import pbc_loc
pbc_loc('water64-MOS-1_0.molden', box=np.eye(3)*12.42, method='berry')
```
The method can be either 'berry' or 'pm'. The wannier centers of the localized orbitals will be exported into a new file with suffix `*_wannier.xyz` by default.

### 4.6.1.3 uno
Generate UHF natural orbitals(UNOs) from a given Gaussian .fch(k) file. For example

```python
from mokit.lib.gaussian import uno
uno(fchname='benzene_uhf.fch')
```

### 4.6.1.4 nio
Generate natural ionization orbitals (NIOs) for N -> (N-1) electrons ionization process (see the original paper [JCP 2016, 144, 204117](https://doi.org/10.1063/1.4951738)). It is simply calculated by diagonalizing `(P_N - P_(N-1))*S`. This way of obtaining natural orbitals are also called natural difference orbitals (NDOs). The "lost" electron is mainly ionized from the NIO which has the eigenvalue closest to 1.0. This can be viewed as a good approximation to Dyson orbitals, while the computational cost of NIOs is usually much less than that of Dyson orbitals. 

Syntax: `nio(n_fch, n_1_fch)`, where `n_fch`/`n_1_fch` are the .fch(k) file for N/(N-1) electron state, respectively. An example is shown below

```python
from mokit.lib.gaussian import nio
nio('C6H12O3_neutral.fch','C6H12O3_frag1_uhf.fch')
```

These two fch files must include correct total densities in `Total SCF Density` section. For routine HF/DFT calculations, this is automatically ensured. For automatic CASSCF calculations performed by MOKIT, the density are also ensured to be correct. For other types of calculations, it is the users' responsibility to make sure whether the total densities are correctly generated and stored.

The wave function in two .fch(k) files can either be RHF or UHF-type, as long as the calculations are performed appropriately. Here "UHF-type" means a UHF/UKS calculation.

### 4.6.1.5 permute_orb
Swap/exchange two orbitals in a given .fch(k) file. For example

```python
from mokit.lib.gaussian import permute_orb
permute_orb('ethanol_rhf_proj_loc_pair2gvb8_s.fch',6,13)
```

then the MO 6 and MO 13 will be swapped/exchanged. The index of the first orbital starts from 1.

### 4.6.1.6 gen_fcidump
Generate a FCIDUMP file which contains the effective 1e integrals and 2e integrals from a given Gaussian .fch(k) file. Such a FICUDMP file can be used in CASCI, DMRG-CASCI, GVB-BCCC, or any post-CASCI calculations. For example

```python
from mokit.lib.gaussian import gen_fcidump
gen_fcidump(fchname='anthracene_cc-pVDZ_uhf_uno_asrot2gvb7_s.fch',nacto=14,nacte=14)
```

where the arguments `nacto` and `nacte` are the number of active orbitals and the number of active electrons, respectively. This module requires the PySCF installed.

### 4.6.1.7 get_1e_exp_and_sort_pair
Compute 1e expectation values of MOs in file `mo_fch`, using information from
file `no_fch` (which usually includes NOs).
Sort paired MOs in `mo_fch` by 1e expectation values.

```python
from mokit.lib.rwwfn import get_1e_exp_and_sort_pair as sort_pair
sort_pair(mo_fch, no_fch, npair)
```
### 4.6.1.8 read_mo_from_fch
Read MOs from a Gaussian .fch(k) file. For example

```python
from mokit.lib import rwwfn
mo = rwwfn.read_mo_from_fch(fchname='00-h2o_cc-pVDZ_1.5.fchk',nbf=24,nif=24,ab='a')
```

Argument `ab`: character with length=1, 'a'/'b' for reading alpha/beta orbitals.

See also [4.6.4.1](#4641-readwrite-mo) to read/write MOs from files of other programs.

### 4.6.1.9 read_density_from_gau_log
Read various types of density matrix from a Gaussian output file. For example, read the Alpha Density Matrix from a .log file

```python
from mokit.lib import rwwfn
den = rwwfn.read_density_from_gau_log(logname='00-h2o_cc-pVDZ_1.5.log', itype=2, nbf=24)
```

argument `itype`:  
1/2/3 for Total/Alpha/Beta Density Matrix.

### 4.6.1.10 read_dm_from_fch
Read various types of density matrix from a Gaussian .fch(k) file. For example, read the Total SCF Density from a .fchk file

```python
from mokit.lib import rwwfn
den = rwwfn.read_dm_from_fch(fchname='00-h2o_cc-pVDZ_1.5.fchk',itype=1,nbf=24)
```

| itype | type of density | itype | type of density |
| --- | --- | --- | --- |
| 1 | Total SCF Density | 6 | Spin MP2 Density |
| 2 | Spin SCF Density | 7 | Total CC Density |
| 3 | Total CI Density | 8 | Spin CC Density |
| 4 | Spin CI Density | 9 | Total CI Rho(1) |
| 5 | Total MP2 Density | 10 | Spin CI Rho(1) |

See also [4.6.4.4](#4644-readwrite-density-and-other-matrices) for other operations on density matrix. 

### 4.6.1.11 write_pyscf_dm_into_fch
Write a PySCF density matrix into a given Gaussian .fch(k) file. This module does two things: (1) deal with the order of basis functions and their normalization factors, (2) then export density matrix into a given .fch(k) file. All arguments of this module are shown below

```python
write_pyscf_dm_into_fch(fchname, itype, nbf, dm, force)
```

where `dm` is the PySCF density matrix with dimension (nbf,nbf). `itype` has the same meaning with that in [read_dm_from_fch](#46110-read_dm_from_fch). `force` is a bool variable, and its meaning is:

(1) If the string corresponding to `itype` (e.g. 'Total SCF Density') can be found/located in the given .fch file, this parameter will not be used, i.e. setting True/False does not matter.

(2) If the string corresponding to `itype` cannot be found, setting `force=True` will enforce writing density matrix into the given file; setting `force=False` will stop/abort the program and signal errors immediately (this can be used to check whether desired strings exists in the specified file).

You should use this module via

```python
from mokit.lib.py2fch import write_pyscf_dm_into_fch
```

Note that this module cannot generate a .fch(k) file from scratch, the user must provide one such file. The recommended approach is firstly using the utility `bas_fch2py` to generate PySCF input file or using the Python module `load_mol_from_fch` to a generate proper PySCF object, then do computations in PySCF. Finally using the module `write_pyscf_dm_into_fch` to export desired density matrix.

### 4.6.1.12 export_mat_into_txt
Export a square matrix into a plain text file. The example of exporting transition density matrix is shown in Section 4.6.1.11. Here I offer one more example - export the lower triangle of a symmetric AO-basis overlap matrix

```python
from mokit.lib import rwwfn
S = rwwfn.read_int1e_from_gau_log(logname='00-h2o_cc-pVDZ_1.5.log',itype=1,nbf=24)
rwwfn.export_mat_into_txt(txtname='ovlp.txt',n=24,mat=S,lower=True,label='Overlap')
```

| Argument | Explanation |
| --- | --- |
| txtname | file name |
| mat | the square matrix with dimension (n,n) |
| lower | True/False for whether or not printing only the lower triangle part of a matrix |
| label | a string, the meaning of exported matrix (provided by yourself) |

### 4.6.1.13 pairing_open_with_vir
Paring each singly occupied orbital with a virtual one, for a high spin GVB wave function. These new pairs would be put after all normal GVB pairs. Such type of .fch file may be used for high-spin GVB-BCCC calculations. The order of MOs before calling this function is expected to be docc-socc-pair-vir. The order of MOs after calling this function would be docc-pair-(socc-vir1)-vir2. The input file `fchname` will be updated.

```python
from mokit.lib.rwwfn import pairing_open_with_vir
pairing_open_with_vir(fchname='ben_triplet_uhf_uno_asrot2gvb2.fch')
```

The input filename is supposed to end with `gvbN.fch`. For example, `gvb2.fch` will be identified as 2 GVB pairs. The number of doubly and singly occupied orbitals are automatically determined from information in .fch file. If a singlet .fch file is provided (i.e. no singly occupied orbital), a warning would be printed and `fchname` would be left unchanged. The `Alpha Orbital Energies` section in .fch file would be updated as: doubly occupation orbitals with 2.0; normal GVB pairs with 1.8/0.2; singly occupied orbitals (paired with virtual MOs) with 1.0/0.0; and remaining virtual orbitals with 0.0. These occupation numbers are not real occupation numbers from any GVB calculation, but just some numerical values for visualizing orbitals conveniently (i.e. to remind the user that different ranges for various types of orbitals).

If you do not have such a .fch file and want to create one, you can run the following Shell commands after an `automr` GVB or CASSCF calculation
```
cp ben_triplet_uhf_uno_asrot2gvb2_s.fch ben_triplet_uhf_uno_asrot2gvb2.fch
dat2fch ben_triplet_uhf_uno_asrot2gvb2.dat ben_triplet_uhf_uno_asrot2gvb2.fch
```
the filenames above come from a triplet CASSCF(6,6) calculation for benzene.



## 4.6.2 APIs from module `rwgeom`

### 4.6.2.1 read_natom_from_pdb
Read the number of atoms from a given pdb file. If there exists more than one frame in the pdb file, only the 1st frame will be detected. See an example in [Section 4.6.2.4](#4624-writeframeintopdb).

### 4.6.2.2 read_nframe_from_pdb
Read the number of frames from a given pdb file. See an example in [Section 4.6.2.4](#4624-write_frame_into_pdb).

### 4.6.2.3 read_iframe_from_pdb
Read the i-th frame from a given pdb file. See an example in [Section 4.6.2.4](#4624-write_frame_into_pdb).

### 4.6.2.4 write_frame_into_pdb
Write a frame into the given pdb file. A python script is shown below, to illustrate how to use these pdb-related APIs:

```python
import mokit.lib.rwgeom as rg
natom = rg.read_natom_from_pdb(pdbname='test.pdb')
cell, elem, resname, coor = rg.read_iframe_from_pdb('test.pdb', 10, natom)
for i in range(0,natom):
  s1 = elem[i][0].decode('utf-8')
  s2 = elem[i][1].decode('utf-8')
  print(s1, s2)
rg.write_frame_into_pdb('a.pdb', 10, natom, cell, elem, resname, coor, False)
```

### 4.6.2.5 fch2hess
Extract hessian data from a specified Gaussian .fch(k) file and write an ORCA .hess file. This is useful for cases like excited state geometry optimization, transition state geometry optimization and ESD calculations using ORCA, where one can perform the analytic hessian calculation using Gaussian firstly and then feed the Hessian matrix to ORCA. ORCA does not have analytic hessian for TDDFT so far.

```python
from mokit.lib.rwgeom import fch2hess
fch2hess('h2o_freq.fch')
```
The file `h2o_freq.hess` would be generated. Note that `h2o_freq.fch` is supposed to be generated by a Gaussian frequency calculation (or in any other possible ways to include reasonable hessian data), otherwise `fch2hess` cannot work normally. An example about excited state geometry optimization is shown below.

Assuming we have performed a Gaussian TDDFT frequency calculation for the azobenzene molecule using the input
```
%chk=azobenzene_S1_freq.chk
%mem=160GB
%nprocshared=64
#p TD freq PBE1PBE/6-311G(d,p) em=GD3BJ 5D 7F nosymm int=nobasistransform

Title Card Required

0 1
N    0.726960   -0.164724    0.503582
...
```
and we have the checkpoint file `azobenzene_S1_freq.chk`. Run the following commands
```
formchk azobenzene_S1_freq.chk azobenzene_S1_freq.fch
fch2mkl azobenzene_S1_freq.fch -dft 'PBE0 D3BJ'
orca_2mkl azobenzene_S1_freq_o -gbw
mv azobenzene_S1_freq_o.gbw azobenzene_S1_freq.gbw
```

> [!NOTE]
> In an ORCA geometry optimization job, we need to specify both `MOread` and `%moinp` in order to read MOs from .gbw file. It is not like the single-point calculation case that ORCA will automatically reads MOs from .gbw file as long as .inp/.gbw share the same basename. And when using `%moinp`, the filename in double quotation marks cannot be the same as that generate by .inp file, so here we use `azobenzene_S1_freq.gbw` instead of `azobenzene_S1_freq_o.gbw`.

Now we have three related files
```
azobenzene_S1_freq.gbw
azobenzene_S1_freq_o.inp
azobenzene_S1_freq_o.mkl
```

Start Python and type
```python
from mokit.lib.rwgeom import fch2hess
fch2hess('azobenzene_S1_freq.fch')
```
we further obtain the file `azobenzene_S1_freq.hess`. To perform TD-PBE0-D3(BJ)/6-311G(d,p) geometry optimization using ORCA (and read corresponding analytic hessian as well as MOs from Gaussian), we need to open the ORCA input file and make some simple modifications, e.g.
```
%pal nprocs 32 end
%maxcore 5200
! RKS PBE0 D3BJ def2/J RIJCOSX defgrid3 TightSCF noTRAH MOread Opt Freq
%moinp "azobenzene_S1_freq.gbw"
%geom
 InHess Read
 InHessName "azobenzene_S1_freq.hess"
end
%tddft
 NRoots 3
 IRoot 1
end
%scf
...
```

Finally, we can submit the ORCA job like
```
orca azobenzene_S1_freq_o.inp >azobenzene_S1_freq_o.out 2>&1 &
```


## 4.6.3 Other wavefunction APIs
### 4.6.3.1 calc_unpaired_from_fch
Calculate the Yamaguchi's unpaired electrons and Head-Gordon's unpaired electrons using the provided .fch(k) file. Biradical and tetraradical indices are printed as well. This .fch file must include natural orbitals and their corresponding occupation numbers. For example, a UNO, GVB or CASSCF NO .fch file is expected. DO NOT use a UHF .fch file. A python script is shown below

```python
from mokit.lib.wfn_analysis import calc_unpaired_from_fch
calc_unpaired_from_fch(fchname='00-h2o_cc-pVDZ_1.5_uhf_uno_asrot2gvb4_s.fch', wfn_type=2, gen_dm=False)
```

The argument `wfn_type` has values 1/2/3 for UNO/GVB/CASSCF NOs, respectively. The example above will print the following
```
----------------------- Radical index -----------------------
biradical character   (2c^2) y0=  0.155
tetraradical character(2c^2) y1=  0.155
Yamaguchi's unpaired electrons  (sum_n n(2-n)      ):  1.183
Head-Gordon's unpaired electrons(sum_n min(n,(2-n))):  0.639
Head-Gordon's unpaired electrons(sum_n (n(2-n))^2  ):  0.328
-------------------------------------------------------------
```

If the argument `gen_dm` is set to `True`, then a file like `*_unpaired.fch` will also be generated, in which the unpaired electron density is stored. The unpaired density can be visualized by GaussView or Multiwfn+VMD.

Yamaguchi's biradical index for UHF: \\( t = (n_{\text{HONO}} - n_{\text{LUNO}})/2, y = 1 - 2t/(1 + t^2) \\)
 
Yamaguchi's biradical index for CAS(2,2): \\( y = 2{c_2}^{2} = n_{\text{LUNO}} \\)

where \\( c_2 \\) is the CI coefficients of the 2nd configurations in CASCI wave function (assuming natural orbitals are used). 
Note that there is no unique way to define biradical (or tetraradical, etc) index for general cases including CASSCF (*m*,*m*) where *m*>=4, or GVB(*n*) where *n*>=2. 
The \\( y_i = n_{\text{LUNO}+i} \\) formula is adopted for these general cases.

There are also modules which calculate the number of unpaired electrons of GVB by reading information from GAMESS .dat/.gms file:
```python
from mokit.lib.wfn_analysis import calc_unpaired_from_dat
calc_unpaired_from_dat(datname='00-h2o_cc-pVDZ_1.5_uhf_uno_asrot2gvb4_s.fch',mult=1)
```

It requires the user to input the spin multiplicity since there is no spin information in .dat file. Or you can use
```python
from mokit.lib.wfn_analysis import calc_unpaired_from_gms_out
calc_unpaired_from_gms_out(outname='00-h2o_cc-pVDZ_1.5_uhf_uno_asrot2gvb4.gms')
```

Reference for these two types of unpaired electrons:  
[1] Theoret. Chim. Acta (Berl.) 48, 175-183 (1978). DOI: 10.1007/BF00549017.  
[2] Chemical Physics Letters 372 (2003) 508–511. DOI: 10.1016/S0009-2614(03)00422-6.

### 4.6.3.2 get_gvb_bond_order_from_fch
Perform GVB bond order analysis using `_s.fch` and `_s.dat` files.
```python
from mokit.lib.wfn_analysis import population
population.get_gvb_bond_order_from_fch('ben_uhf_uno_asrot2gvb15_s.fch')
```

The `_s.fch` and `_s.dat` files can be obtained after a GVB or CASSCF job.

### 4.6.3.3 gen_no_using_density_in_fch
Generate natural orbitals using specified density in a .fch file. The density is read from the specified section of .fch file and the AO-basis overlap is obtained by calling Gaussian. An example is shown below

```python
from mokit.lib.lo import gen_no_using_density_in_fch
gen_no_using_density_in_fch('ben.fch', 1)
```

where 1 means reading density from "Total SCF Density" section.

### 4.6.3.4 gen_cf_orb
Generate Coulson-Fischer orbitals using GVB natural orbitals from a GAMESS .dat file. The Coulson-Fischer orbitals are non-orthogonal and the GVB natural orbitals are orthogonal, so this module actually performs an orthogonal -> non-orthogonal orbital transformation. This transformation requires the information of GVB pair coefficients so currently only the GAMESS .dat file is accepted while the .fch file is not supported. An example is shown below

```python
from mokit.lib.lo import gen_cf_orb
gen_cf_orb(datname='naphthalene_gvb5.dat',ndb=29,nopen=0)
```

where `ndb` is the number of doubly occupied orbitals and `nopen` is the number of singly occupied orbitals. A new file named naphthalene_gvb5_new.dat will be generated. If you want to visualize the Coulson-Fischer orbitals, you need to use the `dat2fch` utility to transfer orbitals from .dat to .fch.

### 4.6.3.5 make_orb_resemble
Make a set of target MOs resembles the reference MOs. Ddifferent basis set in two .fch files are allowed, but their geometries should be identical or very similar. An example is shown below

```python
from mokit.lib.gaussian import make_orb_resemble
make_orb_resemble(target_fch, ref_fch, nmo=None, align=False)
```

where  
`target_fch`: the .fch file which holds MOs to be updated  
`ref_fch`: the .fch file which holds reference MOs  
`nmo`: indices 1~nmo MOs in `ref_fch` will be set as reference MOs. If `nmo` is not given, it will be set as na (number of alpha electrons) for R(O)HF-type wave function, and set to na/nb respectively for UHF-type wave function. For GVB and CASSCF methods, one should manully specify `nmo`, which is usually equal to ndb+nact (the number of doubly occupied orbitals + the number of active orbitals). For example, `nmo=7` is required for a CASSCF(4,4) calculation of the H2O molecule (3 doubly occ. + 4 active orbitals).  
`align`: whether to align two molecules in files `target_fch` and `ref_fch`. The default is `False`, i.e. assuming the geometries are already in a best alignment. If this is not your case, you need to set `align=True`.

### 4.6.3.6 proj2target_basis
Project MOs of the original basis set onto the target basis set. `cart` is True/False for Cartesian-type or spherical harmonic type functions, respectively. `target_basis` can be smaller than, equal to, or larger than the basis set in `fchname`, although usually a larger basis set would be used.

```python
from mokit.lib.gaussian import proj2target_basis
proj2target_basis(fchname, target_basis='cc-pVTZ', nmo=None, cart=False)
```

Note: use this module to projecting MOs between two all-electron basis sets, or between two basis sets with the same ECP/PP. For example, the following 3 cases are recommended  
(1) cc-pVDZ -> cc-pVTZ;  
(2) cc-pVDZ -> def2TZVP (if the studied molecule include no element >Kr);  
(3) def2SVP -> def2TZVP.  
The basis sets def2SVP and def2TZVP have the same ECP/PP data, and they differ only in orbital basis data.

The following 3 cases are not recommended  
(1) LANL2DZ -> cc-pVTZ;  
(2) cc-pVDZ -> LANL2TZ(f);  
(3) LANL2DZ -> def2TZVP.  
when the studied molecule include any element >Ne. This is because the number of doubly occupied core MOs are not consistent among these basis sets with ECP/PP. If you use projected MOs to perform SCF calculations, SCF will probably be oscillating.

`cart=True`/`cart=False` means using Cartesian type (6D 10F) or spherical harmonic (5D 7F) type basis functions. Usually the spherical harmonic is recommended.

`nmo`: indices 1~nmo MOs in `fchname` will be projected onto the target basis set. If `nmo` is not given, it will be set as na (number of alpha electrons) for R(O)HF-type wave function, and set to na/nb respectively for UHF-type wave function. For GVB and CASSCF methods, one should manully specify `nmo`, which is usually equal to ndb+nact (the number of doubly occupied orbitals + the number of active orbitals). For example, if one wants to project CASSCF(4,4)/cc-pVDZ MOs onto CASSCF(4,4)/cc-pVTZ for H2O, `nmo=7` is required (3 doubly occ. + 4 active orbitals), whereas remaining orbitals at cc-pVTZ are constructed as orthonormalized virtual MOs using the PAO (projected atomic orbitals) technique. By the way, the function `make_orb_resemble()` is called with `proj2target_basis()`, so you see they share the same parameter `nmo`.

### 4.6.3.7 lin_comb_two_mo
Perform \\(\sqrt{2}/2 \\) (mo1+mo2) and \\(\sqrt{2}/2 \\) (mo1-mo2) unitary transformation for two specified MOs in a Gaussian .fch file. 
When the \\( \sigma \\) and \\( \pi \\) orbitals of a double bond are mixed (i.e. a banana bond), this can be used to make them separated.

```python
from mokit.lib.gaussian import lin_comb_two_mo
lin_comb_two_mo(fchname, orb1, orb2)
```

`orb1`/`orb2` are in Python convention (starts from 0). You need to call this module 2 times for a double bond, for example

```python
lin_comb_two_mo('polyacene.fch', 30, 31) # for bonding orbitals
lin_comb_two_mo('polyacene.fch', 50, 51) # for anti-bonding orbitals
```

### 4.6.3.8 find_antibonding_orb
Construct antibonding orbitals according to bonding orbitals provided. Sometimes the users already have a set of bonding orbitals, e.g. constructed by IAO, Boys/PM localization as well as manual selection, etc. And they want to obtain the corresponding antibonding orbitals without changing any bonding orbital. Then this module would be best choice. The orbital index range `[i1,i2]` contains the bonding orbitals, the index range `[i3,nif]` contains orbitals which can be optimized/updated to obtain antibonding orbitals. `i3>i2` is required. Once the antibonding orbitals are generated, they will be stored in index range `[i3,i3+i2-i1]`. The orbitals `[1,i3-1]` will not be changed by this module, which is useful when the user wants to keep some orbitals fixed. All MOs are orthonormal before and after using this module.

```python
from mokit.lib.wfn_analysis import find_antibonding_orb
find_antibonding_orb(fchname, i1, i2, i3)
```

### 4.6.3.9 reorder2dbabasv
Reorder MOs in the file `gvbN_s.fch`. A new file `gvbN_new.fch` would be generated. The order of MOs in `gvbN_s.fch` is expected to be docc-bonding-socc-antibonding-vir. The order of MOs in `gvbN_new.fch` would be docc-bonding1-antibonding1-bonding2-antibonding2-...-socc-vir.

```python
from mokit.lib.rwwfn import reorder2dbabasv

reorder2dbabasv(fchname='ben_triplet_uhf_uno_asrot2gvb2_s.fch')
```

## 4.6.4 Other functions in rwwfn

### 4.6.4.1 read/write mo

```
read_mo_from_chk_txt(txtname, nbf, nif, ab, mo)  
read_mo_from_orb(orbname, nbf, nif, ab, mo)  
read_mo_from_xml(xmlname, nbf, nif, ab, mo)  
read_mo_from_bdf_orb(orbname, nbf, nif, ab, mo)  
read_mo_from_dalton_mopun(orbname, nbf, nif, coeff)  
read_mo_from_mos(fname, nbf, nif, coeff)
write_mo_into_fch(fchname, nbf, nif, ab, mo)  
write_mo_into_psi_mat(matfile, nbf, nif, mo)  
copy_orb_and_den_in_fch(fchname1, fchname2, deleted)  
```
See also [4.6.1.8](#4618-read_mo_from_fch).

### 4.6.4.2 read/write eigenvalue or occupation number

```
read_eigenvalues_from_fch(fchname, nif, ab, noon)
read_ev_from_mkl(mklname, nmo, ab, ev)
read_ev_from_bdf_orb(orbname, nif, ab, ev)
read_ev_from_amo(amoname, nif, ab, ev)
read_on_from_orb(orbname, nif, ab, on)
read_on_from_gms_dat(datname, nmo, on, alive)
read_on_from_xml(xmlname, nmo, ab, on)
read_on_from_bdf_orb(orbname, nif, ab, on)
read_on_from_dalton_mopun(orbname, nif, on)
read_on_from_dalton_out(outname, nacto, on)
read_on_from_mkl(mklname, nmo, ab, on)
read_on_from_molden(molden, nmo, occ)
read_nmo_from_molden(molden, nmo_a, nmo_b) 
write_eigenvalues_to_fch(fchname, nif, ab, on, replace)  
write_on_to_orb(orbname, nif, ab, on, replace)  
```

### 4.6.4.3 read/write basic information

```
modify_irohf_in_fch(fchname, k)  
read_mult_from_fch(fchname, mult)  
read_charge_and_mult_from_fch(fchname, charge, mult)  
read_charge_and_mult_from_mkl(mklname, charge, mult)
modify_charge_and_mult_in_fch(fchname, charge, mult) 
read_na_and_nb_from_fch(fchname, na, nb)  
read_nbf_and_nif_from_fch(fchname, nbf, nif)  
read_nbf_and_nif_from_orb(orbname, nbf, nif)  
read_cart_nbf_from_dat(datname, nbf)  
read_ncontr_from_fch(fchname, ncontr)  
read_shltyp_and_shl2atm_from_fch(fchname, k, shltyp, shl2atm)  
```

### 4.6.4.4 read/write density and other matrices  

**read_int1e_from_gau_log**

Read various one-electron integral matrices from a Gaussian output file. For example, read AO-basis overlap

```python
from mokit.lib import rwwfn
S = rwwfn.read_int1e_from_gau_log(logname='00-h2o_cc-pVDZ_1.5.log',itype=1,nbf=24)
```

Argument `itype`: The type of one-electron integral matrices. Allowed values are 1,2,3,4 for Overlap, Kinetic Energy, Potential Energy, and Core Hamiltonian, respectively.

**Others**
```
read_ovlp_from_molcas_out(outname, nbf, S)  
write_dm_into_fch(fchname, nbf, total, dm)  
copy_dm_between_fch(fchname1, fchname2, itype, total)
detect_spin_scf_density_in_fch(fchname, alive)  
add_density_str_into_fch(fchname, itype)  
update_density_using_mo_in_fch(fchname)  
update_density_using_no_and_on(fchname)  
read_ao_ovlp_from_47(file47, nbf, S)  
```
See also [4.6.1.9](#4619-read_density_from_gau_log), 
[4.6.1.10](#46110-read_dm_from_fch), [4.6.1.11](#46111-write_pyscf_dm_into_fch).

### 4.6.4.5 read energy and other results

```
read_npair_from_uno_out(nbf, nif, ndb, npair, nopen, lin_dep)  
read_gvb_energy_from_gms(gmsname, e)  
read_cas_energy_from_output(cas_prog, outname, e, scf, spin, dmrg, ptchg_e, nuc_pt_e)  
read_cas_energy_from_gaulog(outname, e, scf)  
read_cas_energy_from_pyout(outname, e, scf, spin, dmrg)  
read_cas_energy_from_gmsgms(outname, e, scf, spin)  
read_cas_energy_from_molcas_out(outname, e, scf)  
read_cas_energy_from_orca_out(outname, e, scf)  
read_cas_energy_from_molpro_out(outname, e, scf)  
read_cas_energy_from_bdf_out(outname, e, scf)  
read_cas_energy_from_psi4_out(outname, e, scf)  
read_cas_energy_from_dalton_out(outname, e, scf)  
read_mrpt_energy_from_pyscf_out(outname, ref_e, corr_e)  
read_mrpt_energy_from_molcas_out(outname, itype, ref_e, corr_e)  
read_mrpt_energy_from_molpro_out(outname, itype, ref_e, corr_e)  
read_mrpt_energy_from_orca_out(outname, itype, ref_e, corr_e)  
read_mrpt_energy_from_gms_out(outname, ref_e, corr_e)  
read_mrpt_energy_from_bdf_out(outname, itype, ref_e, corr_e, dav_e)  
read_mrci_energy_from_output(CtrType, mrcisd_prog, outname, ptchg_e, nuc_pt_e, davidson_e, e)  
read_mcpdft_e_from_output(prog, outname, ref_e, corr_e)  
find_npair0_from_dat(datname, npair, npair0)  
read_no_info_from_fch(fchname, nbf, nif, ndb, nopen, nacta, nactb, nacto, nacte)  
```

### 4.6.4.6 miscellaneous

```
determine_sph_or_cart(fchname, cart)  
check_cart_in_fch(fchname, cart)  
check_sph_in_fch(fchname, sph)  
check_if_uhf_equal_rhf(fchname, eq)  
gen_no_from_density_and_ao_ovlp(nbf, nif, P, ao_ovlp, noon, new_coeff)  
sort_no_by_noon(fchname, i1, i2)
get_core_valence_sep_idx(fchname, idx)
fch_r2u(fchname, brokensym)
```

## 4.6.5 The py2xxx modules
There are a few py2xxx modules. The functions provided are as follows.

```python
py2amesp(mf, aipname)
py2bdf(mf, inpname, write_no=None)
py2cfour(mf)
py2dalton(mf, inpname)
py2gms(mf, inpname, npair=None, nopen=None)
py2molcas(mf, inpname)
py2molpro(mf, inpname)
py2mrcc(mf)
py2orca(mf, inpname)
py2psi(mf, inpname)
py2qchem(mf, inpname, npair=None)
```

Taking `py2qchem` as an example, it exports MOs from PySCF to Q-Chem. An input file (.in) and a directory containing orbital files will be generated. The restricted/unrestricted-type (i.e. R(O)HF or UHF) can be automatically detected. Two examples are shown below:

(1) Transfer RHF, ROHF or UHF orbitals. Create/Write a PySCF input file, e.g. `h2o.py`
```python
from pyscf import gto, scf
from mokit.lib.py2qchem import py2qchem

mol = gto.M(atom = '''
O  -0.49390246   0.93902438   0.0
H   0.46609754   0.93902438   0.0
H  -0.81435705   1.84396021   0.0
''', basis = 'cc-pVDZ')

mf = scf.RHF(mol).run()
py2qchem(mf, 'h2o.in')
```

> [!NOTE]
> After MOKIT version 1.2.6rc5, a lazy import of functions in this subsection is enabled. `from mokit.lib.py2qchem import py2qchem` can be replaced by `from mokit.lib import *` or `from mokit.lib import py2qchem`.


Run it using python and then a Q-Chem input file `h2o.in` and a scratch directory `h2o` will be generated. The orbital file(s) is put in `h2o/`. If you run
```
qchem h2o.in h2o.out h2o
```
you will find RHF in Q-Chem converges in 2 cycles. If the environment variable `QCSCRATCH` is already defined in your node/computer, the scratch directory `h2o` will be automatically put into `$QCSCRATCH/`; otherwise it will be put in the current directory.

(2) Transfer GVB orbitals
If you have performed GVB calculations and stored GVB orbitals in the object mf, then you can write in python
```python
py2qchem(mf, 'h2o.in', npair=2)
```
to generate the GVB-PP input file of Q-Chem, where `npair` tells `py2qchem` how many pairs you want to calculate.


## 4.6.6 The qchem module
```python
qchem2amesp(fchname, aipname)
qchem2bdf(fchname, inpname)
qchem2cfour(fchname)
qchem2dalton(fchname, dalname)
qchem2gms(fchname, inpname)
qchem2molcas(fchname, inpname)
qchem2molpro(fchname, inpname)
qchem2mrcc(fchname)
qchem2psi(fchname, inpname)
qchem2pyscf(fchname, pyname)
qchem2orca(fchname, inpname)
standardize_fch(fchname)
```

Taking `qchem2molpro` as an example, it will first standardize your provided .fch(k) file and then export MOs from Q-Chem to Molpro. The restricted/unrestricted-type (i.e. R(O)HF or UHF) can be automatically detected. Start Python and run

```python
from mokit.lib.qchem import qchem2molpro

qchem2molpro('water.FChk','h2o.com')
```

Two files(`h2o.com` and `water_std.a`) will be generated. Keywords for reading MOs from `water_std.a` have been written in `h2o.com`.

If you only want to standardize a Q-Chem .fch(k) file and do not want to transfer MOs. Then you only need

```python
from mokit.lib.qchem import standardize_fch

standardize_fch('water.FChk')
```

A file named `water_std.fch` will be generated.


## 4.6.7 The mirror_wfn module
```python
rmsd_wrapper(fname1, fname2)
mirror_wfn(fchname)
mirror_c2c(chkname)
rotate_atoms_wfn(fchname, coor_file)
permute_atoms_wfn(fchname, coor_file)
geom_lin_intrplt(gjfname1, gjfname2, n)
```
The `rmsd_wrapper` function is able to calculate the RMSD value between two molecules. The input files can be any type of xyz/gjf/fch. For example,

```python
from mokit.lib.mirror_wfn import rmsd_wrapper
rmsd_v = rmsd_wrapper('h2o_1.gjf','h2o_2.gjf')
print(rmsd_v)
```
Note that two molecules can be in different orientations, since RMSD calculation will firstly rotate molecules to achieve maximum overlap. But two molecules must have one-to-one correspondence of atomic labels. e.g. H1-H1, C2-C2. An automatic adjustment of atomic labels to achieve maximum overlap has not been implemented yet.

<br>

The `mirror_wfn` function transforms a molecule into its mirror image by multiplying all z-components of Cartesian coordinates with -1. Besides, the MO coefficients as well as density matrix in the .fch(k) file are updated. For example, if we provide a chiral molecule with R-chirality on the carbon center,

```python
from mokit.lib.mirror_wfn import mirror_wfn
mirror_wfn('CHFClBr_R.fch')
```

the function `mirror_wfn` will return a file named `CHFClBr_R_m.fch`, where `_m` means mirror. This new file includes new Cartesian coordinates with S-chirality, updated MO coefficients, total density matrix and spin density matrix (if it was in the CHFClBr_R.fch file).

If you start with the file `CHFClBr_R.chk`, not `CHFClBr_R.fch`, and you want to get the .chk file of the mirror image, you can use the function `mirror_c2c`, which is a wrapper of `formchk -> mirror_wfn -> unfchk`. For example,

```python
from mokit.lib.mirror_wfn import mirror_c2c
mirror_c2c('CHFClBr_R.chk')
```

then you can directly obtain the file `CHFClBr_R_m.chk` without running `formchk` or `unfchk` manually. If you write/create a .gjf file to read MOs from `CHFClBr_R_m.chk`, the SCF will be converged in 1 cycle if you use the same computational level, because the mirror transformation here is exact.

<br>

The function `rotate_atoms_wfn` generates the MO coefficients of a translated and rotated molecule. For example

```python
from mokit.lib.mirror_wfn import rotate_atoms_wfn
rotate_atoms_wfn('h2o.fch','h2o_new.gjf')
```

The original geometry and MO coefficients are stored in `h2o.fch`, while the geometry after translation and rotation is stored in `h2o_new.gjf`. The xyz format is supported as well, e.g. you can provide `h2o_new.xyz` instead. A file named `h2o_r.fch` will be generated, which includes new Cartesian coordinates and MO coefficients. Again, if you read MOs from `h2o_r.fch` and use the same computational level, the SCF will be converged in 1 cycle.

Note that the one-to-one atom correspondence in two files `h2o.fch` and `h2o_new.gjf` must be ensured by the user, this function will check whether the number of atoms in two files are equal to each other, but it will not check the atom correspondence.

<br>

The function `permute_atoms_wfn` generates the MO coefficients of a molecule where some atoms are exchanged/permuted. For example

```python
from mokit.lib.mirror_wfn import permute_atoms_wfn
permute_atoms_wfn('h2o.fch','h2o_new.gjf')
```

The original geometry and MO coefficients are stored in `h2o.fch`, while the geometry after exchanging/permutations are stored in `h2o_new.gjf`. The xyz format is supported as well, e.g. you can provide `h2o_new.xyz` instead. A file named `h2o_p.fch` will be generated, which includes the Cartesian coordinates of `h2o_new.gjf` and generated MO coefficients. Again, if you read MOs from `h2o_p.fch` and use the same computational level, the SCF will be converged in 1 cycle.

Note that only exchanging/permutations of atoms are allowed, while translation or rotation of the molecule are not allowed for this function. In the latter case, you should use `rotate_atoms_wfn` above.

<br>

The function `geom_lin_intrplt` generates a series of Cartesian coordinates by linear interpolation between the initial and final geometry. For example, if we have the geometries of the reactant and product in a Diels-Alder reaction, we can use this function to generate geometries connecting the reactant and product

```
from mokit.lib.mirror_wfn import geom_lin_intrplt
geom_lin_intrplt('DA_reactant.gjf','DA_product.gjf',7)
```

The integer 7 means generating seven geometries along the linear interpolation path. These geometries can be used in subsequent (un)relaxed scan or NEB calculations. Also, you may visualize the generated geometries and choose one as the initial geometry of transition state. The one-to-one atom correspondence in two files should be ensured by the user, since this function will not check that.

