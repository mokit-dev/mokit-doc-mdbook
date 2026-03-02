## 3.4 Do I need to specify the active space?

The `automr` program can automatically determine the active space, thus you need not specify that. If you do not want the automatically determined one, you can manually specify it, such as CASSCF(m,n) or NEVPT2(m,n), with m, n being the number of active electrons and active orbitals, respectively (this is also the convention defined by Gaussian). Usually it is recommended to specify CASSCF(n,n), i.e. m=n (which usually corresponds to all important bonding orbitals and antibonding orbitals, see DOI: 10.1021/acs.jctc.0c00123).

`m != n` is also supported for `automr`, e.g. CASSCF(8,7) means 8 active electrons and 7 active orbitals. `automr` will find candidate orbitals by the occupation number threshold. Taking the phenol (C6H5OH) as an example, one may want the active space to include the lone electron pair of the oxygen atom and six C=C pi orbitals, so CAS(8e,7o) is desired by the user in such case. And he/she is supposed to specify `CASSCF(8,7)` in gjf file.

Note that although the desired active space can be correctly specified in the input, the calculating results do not necessarily include active MOs which is exactly wanted by the user. There are many factors which affect the calculation. The most important factor may be that the occupation number of some lone pair orbital is too close to 2.0, and thus it will probably be rotated out of the active space during CASSCF orbital optimization. To avoid this problem, the user might need to use SA-CASSCF rather than CASSCF, and the number of states must be specified, e.g. `mokit{Nstates=4}`, because the occupation number of some lone pair orbital could be deviate from 2.0 in some excited state. Automatically determining the number of electronic states is not yet implemented in MOKIT.

If you want to perform a simple GVB calculation rather than CASSCF, you can simply specify GVB or GVB(n), where n is the number of pairs. Manually specifying pairs is only recommended for experienced users. One can also specify the number of GVB pairs within a black-box CASSCF calculation, e.g.
```
%mem=32GB
%nprocshared=16
#p CASSCF/cc-pVDZ

mokit{npair=3}

0 1
...
```

Recall that by default, `automr` will firstly call Gaussian to perform RHF/UHF calculations, generate paired localized orbitals and perform a GVB calculation. Then pick up important GVB pair orbitals (as well as all singly occupied orbitals) and put them into the CAS active space. Without specifying `npair=3`, `automr` will perform a GVB(15) calculation for the benzene molcule, but if one specifies npair=3, `automr` will conduct the GVB(3) calculation when it comes to the GVB step. The final step - the CASSCF calculation will use (6,6) in both cases since we know the important orbitals are six pi orbitals. Therefore, it can save some computational cost if you know the studied molecule well. Similarly, `npair=5` can be specified for the naphthalene molecule.

Note that usually there is no need to write `DMRGCI` or `DMRGSCF`, the users can just write `CASCI` or `CASSCF`. For a CASCI job, if the active space is larger than that of singlet (16,16), it will switch from CASCI to DMRG-CASCI automatically. For a CASSCF job, if the active space is larger than that of doublet (15,15), the program will automatically switch from CASSCF to DMRG-CASSCF. Similarly, the NEVPT2/CASPT2/MC-PDFT will be automatically switched into DMRG-NEVPT2/DMRG-CASPT2/DMRG-PDFT. The only exception is that the users just want to perform a DMRG calculation with a small active space, then he/she needs to specify `DMRGCI` or `DMRGSCF`.

Usually the automatically determined active space is reasonable. The algorithm in MOKIT is designed to automatically determine the minimum active space for a given molecule. However, when you are studying a potential energy curve/surface, the automatically determined active space may be not the same size for each geometry. For example, for N<sub>2</sub> molecule at *d*(N-N) = 1.15 Å, the CAS(4,4) may be automatically determined by `automr`, but for for *d*(N-N) = 4.0 Å, the active space turns into CAS(6,6). Thus, if you want to keep the size to be CAS(6,6), you need to specify CASSCF(6,6), NEVPT2(6,6), MRCISD(6,6), etc.

