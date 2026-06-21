## 3.1 Quick Start with Examples

### 3.1.1 Automatic multiconfigurational computations
The input syntax of the `automr` program is the same to Gaussian gjf. For example, the input file `00-h2o_cc-pVDZ_1.5.gjf` of the water molecule at d(O-H) = 1.5 A is shown below
```
%mem=4GB
%nprocshared=4
#p CASSCF/cc-pVDZ

mokit{}

0 1
O      -0.23497692    0.90193619   -0.068688
H       1.26502308    0.90193619   -0.068688
H      -0.73568721    2.31589843   -0.068688
```

Run the following command
```
automr 00-h2o_cc-pVDZ_1.5.gjf >00-h2o_cc-pVDZ_1.5.out 2>&1
```
The `automr` program will successively perform HF, GVB, and CASSCF computations by calling Gaussian, GAMESS and PySCF, respectively. The active space will be automatically determined as (4,4) during computations. Detailed instructions for `automr` input can be found in [Section 4.1 - 4.4](./chap4-1.md).

See more examples in [Section 5.1](./chap5-1.md#51-examples-of-automr).

### 3.1.2 Automatic multireference computations
The `automr` program supports automatic computation of many multireference methods, e.g. NEVPT2/CASPT2/MRCISD/MC-PDFT.
```
%mem=8GB
%nprocshared=4
#p NEVPT2/cc-pVTZ

mokit{}

0 1
O      -0.23497692    0.90193619   -0.068688
H       1.26502308    0.90193619   -0.068688
H      -0.73568721    2.31589843   -0.068688
```

### 3.1.3 Automatic single-reference computations
There are many factors to be considered in some advanced single-reference computations: the auxiliary basis set for RIJK, the auxiliary basis set for correlated methods, the complementary auxiliary basis sets for F12 calculations, SCF convergence, HF wave function stability, etc. These factors can be automatically dealt with by using `autosr`. For example,
```
%mem=8GB
%nprocshared=4
#p CCSD(T)-F12/cc-pVTZ-F12

mokit{CC_prog=ORCA}

0 1
O         0.00000000     0.00000000     0.06200700
H         0.00000000    -0.78397600    -0.49205200
H         0.00000000     0.78397600    -0.49205200

```

Run the following command to submit the `autosr` job.
```
autosr h2o.gjf >h2o.out 2>&1 &
```

`autosr` will call Gaussian to perform HF/cc-pVTZ-F12 calculations, transfer MOs to ORCA, and call ORCA to perform the CCSD(T)-F12/cc-pVTZ-F12 calculation. cc-pVTZ-F12 is not a built-in basis set in Gaussian, but `autosr` will deal with this problem automatically since MOKIT has included this basis set.


### 3.1.4 Using one utility
Running the following command
```
fch2inp h2o.fch
```
will generate the GAMESS input file `h2o.inp`, in which the Cartesian coordinates, basis set data, molecular orbitals(MOs) and some necessary keywords are written. If the MOs in `h2o.fch` are RHF/ROHF/UHF MOs, you can simply submit the GAMESS job and you will find the SCF is converged in ~1 cycle in GAMESS. If you want to perform other types of calculation, remember to modify `h2o.inp`. Note that keywords `nosymm int=nobasistransform` should be written in `h2o.gjf` before submitting the Gaussian job, since this facilitates the orbital transferring and SCF convergence.

Some commonly used utilities and their functionalities are listed below

| Utility name | Functionality |
| --- | --- |
| fch2amo | Gaussian -> AMESP |
| fch2bdf | Gaussian -> BDF |
| fch2cfour | Gaussian -> CFOUR |
| fch2com | Gaussian -> Molpro |
| fch2dal | Gaussian -> Dalton |
| fch2inp | Gaussian -> GAMESS |
| fch2inporb | Gaussian -> (Open)Molcas |
| fch2mkl | Gaussian -> ORCA |
| fch2mrcc | Gaussian -> MRCC |
| fch2nwc | Gaussian -> NWChem |
| fch2openqp | Gaussian -> OpenQP |
| fch2psi | Gaussian -> PSI4 |
| fch2qchem | Gaussian -> Q-Chem |
| fch2qm4d | Gaussian -> QM4D |
| fch2rest | Gaussian -> REST |
| fch2tm | Gaussian -> Turbomole |
| mkl2fch | ORCA -> Gaussian |
| molden2fch | others -> Gaussian |

You can search a utility and read the documentations in Section [4.5](./chap4-5.md) or [4.6](./chap4-6.md).

