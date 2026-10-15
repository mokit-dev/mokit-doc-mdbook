# 4.8 List of autosr Keywords
There are quite a few keywords that do the same or similar thing here as in `automr`, as follows

`readrhf`, `readuhf`, `Cart`, `DKH2`, `X2C`, `charge`

Search them in [Section 4.4](./chap4-4.md) for their usage. Note that the keyword `ist` cannot be used here.

## 4.8.1 HF_prog
The quantum chemistry program for RHF/UHF calculations. Supported programs are Gaussian/PySCF/PSI4/ORCA.

## 4.8.2 MP2_prog
The quantum chemistry program for MP2 calculations. Supported programs are Molpro/Gaussian/PySCF/PSI4/ORCA/QChem/OpenMolcas.

## 4.8.3 CC_prog
The quantum chemistry program for CC calculations. Supported programs are Molpro/Gaussian/PySCF/PSI4/ORCA/QChem/OpenMolcas.

## 4.8.4 ADC_prog
The quantum chemistry program for ADC(2)/ADC(3) calculations.

## 4.8.5 EOM_prog
The quantum chemistry program for EOM-CCSD, IP-EOM-CCSD, EA-EOM-CCSD calculations. Supported programs are Molpro/Gaussian/PSI4/ORCA/QChem.

## 4.8.6 Nstates
The number of excited states. `Nstates=1` means the ground state plus the first excited state.

## 4.8.7 RI settings

### 4.8.7.1 NoRI
Request to turn off the RI acceleration technique.

### 4.8.7.2 RIJK_bas
Specify the RI-JK auxiliary basis set.

### 4.8.7.3 RIC_bas
Specify the RI-C auxiliary basis set.

## 4.8.8 F12
To be documented.

## 4.8.9 F12_cabs
To be documented.

## 4.8.10 DLPNO
Request to turn on the DLPNO local correlation approximation in MP2/CCSD/CCSD(T) calculations.

## 4.8.11 NO
This keyword requests the generation of Natural Orbitals(NOs) in a post-HF calculation. The NOs will be stored in a file like `xxx_NO.fch`, which can be visualzied using GaussView/Multiwfn/Qbics-Molstar. By default NOs will not be generated. If you want NOs, do not write `mokit{NO=True}` but simply write `mokit{NO}`. See the next keyword for using relaxed or unrelaxed density to generate NOs.

## 4.8.12 RelaxedDM
Generate NOs using the relaxed density. If this keyword is not specified, unrelaxed density will be calculated and used.

## 4.8.13 Force
This keyword requests the calculation of analytical nuclear gradients.

## 4.8.14 FC
Control the number of frozen core orbitals in a post-HF calculation. By default, the number of frozen core orbitals is equal to settings in ORCA manual `9.11 Frozen Core Options`. For example, if we perform a post-HF calculation for a water molecule, `FC=1` will be automatically set and the user does not need to take care of this. If you want no frozen core, write `mokit{FC=0}`.

