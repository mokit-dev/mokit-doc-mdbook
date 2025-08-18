# 4.5 List of Utilities in MOKIT
**The utilities for transferring MOs** are summarized in the following figure.

<iframe width=750 height=510  style="border:none; background:white;" src="./chap4-5_head.html">
</iframe>

For detailed explanations of all utilities, please read the following subsections. Clicking any utility label (in red color) -- for example, `fch2py` -- in the figure will redirect you to the corresponding subsection.

**Other useful utilities**

| &emsp;| | | | 
| --- | --- | --- | --- | 
| Pre-processing | [add_bgcharge_to_inp](#451-add_bgcharge_to_inp) | [replace_xyz_in_inp](#4543-replace_xyz_in_inp) |
| Read/write fch | [extract_noon2fch](#4517-extract_noon2fch) | [fch_mo_copy](#4532-fch_mo_copy) | [fch_u2r](#4533-fch_u2r) |
| Generating guess | [frag_guess_wfn](#4534-frag_guess_wfn) |
| Math operation | [mo_svd](#4541-mo_svd) | [solve_ON_matrix](#4542-solve_on_matrix) | [gvb_sort_pairs](#4535-gvb_sort_pairs) |


## 4.5.1 add_bgcharge_to_inp
This utility is designed to add background charges to the input file of various software packages. If you do not use background charges in your computation, you can skip this section. But if you do use them (e.g. in subsystems of fragmentation-based or embedding methods), they are not recorded in any .fch(k) file. This can be viewed as a defect of the .fch(k) file. Therefore, the generated input file by utilities `fch2com`, `fch2inp`, `fch2iporb`, `fch2psi` or `bas_fch2py` will contain no background charges either.

To add background charges into the input file, you have to provide a .chg file which contains information of those charges. An example of such file is shown below
```
2
 4.0   0.0   0.0    0.1
-4.0   0.0   0.0    0.1
```
The first line holds the number of point charges. While the charges are written starting from the second line, with format x, y, z, charge.

Note that in all computations of the `automr` program, this situation is explicitly considered. This utility will be automatically called if needed. The only situation when you need this utility is merely using utilities `fch2com`, `fch2inp`, `fch2iporb`, `fch2psi` or `bas_fch2py` (and of course, with background charges).


## 4.5.2 addH2singlet
Add hydrogen atoms onto the principle axes of a high-spin open-shell molecule with a distance around 100.0 Å. The number of added atoms \\( n_{H} \\) = 1, 2, 3, ... for doublet, triplet, quartet, ..., respectively. And the basis set of added hydrogen atoms are automatically set to STO-2G. This utility is designed to transform a high-spin open-shell molecule into a singlet complex, then one can use the singlet GVB-BCCC code to calculate the complex. The electronic energy of the open-shell molecule is obtained by \\( E_{\text{target}} \\) = \\( E_{\text{complex}} \\) - \\( n_{H}*E_{H} \\), where \\( E_{H} \\) is the ROHF/STO-2G energy of an H atom, i.e. -0.454397402 a.u. An example of using this utility is shown below

```
addH2singlet benzene_triplet_gvb14_s.fch
```

## 4.5.3 amo2fch
Convert MOs of [Amesp](www.amesp.xyz) .amo file into Gaussian .fch(k) file.

(1) `amo2fch h2o.amo`  
Convert the file `h2o.amo` to `h2o.fch`. When no .fch(k) file is specifed, the default filename is `xxx.fch`.

(2) `amo2fch h2o.amo h2o_new.fch`  
Convert the file `h2o.amo` to `h2o_new.fch`. Now a filename is specifed by the user. If the specifed .fch(k) file already exists in the current directory (for example, one had run `fch2amo` and performed Amesp calculations before, and now wants to use `amo2fch` to convert MOs back to Gaussian), `amo2fch` will try to use the existing .fch(k) file directly, since this saves some time for reading and conversion. If the specifed file does not exist, `amo2fch` will try to generate one such file from scratch.

(3) `amo2fch h2o.amo h2o.fch -no`  
To be documented.

To transfer MOs from Gaussian to Amesp, you can use the [fch2amo](#4518-fch2amo) utility.


## 4.5.4 bas_fch2py
Generate a PySCF .py file from a Gaussian .fch file. The Cartesian coordinates, basis set data and keywords to read MOs from the .fch file are written in the .py file. Please do not delete these basis set data and do not use PySCF built-in basis set name, since the auto-generated basis set data ensures an exact correspondence of MOs in two programs. Two examples are shown below

(1) transfer RHF/ROHF/UHF/GHF/CAS MOs from Gaussian to PySCF
```
bas_fch2py h2o.fch
```

This will generate a Python input script `h2o.py`. You can find that the last 3 lines in `h2o.py` are commented
```
#dm = mf.make_rdm1()
#mf.max_cycle = 10
#mf.kernel(dm0=dm)
```
If you want to perform a HF calculation, remember to uncomment these lines. If you want to directly perform a CASSCF calculation or perform any orbital localization, just ignore or delete these lines.

(2) transfer DFT MOs from Gaussian to PySCF
```
bas_fch2py h2o.fch -dft
```
Assuming the file `h2o.fch` contains the information of a B3LYP calculation, you can find that the last few lines in `h2o.py` are
```
dm = mf.make_rdm1()
mf = dft.RKS(mol)
mf.xc = 'b3lypg'
mf.grids.atom_grid = (99,590)
mf.verbose = 4
mf.max_cycle = 128
mf.kernel(dm0=dm) 
```
The functional `b3lypg` corresponds to `B3LYP` in Gaussian, and the grid `(99,590)` corresponds to `ultrafine` DFT grid of Gaussian. PySCF usually starts an SCF calculation using the density matrix, so `mf.make_rdm1()` requires to construct the density matrix using input MOs. And `mf.kernel(dm0=dm)` means that the constructed density matrix would be used as the initial guess.

This utility is in fact a wrapper of two utilities `fch2inp` and `bas_gms2py`. So if you only want to use this utility, you need to make sure that `fch2inp` and `bas_gms2py` are available (i.e. also be compiled).

Note that if you use background charges in your studied system, the background charges are not recorded in the .fch(k) file, which might be viewed as a shortcoming of .fch file. So there would be no background charges in the generated .py file. To add background charges, you need to use the utility [add_bgcharge_to_inp](#451-add_bgcharge_to_inp).

See relevant Python modules [fch2py](#4545-fch2py), [py2fch](#4546-py2fch), and [fchk](#4548-fchk).


## 4.5.5 bas_gau2molcas
Transform a basis set file in Gaussian format to another in (Open)Molcas format. If you turn on [RI](#4428-ri) and use OpenMolcas as the `CASSCF_prog`, there is no RI-JI auxiliary basis set file in current version of OpenMolcas package. Therefore, this utility will be called automatically to transform the auxiliary basis set file in `$MOKIT_ROOT/mokit/basis/` directory to the (Open)Molcas syntax. And the transformed file will normally be in `$MOLCAS/basis_library/jk_Basis/`.

You can, of course, use this utility by yourself. An example is shown below
```
bas_gau2molcas def2-universal-jkfit
```
Assuming the basis set file def2-universal-jkfit (in Gaussian format) has been put in the current directory, this command will generate a basis set file named DEF2-UNIVERSAL-JKFIT (in (Open)Molcas format).


## 4.5.6 bas_gms2bdf
Generate two BDF files (`_bdf.inp` and .BAS) from a GAMESS .inp/.dat file. Cartesian coordinates are written in the `_bdf.inp` file, while the basis set data is held in .BAS file. Note that BDF does not support Cartesian-type basis functions, so only spherical harmonic functions will be used. The 'ISPHER' keyword in .inp/.dat file (if any) will be ignored. One example is shown below
```
bas_gms2bdf a.inp
```
Generate a_bdf.inp and A.BAS files, for R(O)HF or UHF type wave function.


## 4.5.7 bas_gms2dal
Generate Dalton .dal and .mol files from a GAMESS .inp/.dat file. The Cartesian coordinates and basis set data are written in the .mol file.
Two examples are shown and explained below

(1) `bas_gms2dal a.inp`  
Generate .mol and .dal file, in which the keywords 'Cartesian' for all atom are written (in order to use pure Cartesian type of basis functions)

(2) `bas_gms2dal a.inp -sph`  
Generate .,mol and .dal file, in which the keywords 'Cartesian' of each atom are not written (in order to use pure spherical harmonic type of basis functions).

## 4.5.8 bas_gms2molcas
Generate an (Open)Molcas .input file from a GAMESS .inp/.dat file. The Cartesian coordinates and basis set data are written in the .input file.
Two examples are shown and explained below

(1) `bas_gms2molcas a.inp`  
Generate .inp file, in which the keywords 'Cartesian' of each atom are written (in order to use pure Cartesian type of basis functions)

(2) `bas_gms2molcas a.inp -sph`  
Generate .inp file, in which the keywords 'Cartesian' of each atom are not written (in order to use pure spherical harmonic type of basis functions).


## 4.5.9 bas_gms2molpro
Generate a Molpro .com file from a GAMESS .inp/.dat file. The Cartesian coordinates and basis set data are written in the .com file. Two examples are shown and explained below

(1) `bas_gms2molpro h2o.inp`  
Generate `h2o.com`, in which the keyword 'Cartesian' is written (in order to use pure Cartesian type of basis functions)

(2) `bas_gms2molpro h2o.inp -sph`  
Generate `h2o.com`, in which the keyword 'Cartesian' is not written (in order to use pure spherical harmonic type of basis functions).

(3) `bas_gms2molpro h2o.inp -sph -m15`  
Generate `h2o.com` for Molpro 2015. Only useful when you are using Molpro 2015 and dealing with ROHF/UHF high-spin wave functions.


## 4.5.10 bas_gms2psi
Generate a PSI4 `_psi.inp` file from a GAMESS .inp/.dat file. The Cartesian coordinates and basis set data are written in the `_psi.inp` file.
Two examples are shown and explained below

(1) `bas_gms2psi a.inp`  
Generate a_psi.inp file, in which the keyword 'cartesian' is written (in order to use pure Cartesian type of basis functions)

(2) `bas_gms2psi a.inp -sph`  
Generate a_psi.inp file, in which the keyword 'spherical' is written (in order to use pure spherical harmonic type of basis functions).


## 4.5.11 bas_gms2py
Generate a PySCF .py file from a GAMESS .inp/.dat file. The Cartesian coordinates and basis set data are written in the .py file.
Two examples are shown and explained below

(1) `bas_gms2py a.inp`  
Generate .inp file, in which the keyword 'mol.cart = True' is written (in order to use pure Cartesian type of basis functions)

(2) `bas_gms2py a.inp -sph`  
Generate .inp file, in which the keyword 'mol.cart = True' is not activated (in order to use pure spherical harmonic type of basis functions).


## 4.5.12 bdf2fch
Transfer MOs from BDF (.scforb/.inporb/.casorb, etc) to Gaussian .fch file. Two examples are shown and explained below

(1) `bdf2fch a.scforb a.fch`  
This is used for transferring R(O)HF, UHF or CASSCF orbitals.

(2) `bdf2fch a.casorb a.fch -no`  
This is used for transferring CASCI or CASSCF NOs and NOONs.

```admonish note
`bdf2fch` cannot generate a fch file from scratch, and a fch file must be provided and the MOs in it will be replaced. You should firstly use Gaussian to generate a .fch file (with keywords 'nosymm int=nobasistransform'), then generate the `*_bdf.inp` file using `fch2bdf`. After BDF computations finished, you can transfer MOs from .scforb/.casorb back to .fch file using `bdf2fch`. This procedure seems a little bit tedious, but it ensures an exact reproduce of energy.
```

This utility supports only spherical harmonic functions. To transfer MOs from Gaussian to BDF, see [fch2bdf](#4519-fch2bdf).


## 4.5.13 bdf2mkl
Transfer MOs from BDF (.scforb/.inporb/.casorb, etc) to ORCA. The ORCA .inp and .mkl file will be generated. This utility is actually a wrapper of two utilities `bdf2fch` and `fch2mkl`. Thus `bdf2mkl` cannot generate a .fch file from scratch, either. The user must provide a .fch(k) file, and MOs in that file will be replaced. Two examples are shown and explained below

(1) `bdf2mkl a.scforb a.fch`  
This is used for transferring RHF, ROHF, UHF or CASSCF orbitals.

(2) `bdf2mkl a.casorb a.fch -no`  
This is used for transferring CASCI or CASSCF NOs and NOONs.

NOTE: it is recommended to firstly use Gaussian to generate a .fch file (with keywords 'nosymm int=nobasistransform'), then generate the `_bdf.inp` file using `fch2bdf`. After BDF computations finished, you can transfer MOs from .scforb/.casorb of BDF to ORCA. This procedure seems a little bit tedious, but it ensures an exact reproduce of energy.


## 4.5.14 chk2gbw
Convert one (or more) .chk file(s) into .gbw file(s). The Gaussian utility formchk and ORCA utility orca_2mkl will be called automatically. Multiple chk files are supported. For example,

(1) `chk2gbw a.chk`  
Convert a.chk to a.gbw.

(2) `chk2gbw *.chk`  
Convert all chk files in the current directory into corresponding gbw files.


## 4.5.15 dal2fch
Transfer MOs from Dalton (i.e. DALTON.MOPUN file) to Gaussian .fch(k) file. Two examples are shown and explained below

(1) `dal2fch a.dat a.fch`  
This is used for transferring R(O)HF or CASSCF orbitals.

(2) `dal2fch a.dat a.fch -no`  
This is used for transferring CASCI/CASSCF natural orbitals and corresponding natural orbital occupation numbers. To transfer MOs from Gaussian to Dalton, see [fch2dal](#4522-fch2dal).

NOTE: `dal2fch` cannot generate a fch file from scratch, and a fch file must be provided and the MOs in it will be replaced. You should firstly use Gaussian to generate a .fch file (with keywords 'nosymm int=nobasistransform'), then generate the `*.dal` and `*.mol` files using `fch2dal`. After Dalton computations finished, you can transfer MOs from Dalton back to Gaussian using `dal2fch`. This procedure seems a little bit tedious, but it ensures an exact reproduce of energy.


## 4.5.16 dat2fch
Transfer MOs from GAMESS/Firefly (.inp/.dat file) to Gaussian .fch file. Five examples are shown and explained below

(1) `dat2fch a.dat`  
This is used for transferring R(O)HF, UHF or CASSCF orbitals. If the file `a.fch` exists, it will be used and MOs are written into that file. If `a.fch` does not exist, the utility `dat2fch` would try to create one such file from scratch. And this requires that the 1st line contains a `$CONTRL` section in file `a.dat`, so that charge and spin multiplicity can be read, e.g.
```
 $CONTRL SCFTYP=ROHF RUNTYP=ENERGY ICHARG=0 MULT=2 ISPHER=1 $END
```
Usually we have the corresponding .fch file in hand. Creating a .fch file from scratch is only recommended for users who cannot use the Gaussian software, and he/she may have performed the calculation using user-defined basis set in GAMESS/Firefly.

(2) `dat2fch a.dat b.fch`  
This is the same as (1), but transferring MOs to the specified file `b.fch`.

(3) `dat2fch a.dat a.fch -gvb 5`  
This is used for transferring GVB orbitals for spin singlet molecule. The order of GVB orbitals is different between Gaussian and GAMESS. Thus you must specify `-gvb [npair]` to tell the utility the number of GVB pairs, so that dat2fch can adjust the order of MOs.

(4) `dat2fch a.dat a.fch -gvb 5 -open 1`  
This is used for transferring GVB orbitals for non-singlet molecule. In this way, you tell the utility the number of GVB pairs and singly-occupied orbitals, so that dat2fch can adjust the order of MOs.

(5) `dat2fch a.dat a.fch -no 10`  
This is used for transferring natural orbitals (NOs) of CASCI/CASSCF. In this way, you tell `dat2fch` the number of NOs such that it would work correctly.

This utility supports two types of basis functions: (1) pure spherical harmonic functions; (2) pure Cartesian functions. But for creating a .fch file from scratch, only the former is supported currently. To transfer MOs from Gaussian to GAMESS/Firefly, see [fch2inp](#4523-fch2inp).


## 4.5.17 extract_noon2fch
Extract natural orbital occupation numbers (NOONs) from the following types of files  
(1) .out file of PySCF  
(2) .dat file of GAMESS  
(3) .gms file of GAMESS  
(4) .out file of ORCA  
(5) .out file of PSI4  
and write NOONs (as the 'Alpha Orbital Energies' section) into a given .fch file. This is for the convenience of visualizing orbitals using GaussView or Multiwfn+VMD.


## 4.5.18 fch2amo
Generate [Amesp](www.amesp.xyz) files (.aip and .amo) from a Gaussian .fch(k) file. Cartesian coordinates are written in the .aip file, while the basis set data is written both in the .aip and .amo files (in different formats). Both spherical harmonic type and Cartesian type basis functions are supported. An example is shown below
```
fch2amo h2o.fch
a2m h2o.amo
```

The first line will generate `h2o.aip` and `h2o.amo` files. `h2o.aip` is the input file of Amesp, and `h2o.amo` is the wave function file. The second line converts the ASCII text file `h2o.amo` into a binary file `h2o.mo`. This procedure is very similar to the file operations in Gaussian, where the correspondence is
```
h2o.gjf <-> h2o.aip
h2o.fch <-> h2o.amo
h2o.chk <-> h2o.mo
formchk <-> m2a
unfchk  <-> a2m
```

To transfer MOs from Amesp back to Gaussian, you can use the [amo2fch](#453-amo2fch) utility.

## 4.5.19 fch2bdf
Generate three BDF files (`_bdf.inp`, .BAS, .scforb/.inporb) from a Gaussian .fch(k) file. Cartesian coordinates are written in the `_bdf.inp` file, while the basis set data is held in .BAS file. The molecular orbitals are written in the .scforb/.inporb file. Note that BDF does not support Cartesian-type basis functions, so only spherical harmonic functions will be used. If there exists any Cartesian-type basis function in .fch(k) file, this utility will signal errors. Two examples are shown and explained below

(1) `fch2bdf a.fch`  
This is used for transferring RHF/ROHF/UHF orbitals. Three files will be generated: a_bdf.inp, A.BAS and a.scforb. The data in 'Alpha Orbital Energies' and 'Beta Orbital Energies' sections in .fch file will be read and printed into the a.scforb file.

(2) `fch2bdf a.fch -no`  
This is used for transferring NOs. Three files will be generated: a_bdf.inp, A.BAS and a.inporb. Note the data of 'Alpha Orbital Energies' section in .fch file (assumed occupation numbers) will be read and printed into the a.inporb file, but the occupation numbers do not affect subsequent computations.

This utility will call another two utilities `fch2inp` and `bas_gms2bdf`. So if you want to compile fch2bdf, you have to compile `fch2inp` and `bas_gms2bdf` additionally.

Note that to transfer HF orbitals, the data in 'Alpha Orbital Energies' and 'Beta Orbital Energies' section should be genuine orbital energies, since BDF program will use these values. Random values (like zero) will affect SCF computations and thus it cannot converge in 1 cycle (or even fails to converge). This is totally different with other quantum chemistry software packages where only orbitals are useful and orbital energies are useless. When transferring NOs, the occupation numbers in 'Alpha Orbital Energies' section do not affect subsequent computations.

To transfer MOs from BDF back to Gaussian, see [bdf2fch](#4512-bdf2fch).


## 4.5.20 fch2cfour
Generate CFOUR files (ZMAT, OLDMOS, GENBAS and ECPDATA, if ECP is used). One example is shown below
```
fch2cfour h2o.fch
```

This is used for transferring R(O)HF or UHF orbitals. One should first read the Cartesian coordinates from the CFOUR output and write a .gjf file to perform the single point calculation. Then use `fch2cfour` to generate CFOUR files, otherwise the molecular orientations of two programs will not be the same. Please use CFOUR v2.1 (or higher), since older versions of CFOUR (like v1.0 or v2.00beta) could not correctly read MOs from the file OLDMOS.


## 4.5.21 fch2com
Generate a Molpro .com file from a Gaussian .fch(k) file, with alpha MOs written in a `*.a` file. Two examples are shown below

(1) general use
```
fch2com h2o.fch
```
Files `h2o.com` and `h2o.a` would be generated. If UHF-type wave function is involved, `h2o.b` would also be generated. This is used for transferring R(O)HF, UHF or CASSCF orbitals.

(2) a special case
```
fch2com h2o.fch -m15
```
This is only useful when you are using Molpro 2015 and dealing with ROHF/UHF non-singlet wave function. Please do not use it for general purpose.

This utility will call another two utilities `fch2inp` and `bas_gms2molpro`. So if you want to compile fch2com, you have to compile `fch2inp` and `bas_gms2molpro` additionally.

Note that in Windows OS, any file with .com suffix/extension may be automatically associated with system, in which case double click of the mouse to open this file does not work. You have to right click on the .com file and choose 'open with'. You can modify the suffix/extension to .inp if you do not like that.

Note that if you use background charges in your studied system, the background charges are not recorded in the .fch(k) file. So there are no background charges in the generated .com file, either. To add background charges, you need to use the utility [add_bgcharge_to_inp](#451-add_bgcharge_to_inp).

To convert MOs from Molpro back to Gaussian, see [xml2fch](#4544-xml2fch).


## 4.5.22 fch2dal
Generate Dalton .dal and .mol files from a Gaussian .fch(k) file, where MOs are written in .dal file, and the Cartesian coordinates as well as basis set data are written in .mol file. One example is shown below
```
fch2dal a.fch
```
This is used for transferring R(O)HF or CASSCF orbitals. Note that there is no UHF method (or any methods based on UHF), thus you cannot use a .fch file containing UHF orbitals to transfer orbitals. To transfer MOs from Dalton back to Gaussian, see [dal2fch](#4515-dal2fch).


## 4.5.23 fch2inp
Generate a GAMESS .inp file from a Gaussian .fch(k) file, with Cartesian coordinates, basis set data and MOs written in. The keywords in .inp file is already suitable for common simple calculations, but do check or modify it if you have additional requirements.

Note that due to the different types of MOs (R(O)HF, UHF, GVB, and CASSCF orbitals), the fch2inp offers different options. Five examples are shown and explained below

(1) `fch2inp a.fch`  
This is used for transferring R(O)HF, UHF or CASSCF orbitals.

(2) `fch2inp a.fch -gvb 5`  
This is used for transferring GVB orbitals for spin singlet molecule. The order of GVB orbitals is different between Gaussian and GAMESS. Thus you must specify `-gvb [npair]` to tell the utility fch2inp the number of GVB pairs, so that fch2inp can adjust the order of MOs.

(3) `fch2inp a.fch -gvb 5 -open 1`  
This is used for transferring GVB orbitals for non-singlet molecule. In this way, you tell the utility fch2inp the number of GVB pairs and singly-occupied orbitals, so that fch2inp can adjust the order of MOs.

(4) `fch2inp high_spin.fch -sf`  
This is used to transfer RODFT/UDFT orbitals for the subsequent SF-TDDFT calculation in GAMESS. This spin multiplicity in high_spin.fch is supposed to be >=3, i.e. triplet or higher.

(5) `fch2inp triplet.fch -mrsf`  
This is used to transfer RODFT orbitals for the subsequent MRSF-TDDFT calculation in GAMESS. This spin multiplicity in triplet.fch is supposed to be 3.

This utility supports two types of basis functions: (1) pure spherical harmonic functions; (2) pure Cartesian functions. To transfer MOs from GAMESS back to Gaussian, see [dat2fch](#4516-dat2fch).

Note that if you use background charges in your studied system, the background charges are not recorded in the .fch(k) file. So there are no background charges in the generated .inp file, either. To add background charges, you need to use the utility [add_bgcharge_to_inp](#451-add_bgcharge_to_inp).


## 4.5.24 fch2inporb
Transfer MOs from Gaussian to (Open)Molcas. A .input file and a .INPORB file will be generated. The .input file is the input file of (Open)Molcas, and it contains the geometry, basis set data and keywords. The .INPORB file contains the MOs. Two examples are shown and explained below

(1) `fch2inporb a.fch`  
This is used for transferring R(O)HF, UHF or CASSCF orbitals.

(2) `fch2inporb a.fch -no`  
This is used for transferring NOs.

The option '-no' means reading NOONs and NOs from .fch(k) file and printing them into the .INPORB file. The NOONs written in the .INPORB file does not affect the calculations in OpenMolcas at all. It just make the file appear more readable.

This utility will call another two utilities `fch2inp` and `bas_gms2molcas`. So
if you want to compile `fch2inporb`, you have to compile `fch2inp` and `bas_gms2molcas`
additionally.

Note: if you use background charges in your studied system, the background charges are not recorded in the .fch(k) file. So there are no background charges in the generated .input file, either. To add background charges, you need to use the utility [add_bgcharge_to_inp](#451-add_bgcharge_to_inp).

Note: if you provide a .fch file generated by G09 and this .fch file is generated with DKH2 Hamiltonian in its corresponding .gjf file, the DKH2 information is not recorded in the .fch file. In such case, if you run fch2inporb xxx.fch, the generated xxx.input file will not include keyword Relativistic = R02O, you need to manually add this keyword into the .input file. This is not a problem for G16 since the Route section is recorded in .fch file for G16 so that it can be recognized/identified by fch2inporb.

To transfer MOs from (Open)Molcas back to Gaussian, see [orb2fch](#4540-orb2fch).


## 4.5.25 fch2mkl
Transfer MOs from Gaussian to ORCA. One example is shown below
```
fch2mkl h2o.fch
```
This is used for transferring R(O)HF/UHF/CASSCF orbitals. Two files will be generated: `h2o_o.inp` and `h2o_o.mkl`. The `_o` characters are added to avoid file overwritten in case that there is already a xxx.inp file in the current directory (for example, generated by the utility `fch2inp`). See more examples below:

**(1) Common DFT calculations**
```
fch2mkl h2o.fch -dft wB97M-V
fch2mkl h2o.fch -dft 'b3lyp d3bj'
fch2mkl h2o.fch -dft 'HSE06 D3zero'
```
If `h2o.fch` includes a DFT wave function, you can use the commands shown above to transfer orbitals. The density functional name is supposed to be provided by the user. If there is also dispersion correction to be added, they need to written together in single quotes '' or double quotation marks "".

**(2) Double hybrid functional calculations**  
Assuming you want firstly to perform the SCF part of the double hybrid functional PWPB95 in Gaussian using def2TZVPP, the Gaussian keywords are
```
#p PW91B95/def2TZVPP nosymm int(nobasistransform) IOp(3/76=1000005000,3/77=0000005000,3/78=0731007310)
```
And assuming next you want to perform a PWPB95-D3(BJ) job using ORCA, then you can run
```
fch2mkl h2o.fch -dft 'PWPB95 D3BJ'
```
There is no need to write the orbital basis set def2-TZVPP. The keywords in the generated file `h2o_o.inp` would be
```
! RKS PWPB95 D3BJ def2-TZVPP/C def2/J TightSCF noTRAH defgrid3
```

**(3) DLPNO double hybrid density functional calculations**  
Assume that you have performed a wB97X/def2TZVPP calculation using Gaussian, you can run
```
fch2mkl h2o.fch -dft 'DLPNO-wB97X-2 D3'
```
to generate the ORCA input file. Keywords are well written in the input file, e.g.
```
! RKS DLPNO-wB97X-2 D3 def2-TZVPP/C TightPNO def2/J TightSCF noTRAH defgrid3
```
Usually there is no need to modify the keywords above. But if you use def2QZVPP in Gaussian, you need to modify def2-TZVPP/C to def2-QZVPP/C in the ORCA input file. Similarly, if you have performed a B3LYP/def2TZVPP calculation using Gaussian, you can run
```
fch2mkl h2o.fch -dft 'DLPNO-B2PLYP D3BJ'
```
to generate the ORCA input file.

**(4) SF-TDDFT calculations**
```
fch2mkl O2_UDFT.fch -sf
```
Theoretically, the SF-TDDFT method can be either based on ROHF/ROKS, or based on UHF/UDFT reference. But ORCA only accepts the UHF/UDFT reference. So please provide a UHF/UDFT .fch file.

**(5) DLPNO-CCSD(T) calculations**  
Assuming you have perfored an RHF/cc-pVTZ job for H<sub>2</sub>O in Gaussian, and next you want to perform a DLPNO-CCSD(T)/cc-pVTZ computation in ORCA, then you can open the generated file `h2o_o.inp` and modify the keywords as
```
! RHF TightSCF DLPNO-CCSD(T) TightPNO RIJK cc-pVTZ/JK cc-pVTZ/C
```
The RHF calculation would be accelerated by the RIJK approximation. If you prefer RIJCOSX than RIJK, you can modify keywords as
```
! RHF TightSCF DLPNO-CCSD(T) TightPNO RIJCOSX defgrid3 cc-pVTZ/C
```

Again, there is no need to write the orbital basis set `cc-pVTZ` since the detailed basis set data was already written in the file `h2o_o.inp`. Of course, you need to modify the parallel and memory settings to suit your computer. If you want more accurate results, you can modify `DLPNO-CCSD(T)` to `DLPNO-CCSD(T1)`.

To convert MOs from ORCA to Gaussian, see [mkl2fch](#4537-mkl2fch) and [mkl2gjf](#4538-mkl2gjf). To convert MOs from ORCA to other software packages, you can use `mkl2amo`, `mkl2cfour`, `mkl2com`, `mkl2dal`, `mkl2inp`, `mkl2inporb`, `mkl2mrcc`, `mkl2psi`, `mkl2py`, and `mkl2qchem`.

**Note 1**: There is no need to write `IOp(3/32=2)` in Gaussian .gjf file. And it is always not recommended to write this keyword.

**Note 2**: Assuming your ORCA input file is `h2o_o.inp`, you need to run `orca_2mkl h2o_o -gbw` to generate the `h2o_o.gbw` file since ORCA cannot read `h2o_o.mkl` file directly.

**Note 3**: When calculating relative energies (e.g. the energy barrier), the computation results must share the same theory level. For example, the RIJCOSX approximation is used in all files.

Note that if you use background charges in your studied system, the background charges are not recorded in the .fch(k) file. So there are no background charges in the generated .inp or .mkl file, either. To add background charges, you need to use the utility [add_bgcharge_to_inp](#451-add_bgcharge_to_inp).


## 4.5.26 fch2mrcc
Convert MOs from Gaussian to [MRCC](https://mrcc.hu/index.php). For example,
```
fch2mrcc h2o.fch
```
Three files would be generated: `MINP`, `GENBAS`, and `MOCOEF`. If one has more than one .fch(k) file to be converted, the conversions should be performed in different directories such that generated files will not be overwritten by each other (Note: MRCC uses fixed filenames for these files).

To convert MOs from PySCF to MRCC, one can use the module `py2mrcc`, see [Section 4.6.5](./chap4-6.md#465-the-py2xxx-modules). To convert MOs from ORCA to MRCC, one can use the utility `mkl2mrcc`.


## 4.5.27 fch2psi
Convert MOs from Gaussian to PSI4. Two or three files will be generated: `*_psi.inp`, `*.A`, and `*.B` (if Beta MOs exist). The `_psi.inp` file of PSI4 holds the Cartesian coordinates, basis set data and keywords. The .A (and .B) file contains the Alpha (Beta) MOs. Two examples are shown below

**(1) For R(O)HF, UHF or CASSCF orbitals**
```
fch2psi a.fch
```

**(2) For DFT orbitals**
```
fch2psi a.fch -dft 'wB97M-D3BJ'
```
This will write keywords about the density functional wB97M-D3BJ into the generated input file `a_psi.inp`.

Please note that

(i) There is no need to write `IOp(3/32=2)` in Gaussian .gjf file. And it is always not recommended to write this keyword.

(ii) This utility will call another two utilities – `fch2inp` and `bas_gms2psi`, remember to compile them additionally.

(iii) If you use background point charges in your studied system, the point charges are not recorded in the .fch(k) file. So there is no point charge in the generated .inp file. To add background point charges, you need to use the utility [add_bgcharge_to_inp](#451-add_bgcharge_to_inp).

PSI4 can generate a .fch(k) file after calculation finished, which is equivalent to transferring MOs from PSI4 back to Gaussian.


## 4.5.28 fch2qchem
Transfer MOs from Gaussian to Q-Chem. For R(O)HF, UHF and DFT methods, two files will be generated: `.in` and `53.0`. The molecular orbitals are stored in the file `53.0`. For GVB, three files will be generated: `.in`, `53.0`, and `169.0`. The GVB orbitals are stored both in files `53.0` and `169.0` (the same content). Three examples are shown below

(1) `fch2qchem h2o.fch`  
This is used for transferring R(O)HF, UHF or CASSCF orbitals.

(2) `fch2qchem h2o.fch -gvb 2`  
This is used for transferring GVB orbitals. The orbitals in h2o.fch must be ordered in Gaussian GVB preference (bonding1, bonding2, anti-bonding2, anti-bonding1). And `-gvb 2` tells fch2qchem to write GVB-PP related keywords into the .in file.

(3) `fch2qchem high_spin.fch -sasf`  
This is used to transfer RODFT orbitals for the subsequent SA-SF-DFT calculation in Q-Chem. The spin multiplicity in high_spin.fch is supposed to be >=3, i.e. triplet or higher.

After running any of these commands, a Q-Chem input file `h2o.in` and a scratch directory `h2o` will be generated. The orbital file(s) is put in `h2o/`. Run
```
qchem h2o.in h2o.out h2o
```
to perform the subsequent Q-Chem job. If the environment variable `QCSCRATCH` is already defined in your node/computer, the scratch directory `h2o` will be automatically put into `$QCSCRATCH/`; otherwise it will be put in the current directory.
You can read [Q-Chem Manual](https://manual.q-chem.com/latest/ssect_general-usage.html) for more information.


## 4.5.29 fch2qm4d
Transfer MOs from Gaussian to QM4D. Input files for QM4D (.xyz, .inp, .xml and basis files for each element) will be generated. One example is shown below
```
fch2qm4d a.fch
```
This is used for transferring RHF or UHF orbitals.

Note 1: Currently QM4D supports only Cartesian-type basis sets. Thus this utility will signal error if you provide a .fch(k) file which has spherical harmonic functions (i.e. 5D 7F).

Note 2: QM4D supports ECP, but currently it seems to use spherical harmonic functions in ECP, so even if you specify `6D 10F` in Gaussian, the electronic energies of Gaussian v.s. QM4D will not be equal to each other. This tiny defect does not affect transferring MOs, and SCF can be converged in several cycles with almost no energy change.

Note that if you use background charges in your studied system, the background charges are not recorded in the .fch(k) file. So there are no background charges in the generated .inp file, either. To add background charges, you need to use the utility [add_bgcharge_to_inp](#451-add_bgcharge_to_inp).


## 4.5.30 fch2tm
Transfer MOs from Gaussian to Turbomole, i.e. generate files `control` and `mos` from a .fch(k) file. An example is shown below
```
fch2tm h2o.fch
```

Using this utility, you can skip the interactive module `define` of Turbomole, and obtain files `control` and `mos` directly. Default keywords written in `control` are for RHF/ROHF/UHF calculations. If you want to perform other calculations, further modification of `control` is needed. Currently only spherical harmonic functions are supported for this utility, so please do not use Cartesian-type functions (i.e. `6D 10F`) in the generation of .fch file.

To convert MOs from Turbomole back to Gaussian, see [molden2fch](#4539-molden2fch). Using utilities like [fch2qchem](#4528-fch2qchem), [fch2inporb](#4524-fch2inporb), [fch2com](#4521-fch2com), etc, you can convert MOs from Turbomole to many other quantum chemistry programs.


## 4.5.31 fch2wfn
Generate .wfn file from a specified .fch(k) file. Two examples are shown below

(1) `fch2wfn a.fch`  
For a .fch file in which ground state HF/DFT orbitals are recorded, generate the corresponding a.wfn file.

(2) `fch2wfn a.fch -no`  
For a .fch file in which natural orbitals are recorded, generate the corresponding a.wfn file.


## 4.5.32 fch_mo_copy
Copy MOs from one .fch file into another .fch file. An example is shown below
```
fch_mo_copy a.fch b.fch
```
The default is to copy Alpha MOs in a.fch to Alpha MOs in b.fch. There are 4 optional parameters `-aa`, `-ab`, `-ba`, `-bb`. For example, `-ab` means copying Alpha MOs from a.fch to Beta MOs in b.fch.


## 4.5.33 fch_u2r
Transform a UHF-type .fch file into a R(O)HF-type one. Only alpha MOs are retained.
```
fch_u2r h2o_uhf.fch
```

If you want to transform a R(O)HF-type .fch file into a UHF one, you need to use the Python API `fch_r2u`, e.g.
```python
from mokit.lib.rwwfn import fch_r2u
fch_r2u('h2o_rhf.fch')
```


## 4.5.34 frag_guess_wfn
This utility is designed to generate various EDA input files, e.g. GKS-EDA, SAPT, LMO-EDA and Morokuma-EDA. The Cartesian coordinates, basis set data, converged MO coefficients and necessary keywords will be written in the generated input file. See [examples](./chap5-3.html#53-examples-of-frag_guess_wfn).

The Morokuma-EDA is supported by the GAMESS-US program; the GKS-EDA and LMO-EDA are supported by the [XEDA](https://mp.weixin.qq.com/s/6nuJpYJdNbUJndrYM13Fog) program. These methods are very useful but often suffer from SCF convergence failure due to the "just so-so" SCF convergence techniques in GAMESS. Even if all SCF converge, their wave function stabilities cannot be checked in GAMESS.

The SAPT is supported by the [PSI4](https://mp.weixin.qq.com/s/I7Q1YXX5oSsXDe3oMo2jPw) program. When the molecule is large or the basis set includes diffuse functions, SCF often fails to converge.

This utility will call Gaussian to perform SCF calculations for each fragment (and maybe also for the complex), then write all MOs into the input file, so that all SCF steps during an EDA calculation will converge immediately.

If UHF/UDFT method is specified, wave function stability of each fragment (and of total system in GKS-EDA) will be checked. This is very important for biradical and transition-metal-containing systems. If any fragment is singlet and UHF/UDFT method is specified, broken symmetry initial guess (i.e., guess=mix) will be automatically applied.

If you want to use the Windows pre-built executable `frag_guess_wfn`, you need to define the environment variables of MOKIT and Gaussian. Assuming your input file `nh3_h2o.gjf` is put in `D:360Downloads\`, then you can run the following commands in CMD

```cmd
D:
cd 360Downloads

set PATH=D:\360Downloads\mokit\bin;%PATH%
set PATH=D:\Program files\G09W;%PATH%
set GAUSS_EXEDIR=D:\Program files\G09W

frag_guess_wfn nh3_h2o.gjf >nh3_h2o.out
```

If you use G16W rather than G09W, remember to modify paths showns above.


## 4.5.35 gvb_sort_pairs
Sort (part of) MOs in descending order of the pair coefficients of the 1st natural orbital in each pair. This utility is designed only for the GAMESS .dat file. A new .dat file will be generated, in which the sorted MOs and pair coefficients are held.


## 4.5.36 mkl2com
Transfer MOs from ORCA .mkl file into Molpro (.com, .a and possibly .b files). Two examples are shown and explained below

(1) `mkl2com h2o.mkl`  
This is used for transferring HF/DFT orbitals. `h2o.com` and `h2o.a` files will be generated. If the wavefucntion in .mkl is UHF-type, then `h2o.b` will also be generated.

(2) `mkl2com h2o.mkl water.com`  
Also used for transferring HF/DFT orbitals, but the input filename is specified by the user.

## 4.5.37 mkl2fch
Transfer MOs from ORCA .mkl file into Gaussian .fch(k) file. Four examples are shown and explained below

(1) `mkl2fch a.mkl`  
This is used for transferring HF/DFT orbitals. If the file `a.fch` exists, orbitals will be directly exported into it. If it does not exist, `mkl2fch` will try to create one such file from scratch.

(2) `mkl2fch a.mkl b.fch`  
Also used for transferring HF/DFT orbitals. But the filename `b.fch` is specified. `mkl2fch` will firstly try to find whether the file `b.fch` exists. If it exists, orbitals will be directly exported into it. If it does not exist, `mkl2fch` will try to create one such file from scratch.

(3) `mkl2fch a.mkl a.fch -no`  
This is used for transferring CAS NOs, MP2 NOs, CCSD NOs, etc.

(4) `mkl2fch a.mkl a.fch -nso`  
This is used for transferring Natural Spin Orbitals(NSO), e.g. UCCSD NSOs.

NOTE: If no .fch(k) file is provided, then `mkl2fch` will try to generate one from scratch. This is usually O.K. but remember that there is no ECP/PP information in .mkl file. So if you use ECP/PP, the generated .fch file will not include ECP/PP data, and you need to add them into .gjf file if you want to perform further calculations using Gaussian.

Note that the number of digits in .mkl file is only 7, and the scientific notation is not used. Therefore, the transferred MOs will not be very accurate. This should be sufficient for visualizing orbitals and common wavefunction amalysis, but may cause up to 1e-5 a.u. error on electronic energy in further computations. This can be viewed as a defect of the `orca_2mkl` utility in ORCA program. You are recommended to bring up an issue or request in the [ORCA forum](https://orcaforum.kofo.mpg.de), to suggest ORCA developers to fix this. If the scientific notation is not used, then 10 digits of MO coefficients should be sufficient for further computations.

You can also transfer MOs from ORCA to Gaussian via [mkl2gjf](#4538-mkl2gjf). To transfer MOs from Gaussian to ORCA, see [fch2mkl](#4525-fch2mkl).


## 4.5.38 mkl2gjf
Generate the Gaussian .gjf file from the ORCA .mkl file. The Cartesian coordinates, basis set data and MOs will be printed into the .gjf file. An example is shown and explained below

```
mkl2gjf a.mkl
```

this will generate the file `a.gjf`. Note that the ORCA .mkl file has a defect: it does not contain ECP/PP information. Therefore, if you did not use ECP in your ORCA calculations, there would be no problem. But in case that you used ECP, there would be no ECP data in the generated .gjf file (you must add them manually), and there would be a warning printed on the screen

```
Warning in subroutine mkl2gjf: element(s)>'Ar' detected.
NOTE: the .mkl file does not contain ECP/PP information. If you use ECP/PP
(in ORCA .inp file), there would be no ECP in the generated .gjf file. You
should manually add ECP data into .gjf, and change 'gen' into 'genecp'. If
you are using an all-electron basis set, there is no problem.
```

## 4.5.39 molden2fch
Convert a .molden file into a .fch file. Note that many quantum chemistry packages do not strictly obey the molden format standard, so a .molden file generated by different programs usually have different basis function order and positive/negative signs of MO coefficients. This utility takes care of these details one case by one.

**Example 1** If you have an ORCA wave function file `h2o.gbw`, you can run
```
orca_2mkl h2o -molden
molden2fch h2o.molden -orca
```
to convert a ORCA-type h2o.molden into h2o.fch.

**Example 2** If you have Turbomole files `control` and `mos`, you can run
```
tm2molden
h2o.molden
[Enter]
[Enter]
molden2fch h2o.molden -tm
```
convert a Turbomole-type h2o.molden into h2o.fch. Here `tm2molden` is a built-in utility of Turbomole. Note that there is no charge and spin multiplicity in the .molden file. The utility `molden2fch` will guess the charge and spin multiplicity and write them into the .fch file. Usually the guessing is right for HF/DFT jobs. But if you conducted complicated jobs, or stored natural orbitals in .molden, be careful about the charge and spin multiplicity in .fch. Currently only spherical harmonic functions are supported, so please do not use Cartesian-type basis functions in Turbomole (if you want to use `molden2fch`).

**Example 3** OpenMolcas
```
molden2fch h2o.molden -molcas
```

**Example 4** CFOUR
```
molden2fch h2o.molden -c4
```
Note that only the molden file generated by CFOUR v2.1 is supported currently, since different released versions of CFOUR program may have different conventions of molden.

**Example 5** CP2K
```
molden2fch h2o.molden -cp2k
```
convert a CP2K-type molden file into h2o.fch. A molden generated only by a gamma-point PBC calculation is supported. Becareful that lattice vectors and ECP/PP information are not recoreded in molden, so they would be missing in the .fch file, either. One can further use the `bas_fch2py` utility to generate a PySCF PBC gamma-point Python script, or use `load_cell_from_fch` to load a Cell object directly from the .fch file, which saves much time compared to writing a Python script from scratch. We are working on implementing better APIs for PBC calculations with less information missing of molden.

```admonish warning
.molden file does not include any ECP/PP data, so the generated .fch file would not include that data, either. If you use ECP/PP in your calculations, be careful about this problem.
```

To convert MOs from ORCA to Gaussian, also see [mkl2fch](#4537-mkl2fch) and [mkl2gjf](#4538-mkl2gjf). To convert MOs from Gaussian to Turbomole, see [fch2tm](#4530-fch2tm).


## 4.5.40 orb2fch
Transfer MOs from (Open)Molcas `*Orb` file (e.g. .ScfOrb, .RasOrb.1, .UnaOrb, .UhfOrb, etc) to Gaussian .fch(k) file. 5 examples are shown and explained below

(1) `orb2fch a.ScfOrb a.fch`  
This is used for transferring RHF orbitals.

(2) `orb2fch a.RasOrb a.fch`  
This is used for transferring CASCI/CASSCF or RASCI/RASSCF (pseudo)canonical orbitals.

(3) `orb2fch a.UnaOrb a.fch -no`  
This is used for transferring UNOs.

(4) `orb2fch a.UhfOrb a.fch`  
This is used for transferring UHF alpha and beta MOs.

(5) `orb2fch a.RasOrb.1 a.fch -no`  
This is used for transferring CASCI/CASSCF or RASCI/RASSCF NOs.

NOTE: `orb2fch` cannot generate a fch file from scratch, and a fch file must be provided and the MOs in it will be replaced. You should firstly use Gaussian to generate a .fch file (with keywords 'nosymm int=nobasistransform'), then generate the `*.input` and `*.INPORB` files using `fch2inporb`. After OpenMolcas computations finished, you can transfer MOs from OpenMolcas back to Gaussian using `orb2fch`. This procedure seems a little bit tedious, but it ensures an exact reproduce of energy.

To convert MOs from Gaussian to (Open)Molcas, see [fch2inporb](#4524-fch2inporb).


## 4.5.41 mo_svd
Output the singular values of the overlap matrix between two sets of MOs. This utility will first read the atomic overlap matrix from a given file, then calculate the overlap matrix between two sets of MOs. Finally, perform singular value decomposition (SVD) on the molecular orbital overlap matrix and print information about singular values. The first two command line arguments can be both Gaussian .fch files, or both OpenMolcas orbital files (.INBORB, .RasOrb, etc). The third argument is a Gaussian .log file or a OpenMolcas output file, in which the atomic overlap matrix is written.

The singular values can be used to measure the overlap or similarity of two sets of MOs. If all singular values are close to 1, the compared two sets of MOs are very similar. Any singular value being close to 0 means there exists at least 1 distinct orbital.


## 4.5.42 solve_ON_matrix
Compute the occupation number matrix (a non-diagonal matrix) of a set of MOs. Assuming you have a .fch(k) file which holds some kind of MOs, and another .fch(k) file which holds some kind of NOs and corresponding NOONs, then this utility can compute the occupation numbers of this set of MOs (transformed from NOs and NOONs). Of course, in this case the occupation number matrix of MOs is not a diagonal matrix, only the diagonal elements are written into the 'Alpha Orbital Energies' section of the file which holds MOs.


## 4.5.43 replace_xyz_in_inp
Replace the Cartesian coordinates in an input file, by coordinates from a given .xyz file or an output file. For the input/output file, currently only (Open)Molcas and Molpro formats are supported. Other formats will be supported in the near future. Four examples are shown below

(1) `replace_xyz_in_inp a.xyz b.input -molcas`  
Replace the Cartesian coordinates in b.input file, by coordinates from the a.xyz file. A file named b_new.input will be generated.

(2) `replace_xyz_in_inp a.out b.input -molcas`  
Replace the Cartesian coordinates in b.input file, by coordinates from an (Open)Molcas output file. A file named b_new.input will be generated.

(3) `replace_xyz_in_inp a.xyz b.com -molpro`  
Replace the Cartesian coordinates in b.com file, by coordinates from the a.xyz file. A file named b_new.com will be generated.

(4) `replace_xyz_in_inp a.out b.com -molpro`  
Replace the Cartesian coordinates in b.com file, by coordinates from a Molpro output file. A file named b_new.com will be generated.


## 4.5.44 xml2fch
Transfer MOs from Molpro .xml file to Gaussian .fch(k) file. 2 examples are shown and explained below

(1) `xml2fch a.xml a.fch`  
This is used for transferring R(O)HF or UHF orbitals.

(2) `xml2fch a.xml a.fch -no`  
This is used for transferring NOs.

NOTE: `xml2fch` cannot generate a fch file from scratch, and a fch file must be provided and the MOs in it will be replaced. You should firstly use Gaussian to generate a .fch file (with keywords 'nosymm int=nobasistransform'), then generate the `*.com` file using `fch2com`. After Molpro computations finished, you can transfer MOs from Molpro back to Gaussian using `xml2fch`. This procedure seems a little bit tedious, but it ensures an exact reproduce of energy.

To transfer MOs from Gaussian to Molpro, see [fch2com](#4521-fch2com).
<br/>
<br/>
<br/>
The modules/utilities below are compiled by `f2py`, which is a Fortran to Python interface generator. These utilities are not binary executable files, but dynamic libraries `*.so` in `$MOKIT_ROOT/mokit/lib`. They can only be imported in a Python script. And this is the reason that why one of the environment variables of MOKIT is `PYTHONPATH`, not `LD_LIBRARY_PATH`. These modules can also be viewed as APIs (Application Programming Interface), but they are described in this section not in [4.6](./chap4-6.html#46-apis-in-mokit), just for better comparison with other utilities of transferring MOs.


## 4.5.45 fch2py
Transfer MOs from Gaussian to PySCF. By importing fch2py, PySCF Python script is able to read alpha and/or beta MOs from a provided .fch file. All attributes are shown below

```python
fch2py(fchname, nbf, nif, ab, mf.mo_coeff)
```

There are 4 parameters required by fch2py: (1) filename of .fch(k) file; (2) the number of basis functions; (3) the number of molecular orbitals; and (4) a character 'a'/'b' for reading alpha/beta orbitals. You should write these contents into a Python script, and run that script using the python executable. If you want a more detailed example, just run any `automr` job and see all `*.py` files generated.

See relevant utilities/modules: [bas_fch2py](#454-bas_fch2py) and [load_mol_from_fch](./chap4-6.html#4611-load_mol_from_fch).


## 4.5.46 py2fch
Export MOs from PySCF to a Gaussian .fch file. Note that `py2fch` cannot generate a .fch file from scratch. The user must provide one .fch file, in which the 'Alpha Orbital Energies' and 'Alpha MOs' sections (or beta orbital counterpart) will be replaced by occupation numbers and MOs from PySCF. If you want to generate a .fch file from scratch, see the [fchk](#4548-fchk) module.

The provided .fch file must contain exactly the same geometry and basis set data as those data of PySCF. To obtain a valid .fch file, it is strongly recommended to use the `bas_fch2py` utility to generate a .py file from a .fch file first, and then `import py2fch` in this generated .py file. These descriptions seem a bit of tedious. But a rigorous transferring of MOs between two programs or two files is ensured. All attributes of `py2fch` are shown below

```python
py2fch('a.fch', nbf, nif, mf.mo_coeff, ab, ev, gen_density)
```

The first few parameters are identical to those in [fch2py](#4545-fch2py). The parameter `ev` means eigenvalues, it is supposed to contain orbital energies or orbital occupation numbers. The last parameter `gen_density` is a bool variable, where True/False meaning whether or not generating Total SCF Density (as well as writing density into .fch file) using mf.mo_coeff and ev. If `gen_density=True`, the parameter `ev` must contain orbital occupation numbers, not orbital energies. Because density would be computed using MOs and occupation numbers. If `gen_density=False`, you can assign proper values to `ev` as your wish.

This module is usually used along with these utilities/modules: [bas_fch2py](#454-bas_fch2py), [fch2py](#4545-fch2py) and [load_mol_from_fch](./chap4-6.html#4611-load_mol_from_fch).


## 4.5.47 py2fch_cghf
Export complex gHF MOs from PySCF to a Gaussian .fch file. Note that `py2fch_cghf` cannot generate a .fch file from scratch. The user must provide one .fch file. This module is similar to `py2fch` (which is usually used for R(O)HF, UHF and DFT methods). The difference is that this module is only designed for complex gHF/gDFT methods. All attributes of `py2fch_cghf` are shown below

```python
py2fch_cghf(fchname, nbf, nif, coeff, ev, gen_density)
```


## 4.5.48 fchk
This is a powerful module since it can directly export/generated a Gaussian .fch(k) file (i.e. no need to provide one .fch file in advance). And this module is the basis of many other modules like `py2molpro`, `py2molcas`, etc, see [Section 4.6.5](./chap4-6.md#465-the-py2xxx-modules). A few examples are shown below:

(1) transfer HF/DFT MOs from PySCF to Gaussian
```python
from pyscf import gto
from mokit.lib.py2fch_direct import fchk

mol = gto.M(atom='O 0.0 0.0 0.1; H 0.0 0.0 1.0',
        basis='cc-pvtz',
        charge=-1,
        ).build()
mf = mol.RHF().run()
fchk(mf, 'test.fch')
```

**Options**
* To write the HF/DFT Total SCF Density into the generated fch file, you should use the `density` argument
```python
fchk(mf, 'test.fch', density=True)
```
* To specify another orbital coefficient (instead of `mf.mo_coeff`), use
```python
fchk(mf, 'test.fch', mo_coeff=some_other_mo)
```

```admonish note
After MOKIT version 1.2.6rc5, a lazy import of this function is enabled. `from mokit.lib.py2fch_direct import fchk` can be replaced by `from mokit.lib import *` or `from mokit.lib import fchk`.
```

(2) transfer CASSCF MOs from PySCF to Gaussian
```python
from pyscf import gto, mcscf
from mokit.lib.py2fch_direct import fchk

mol = gto.M(
    atom = 'O 0 0 0; O 0 0 1.2',
    basis = 'ccpvdz',
    spin = 2)

mf = mol.RHF().run()
mc = mcscf.CASSCF(mf, 6, 8).run() #CAS(6o,8e)
fchk(mc, 'O2_cas6o8e.fch')
```

(3) transfer CASSCF NOs from PySCF to Gaussian
```python
from pyscf import gto, mcscf
from mokit.lib.py2fch_direct import fchk

mol = gto.M(
    atom = 'O 0 0 0; O 0 0 1.2',
    basis = 'ccpvdz',
    spin = 2)

mf = mol.RHF().run()
mc = mcscf.CASSCF(mf, 6, 8) #CAS(6o,8e)
mc.natorb = True
mc.kernel()
fchk(mc, 'O2_cas6o8e_NO.fch', density=True)
```
By default, `fchk` will dump `mc.mo_coeff` and `mc.mo_occ` (if present) into the 'Alpha MO' and 'Alpha Orbital Energies' section in .fch file. It's also possible to manually dump any orbitals you want 
```python
fchk(mc, 'O2_cas6o8e_some_other_orb.fch', mo_coeff=some_other_mo, mo_occ=some_other_occ)
```

(4) write UNOs into fch

By default, the `fchk(mf, fchname)` for UHF/UKS object dumps the orbital energies which prevents we writing the UNO occupation numbers. So another function `fchk_uno` can be used
```python
from pyscf import gto, mcscf
from mokit.lib.py2fch_direct import fchk_uno

mol = gto.M(
    atom = 'O 0 0 0; O 0 0 1.2',
    basis = 'ccpvdz',
    spin = 2)

mf = mol.UHF().run()
unoon, uno = mcscf.addons.make_natural_orbitals(mf)
fchk_uno(mf, 'O2_UNO.fch', uno, unoon)
```

There is a module named `fchk()` in PSI4, which is used to export/generated a .fch file. Here we use the same name for easy memorizing. You can also use its alias `py2gau`, e.g.
```python
py2gau(mf, 'test.fch')
```
if you think `py2gau` is easier for you to remember.

