# 4.7 Supported methods in `autosr`
Currently, the keywords of all supported methods in `autosr` are:

[MP2](#471-mp2), [CCD](#472-ccd), [CCSD](#473-ccsd), [CCSD(T)](#474-ccsd(t)), [CCSDT](#475-ccsdt),

[CCSD-F12](476-ccsd-f12), [CCSD(T)-F12](477-ccsd(t)-f12),

[EOM-CCSD](478-eom-ccsd), [EOM-IP-CCSD](479-eom-ip-ccsd), [EOM-EA-CCSD](4710-eom-ip-ccsd),

[CC2](4711-cc2), [CC3](4712-cc3),

[ADC(2)](4713-adc(2)), [ADC(3)](4714-adc(3)),

[QCISD](#4715-qcisd), [QCISD(T)](#4716-qcisd(t))

## 4.7.1 MP2
Second-order Møller-Plesset perturbation theory. An example is shown below
```
%mem=4GB
%nprocshared=2
#p MP2/cc-pVTZ

mokit{noRI}

0 1
O     0.000000    0.000000    0.062007
H     0.000000   -0.783976   -0.492052
H     0.000000    0.783976   -0.492052
```

Different keywords in the Title Card Line can be applied:

| cases | keywords | supported programs |
| --- | --- | --- |
| MP2 energy | mokit{noRI} | all |
| MP2 NO using unrelaxed density | mokit{noRI,NO} | TBD |
| MP2 NO using relaxed density | mokit{noRI,NO,RelaxedDM} | TBD |
| MP2 analytical gradients | mokit{noRI,force} | TBD |
| RI-MP2 energy | mokit{} | TBD |
| RI-MP2 NO using unrelaxed density | mokit{NO} | TBD |
| RI-MP2 NO using relaxed density | mokit{NO,RelaxedDM} | TBD |
| RI-MP2 analytical gradients | mokit{force} | TBD |
| DLPNO-MP2 energy | mokit{DLPNO} | ORCA/PSI4 |
| DLPNO-MP2 NO using unrelaxed density | mokit{DLPNO,NO} | TBD |
| DLPNO-MP2 NO using relaxed density | mokit{DLPNO,NO,RelaxedDM} | TBD |
| DLPNO-MP2 analytical gradients | mokit{DLPNO,force} | TBD |

(TBD: to be determined)

The local correlation method DLPNO-MP2 is supported in ORCA/PSI4, and it can be invoked by
```
#p MP2/def2TZVPP

mokit{DLPNO,MP2_prog=ORCA}
```
or by
```
#p DLPNO-MP2/def2TZVPP

mokit{MP2_prog=ORCA}
```
Please read related keyword MP2_prog.

## 4.7.2 CCD
Coupled cluster doubles.

Please read related keyword CC_prog.

| feature | keywords | supported programs |
| --- | --- | --- |
| CCD energy | mokit{noRI} | all |
| CCD unrelaxed DM | mokit{noRI,NO} | TBD |
| CCD relaxed DM | mokit{noRI,NO,RelaxedDM} | TBD |
| CCD analytical gradients | mokit{noRI,force} | TBD |
| DF-CCD energy | mokit{} | TBD |
| DF-CCD unrelaxed DM | mokit{NO} | TBD |
| DF-CCD relaxed DM | mokit{NO,RelaxedDM} | TBD |
| DF-CCD analytical gradients | mokit{force}| TBD |

## 4.7.3 CCSD
Coupled cluster singles and doubles.

The CCSD nonlinear equations will be solved to obtain the t1 and t2 amplitudes. The T1-diagnostic value will be printed as well.

By default, the MP2 t2 amplitudes are used as the initial guess of a CCSD calculation. For `CC_prog=PySCF`, if the [STC-CC](https://github.com/SUSYUSTC/stc_qc_paper) package is installed, the STC-RI-CCSD amplitudes will be used as the initial guess of CCSD; and if there are converged t1, t2 amplitudes generated from a previous calculation (saved in .npy files), they will be read and the CCSD will be converged in 1 cycle.

The local correlation method DLPNO-CCSD is supported in ORCA, and it can be invoked by
```
#p CCSD/def2TZVPP

mokit{DLPNO,CC_prog=ORCA}
```
or by
```
#p DLPNO-CCSD/def2TZVPP

mokit{CC_prog=ORCA}
```
Please read related keyword CC_prog.

| feature | keywords | supported programs |
| --- | --- | --- |
| CCSD energy | mokit{noRI} | all |
| CCSD unrelaxed DM | mokit{noRI,NO} | Molpro/PySCF/PSI4/ORCA/QChem |
| CCSD relaxed DM | mokit{noRI,NO,RelaxedDM} | Molpro/Gaussian/PSI4/QChem |
| CCSD analytical gradients | mokit{noRI,force} | Molpro/PySCF/Gaussian/PSI4/QChem |
| DF-CCSD energy | mokit{} | TBD |
| DF-CCSD unrelaxed DM | mokit{NO} | TBD |
| DF-CCSD relaxed DM | mokit{NO,RelaxedDM} | TBD |
| DF-CCSD analytical gradients | mokit{force}| TBD |
| DLPNO-CCSD energy | mokit{DLPNO} | TBD |
| DLPNO-CCSD unrelaxed DM | mokit{DLPNO,NO} | TBD |
| DLPNO-CCSD relaxed DM | mokit{DLPNO,NO,RelaxedDM} | TBD |
| DLPNO-CCSD analytical gradients | mokit{DLPNO,force}| TBD |

## 4.7.4 CCSD(T)
Coupled cluster singles, doubles and perturbative triples.

The CCSD nonlinear equations will be solved to obtain the t1 and t2 amplitudes, then t3 amplitudes are directly calculated via perturbation theory (using electron integrals, t1 and t2 amplitudes). The T1-diagnostic value will be printed as well.

The local correlation method DLPNO-CCSD(T) is supported in ORCA, and it can be invoked by
```
#p CCSD(T)/def2TZVPP

mokit{DLPNO,CC_prog=ORCA}
```
or by
```
#p DLPNO-CCSD(T)/def2TZVPP

mokit{CC_prog=ORCA}
```

DLPNO-CCSD(T) employs localized MOs in correlation energy calculation, rather than canonical MOs. So the perturbative triples (i.e. t3 amplitudes) cannot be directly calculated using t1 and t2. In principle, t3 amplitudes must be solved in a iterative way (by solving a set of linear equations). This "iterative" way is different from the iteration in the CCSDT method. For the former, the t1/t2 amplutides are solved in (DLPNO-)CCSD and fixed in t3 computation; for the latter, the t1/t2/t3 amplutides are solved together in a set of nonlinear equations.

By default, when the user requests DLPNO-CCSD(T) using two input formats above, the DLPNO-CCSD(T0) method is used, i.e. the perturbative triples are directly calculated as an approximation. If more accurate results are desired, it is strongly recommended to use DLPNO-CCSD(T1), where the perturbative t3 amplitudes are solved iteratively due to the non-canonical MO basis.

Please read related keyword CC_prog.

| feature | keywords | supported programs |
| --- | --- | --- |
| CCSD(T) energy | mokit{noRI} | all |
| CCSD(T) unrelaxed DM | mokit{noRI,NO} | Molpro/PySCF/PSI4/ORCA/QChem |
| CCSD(T) relaxed DM | mokit{noRI,NO,RelaxedDM} | Molpro/Gaussian/PSI4/QChem |
| CCSD(T) analytical gradients | mokit{noRI,force} | Molpro/PySCF/Gaussian/PSI4/QChem |
| DF-CCSD(T) energy | mokit{} | TBD |
| DF-CCSD(T) unrelaxed DM | mokit{NO} | TBD |
| DF-CCSD(T) relaxed DM | mokit{NO,RelaxedDM} | TBD |
| DF-CCSD(T) analytical gradients | mokit{force}| TBD |
| DLPNO-CCSD(T0) energy | mokit{DLPNO} | TBD |

## 4.7.5 CCSDT
Coupled cluster singles, doubles and (full) triples.

The CCSDT nonlinear equations will be solved to obtain t1, t2 and t3 amplitudes, i.e. three sets of amplitudes are solved together and iteratively. The final numerical values of t1 and t2 amplitudes will be different from those values of t1 and t2 in CCSD. So usually we do not obtain the CCSD T1-diagnostic value in a CCSDT calculation. But when `CC_prog=PySCF`, `autosr` will firstly perform a CCSD calculation and then use obtained t1, t2 amplitudes as the initial guess of CCSDT, so you can see a T1-diagnostic value in such case.

Please read related keyword CC_prog.

| feature | keywords | supported programs |
| --- | --- | --- |
| CCSDT energy | mokit{noRI} | PySCF/MRCC/ORCA |
| CCSDT unrelaxed DM | mokit{noRI,NO} | TBD |
| CCSDT relaxed DM | mokit{noRI,NO,RelaxedDM} | TBD |
| CCSDT analytical gradients | mokit{noRI,force} | MRCC |
| DF-CCSDT energy | mokit{} | MRCC/ORCA |

(TBD: to be determined)

## 4.7.6 CCSD-F12
CCSD with F12 correction.

## 4.7.7 CCSD(T)-F12
CCSD(T) with F12 correction.

CCSD(T)-F12A and CCSD(T)-F12B methods (i.e. the A/B approximations) are also supported when `CC_prog=Molpro`.

## 4.7.8 EOM-CCSD
Equation-of-motion based on the CCSD ground state wave function. This is a frequently used wave function method for excited state calculations. EOMCCSD is a synonym for this option.

Please read related keyword EOM_prog.

## 4.7.9 EOM-IP-CCSD
Ionized potential (IP) EOM framework based on the CCSD ground state wave function. This is a frequently used wave function method for IP calculations. IP-EOM-CCSD is a synonym for this option.

Please read related keyword EOM_prog.

## 4.7.10 EOM-EA-CCSD
Electron attachment (EA) EOM framework based on the CCSD ground state wave function. This is a frequently used wave function method for EA calculations. EA-EOM-CCSD is a synonym for this option.

Please read related keyword EOM_prog.

## 4.7.11 CC2
Second-order approximate coupled cluster singles and doubles.

Please read related keyword CC_prog.

## 4.7.12 CC3
Third-order approximate coupled cluster singles, doubles, and triples.

Please read related keyword CC_prog.

## 4.7.13 ADC(2)
Second-order algebraic diagrammatic construction method.

The SOS-ADC(2) and SCS-ADC(2) variants are also supported. The IP-ADC(2) and EA-ADC(2) methods for IP/EA calculations are also supported.

Please read related keyword ADC_prog.

RI-ADC(2) for electronic excited states
```
%mem=6GB
%nprocshared=4
#p ADC(2)/def2TZVP

mokit{}

0 1
O         0.00000000     0.00000000     0.06200700
H         0.00000000    -0.78397600    -0.49205200
H         0.00000000     0.78397600    -0.49205200

```

RI-accelerated IP-ADC(2) for ionized states
```
%mem=6GB
%nprocshared=4
#p IP-ADC(2)/def2TZVP

mokit{}

0 1
O         0.00000000     0.00000000     0.06200700
H         0.00000000    -0.78397600    -0.49205200
H         0.00000000     0.78397600    -0.49205200

```

RI-accelerated EA-ADC(2) for electron attachment
```
%mem=6GB
%nprocshared=4
#p EA-ADC(2)/def2TZVP

mokit{}

0 1
O         0.00000000     0.00000000     0.06200700
H         0.00000000    -0.78397600    -0.49205200
H         0.00000000     0.78397600    -0.49205200

```

ADC(2) (with RI approximation turned off) for electronic excited states
```
%mem=6GB
%nprocshared=4
#p ADC(2)/def2TZVP

mokit{ADC_prog=PSI4,noRI}

0 1
O         0.00000000     0.00000000     0.06200700
H         0.00000000    -0.78397600    -0.49205200
H         0.00000000     0.78397600    -0.49205200

```

## 4.7.14 ADC(3)
Third-order algebraic diagrammatic construction method.

Please read related keyword ADC_prog.

RI-ADC(3) for electronic excited states
```
%mem=6GB
%nprocshared=4
#p ADC(3)/def2TZVP

mokit{ADC_prog=PSI4}

0 1
O         0.00000000     0.00000000     0.06200700
H         0.00000000    -0.78397600    -0.49205200
H         0.00000000     0.78397600    -0.49205200

```

## 4.7.15 QCISD
Quadratic CI calculation including single and double substitutions.

## 4.7.16 QCISD(T)
Quadratic CI calculation including single and double substitutions with a triples contribution to the energy.

