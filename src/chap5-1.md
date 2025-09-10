# 5.1 Examples of `automr`

## 5.1.1 Single point calculation of the ground state
The input file of `automr` is just the Gaussian .gjf file. For example, the ground state CASSCF calculation of a stretched molecule is shown as follows (h2o.gjf)
```
%mem=8GB
%nprocshared=4
#p CASSCF/def2TZVP

mokit{}

0 1
O      -0.23497692    0.90193619   -0.068688
H       1.26502308    0.90193619   -0.068688
H      -0.73568721    2.31589843   -0.068688

```

Submit the job
```
automr h2o.gjf >h2o.out 2>&1 &
```

After the calculation is accomplished, you can find in the output file that an active space CAS(4,4) is automatically determined. And the CASSCF natural orbitals (NOs) can be found in `h2o_uhf_gvb4_CASSCF_NO.fch`. The calculation will also print RHF, UHF, GVB and CASCI electronic energies since they are supposed to be intermediate steps in a CASSCF job.

The GVB NOs can be found in `h2o_uhf_uno_asrot2gvb4_s.fch`, in which the GVB active space contains 4 pairs of orbitals (O 2s lone pair, O 2p lone pair, and two O-H bonding orbitals as well as two O-H anti-bonding orbitals). The two pairs of O-H orbitals are of significant multireference characters, so they are automatically chosen as the active space for the further CASSCF(4,4) calculation. That is to say, by default the important GVB orbitals are used as the initial guess orbitals of CASCI/CASSCF calculations. The threshold for determining important GVB orbitals is *n*<sub>i</sub> >= 0.02, where *n*<sub>i</sub> is the occupation number of the 2nd natural orbital in a GVB pair.

If you want to use other methods instead of CASSCF, just replace the method name as GVB, CASCI, CASPT2, NEVPT2, MRMP2, MRCISD, MCPDFT, etc. To see all methods supported by `automr`, run `automr -h`.

If you want to use another quantum chemistry program for the CASSCF calculation (say Molpro), you need to write `mokit{CASSCF_prog=Molpro}`. Similarly, you can add 'GVB_prog=QChem' to call Q-Chem to perform the GVB calculation if you wish (By default, GVB_prog=GAMESS). For all programs supported, just run `automr -h`.

## 5.1.2 Single point calculation of an excited state
If you are interested only in one or two low-lying excited state, then the State-Specific CASSCF (SS-CASSCF) method is recommended. For example, the input file of optimizing the orbitals of the CASSCF S<sub>1</sub> state of a water molecule is
```
%mem=8GB
%nprocshared=4
#p CASSCF/def2TZVP

mokit{root=1}

0 1
O      -0.23497692    0.90193619   -0.068688
H       1.26502308    0.90193619   -0.068688
H      -0.73568721    2.31589843   -0.068688

```

where `root=1` means the first excited state which has the same spin as the spin of the ground state. Several electronic states will be calculated and the S<sub>1</sub> state can be automatically tracked by `automr`, via checking the spin of each state. To calculate the S<sub>2</sub>  state, just specify `mokit{root=2}`.

For T<sub>1</sub> state, you need to write `mokit{root=1,Xmult=3}`, where `Xmult` means the spin multiplicity of the target excited state. Of course, we know that the T<sub>1</sub> state can be calculated in an easier way: we can just perform a ground state calculation with spin mmultiplicity 3. So this functionality is useful for S<sub>1</sub>, S<sub>2</sub>, T<sub>2</sub> or higher states.

## 5.1.3 Single point calculation of several excited states
If you are interested in many excited states, or there exists energy degeneracy among your interested states, then the State-Averaged CASSCF (SA-CASSCF) method is recommended. This method calculates several electronic states in a single shot. An example is given below

```
%mem=16GB
%nprocshared=8
#p CASSCF/def2TZVP

mokit{ist=5,readno='h2o_uhf_gvb4_CASSCF_NO.fch',nstates=3}
```

It is recommended to perform the ground state CASSCF calculation firstly, then use the CASSCF .fch file (which includes CASSCF orbitals) to perform the excited state calculations. The .fch file can be either obtained by a previous MOKIT calculation (recommended) or provided by an experienced user. Here `nstates=3` means an average of S<sub>0</sub>, S<sub>1</sub>, S<sub>2</sub> and S<sub>3</sub> states. To mix electronic states with different spins, you need `mokit{...,nstates=3,Mix_Spin}`. The output of the input file above would be like

```
CASCI energies after SA-CASSCF(0 for ground state):   E_ex/eV   fosc
State   0, E =    -75.91837033 a.u. <S**2> = 0.000
State   1, E =    -75.68245998 a.u. <S**2> = 0.000    6.419   0.0005
State   2, E =    -75.65490596 a.u. <S**2> = 0.000    7.169   0.0158
State   3, E =    -75.59084787 a.u. <S**2> = 0.000    8.912   0.0001
```

where the `fosc` is the corresponding oscillator strength of transitions. Besides, hole NTOs and particle NTOs files are generated and stored in .fch files:

```
h2o_uhf_gvb4_CASSCF_NO_SA-CAS_NTO_H01.fch
h2o_uhf_gvb4_CASSCF_NO_SA-CAS_NTO_H02.fch
h2o_uhf_gvb4_CASSCF_NO_SA-CAS_NTO_H03.fch
h2o_uhf_gvb4_CASSCF_NO_SA-CAS_NTO_P01.fch
h2o_uhf_gvb4_CASSCF_NO_SA-CAS_NTO_P02.fch
h2o_uhf_gvb4_CASSCF_NO_SA-CAS_NTO_P03.fch
```

where `H01` means the hole NTOs of S<sub>0</sub> -> S<sub>1</sub> transition, and `P01` means the particle NTOs of S<sub>0</sub> -> S<sub>1</sub> transition, respectively. You can use GaussView or Multiwfn+VMD to visualize the NTOs and corresponding eigenvalues.

The default SA-CASSCF program called by MOKIT is PySCF. OpenMolcas can be an alternative choice if you like (i.e. `CASSCF_prog=OpenMolcas`). To obtain the dynamic correlation, you can change `CASSCF` to `NEVPT2`. NEVPT2 will be performed based on each CASCI state obtained after the SA-CASSCF calculation. More advanced methods like QD-NEVPT2 and XMS-CASPT2 interfaces will be supported in the near future.

## 5.1.4 MRCISD+Q calculation of the ground state
The MRCISD calculations using `automr` are worthy of additional explanations. There are at least 3 variants of MRCISD:  

(1) uncontracted MRCISD (unc-MRCISD);  
(2) internally contracted MRCISD (ic-MRCISD);  
(3) fully internally contracted MRCISD (FIC-MRCISD).

All these variants are not size-consistent, therefore, the Davidson size-consistency correction is usually calculated and added into the total electronic energy (e.g. termed as FIC-MRCISD+Q). The computational cost and accuracy is unc-MRCISD > ic-MRCISD > FIC-MRCISD.

The unc-MRCISD is very expensive and usually used for benchmark of small active space of small molecules. The ic-MRCISD+Q and FIC-MRCISD+Q variants are commonly used for routine MRCISD calculations. Some input examples are shown below

(1) **unc-MRCISD+Q**
```
%mem=20GB
%nprocshared=4
#p MRCISD/TZVP

mokit{CtrType=1}

0 1
O      -0.23497692    0.90193619   -0.068688
H       1.26502308    0.90193619   -0.068688
H      -0.73568721    2.31589843   -0.068688

```

This example will triggers UHF, GVB(4), CASSCF(4,4) and MRCISD+Q based on CAS(4,4) calculations sequentially. By default, `MRCISD_prog` is OpenMolcas for unc-MRCISD and thus there is no need to write `MRCISD_prog=OpenMolcas`. The `+Q` correction energy is calculated by `automr` and you can find related energies in the output of `automr`
```
Davidson correction=       -0.00797009 a.u.
E_corr(MRCISD) =       -0.17768879 a.u.
E(MRCISD+Q)    =      -76.11894741 a.u.
```

If you want to use another `MRCISD_prog`, you can choose ORCA/GAMESS/PSI4/Gaussian/Dalton. But currently for Gaussian and Dalton, `automr` cannot provide the `+Q` correction energy. Note that you need to install HDF5-enabled OpenMolcas since `automr` need to read the .h5 file in order to calculate the `+Q` energy.

(2) **ic-MRCISD+Q**
```
%mem=20GB
%nprocshared=4
#p MRCISD/TZVP

mokit{CtrType=2}

0 1
O      -0.23497692    0.90193619   -0.068688
H       1.26502308    0.90193619   -0.068688
H      -0.73568721    2.31589843   -0.068688

```

By default, `MRCISD_prog` is also OpenMolcas for ic-MRCISD+Q. You can also use Molpro.

(3) **FIC-MRCISD+Q**
```
%mem=20GB
%nprocshared=4
#p MRCISD/TZVP

mokit{CtrType=3,MRCISD_prog=ORCA}

0 1
O      -0.23497692    0.90193619   -0.068688
H       1.26502308    0.90193619   -0.068688
H      -0.73568721    2.31589843   -0.068688

```
For FIC-MRCISD+Q, the only option of `MRCISD_prog` is ORCA.

The definition of size-consistency correction is not unique, and `automr` adopts the simplest one: \\( E_{+Q} = E_{corr}(1-{c_0}^{2}) \\). If you are interested in other types of size-consistency correction, you are referred to [this paper](https://doi.org/10.1039/C2CP23757A). If you find your result is sensitive to the type of size-consistency correction, maybe MRCISD+Q is not appropriate for your molecule, and you may want to try more advanced methods like FIC-MRCCSD.

## 5.1.4 Acceleration techniques
To speed up the calculation for a large molecule and/or a large basis set, one can add `RI` to enable the Resolution-of-Identity (RI) techniques whenever possible. For example,
```
%mem=16GB
%nprocshared=16
#p CASSCF(6,6)/cc-pVTZ

mokit{RI}

0 1
[Cartesian coordinates of benzene]
```
The RI-JK will be enabled in the CASSCF calculation and the auxiliary basis set are automatically dealt with.

To perform the NEVPT2 calculation for a large molecule, you can use the local correlation method DLPNO-NEVPT2 in ORCA, e.g.
```
%mem=96GB
%nprocshared=48
#p NEVPT2(6,6)/cc-pVTZ

mokit{CASSCF_prog=ORCA,NEVPT_prog=ORCA,DLPNO}
...
```

Note that `RI` is useless for DMRG related calculations.

