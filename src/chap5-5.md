# 5.5 Manually making consistent active space

The active space automatically determined by `automr` of MOKIT may not be the same
size for cases involving chemical reactions, different local minima, or different
spin states of the same geometry. Or although the active space size remains unchanged,
but the active orbitals become substantially different among different geometries.
In such cases, if we want to obtain an accurate relative energy, we should pay
attention to the following two points:

(1) Different species are required to have the same size of active space.

(2) Active orbitals of different species are supposed to contain the same main
components.

Point (2) is merely recommended by the developer jxzou, not a rule which you must
obey. Taking the singlet-triplet gap of a molecule as an example: if MOKIT automatically
determined/recommends the active space of singlet to be CAS(2,2), but triplet to
be CAS(4,4), then it would be unfair to directly compare the energies of CASSCF(2,2)
*v.s.* CASSCF(4,4). Of course, it would be unfair to simply compare NEVPT2 energies
(based on corresponding CASSCF), either. In such cases, the author recommends two
approaches:

(i) If we find two orbitals in CAS(4,4) of triplet to be not active, i.e. occupation
numbers close to 2 and 0, respectively, we can read CAS(4,4) orbitals, move these
two orbitals out of the active space, and perform a CAS(2,2) computation. Then
active space size of two spin states are the same.

(ii) If there are similar active orbitals but they are too active to be moved out
of the active space, or if we cannot find any similar active orbital between two
active space. We should consider to enlarge the size of CAS(2,2) for singlet. The
extension target can be CAS(4,4), or can be even larger, depending on the similar
extent of two active space.

Obviously the approaches above correspond to intersection set and union set in mathematics.
Currently these two approaches have not been implemented in MOKIT, since it requires
a lot of techniques to make them become automatic and black-box. Below I will provide
two examples to illustrate how to manually make two active space consistent. For comparison
of active space from more than two species, the spirit is identical.

## 5.5.1 The Diels-Alder reaction of 1,3-butadiene and ethane
To be added.

## 5.5.2 Singlet-Triplet gap of an iron-containing molecule
To be added.

## 5.5.3 Find a C-C bond for an equilibrium geometry
If one studies the C-C bond dissociation in ethanol using multireference computations
conducted by `automr` in MOKIT, he/she may find that when the bond is stretched,
`automr` usually provide desired CAS(2,2) results. However, for the equilibrium
geometry (or near that), `automr` may provide a CAS(2,2) which does not contain
the desired chemical bond. This is because there is no multireference character
for the equilibrium geometry, `automr` cannot locate the chemical bond as desired.
In fact, in such case the word 'desired' has different meanings for different users.
Some other users may want the CAS(2,2) to contain the C-O bond, and some users may
want the C-H bond.

Here I will show how to swap two orbitals to get desired results. Using the ethanol as an example, assuming that we have perform a simple CASPT2(2,2)/cc-pVDZ calculation, the input file ethanol.gjf is shown below

```
%mem=4GB
%nprocshared=4
#p CASPT2(2,2)/cc-pVDZ

mokit{CASPT_prog=ORCA}

0 1
C     0.00000000  0.00000000  0.00000000
H     0.00000000  0.00000000  1.09270400
H     1.03726900  0.00000000 -0.34698400
H    -0.48033400  0.91954100 -0.34310600
C    -0.73649800 -1.21494100 -0.53167000
H    -0.24338800 -2.13583800 -0.19123400
H    -0.72503400 -1.21412700 -1.63028700
O    -2.08207000 -1.16197400 -0.04736100
H    -2.56286600 -1.92666800 -0.37754100

```

And we find that the HONO and LUNO of CASSCF(2,2) correspond to the C-O bond

<img src="images/5_5_3_ethanol.png" width="75%" height="75%" />

If we want it to be the C-C bond, we should swap/exchange orbitals in GVB NOs since
they are the initial guess of CASSCF. To accomplish that, we start python
```python
from mokit.lib.gaussian import permute_orb
permute_orb('ethanol_rhf_proj_loc_pair2gvb8_s.fch',6,13)
permute_orb('ethanol_rhf_proj_loc_pair2gvb8_s.fch',14,21)
exit()
```

The orbital indices 13 and 14 are just HONO and LUNO of GVB NOs, while the 6 and 21 are the C-C bonding and anti-bonding orbitals we want. To know the orbital indices, you should visualize MOs in advance. Now the file ethanol_rhf_proj_loc_pair2gvb8_s.fch is updated. We use it as the initial guess of CASSCF(2,2), i.e. submit the job
```python
python ethanol_rhf_gvb8_CASSCF.py >ethanol_rhf_gvb8_CASSCF.out 2>&1
```

then we re-open the file ethanol_rhf_gvb8_CASSCF_NO.fch and check whether the HONO
and LUNO now correspond to the C-C bond. Finally, we update the .gbw file and re-
submit the CASPT2 job
```
fch2gbw ethanol_rhf_gvb8_CASSCF_NO.fch ethanol_rhf_gvb8_CASSCF_CASPT2.gbw
orca ethanol_rhf_gvb8_CASSCF_CASPT2.inp >ethanol_rhf_gvb8_CASSCF_CASPT2.out 2>&1
```

where running the `fch2gbw` utility is to update the file ethanol_rhf_gvb8_CASSCF_CASPT2.gbw
since CASSCF orbitals are stored in this file. Note that you should find CASPT2
energy in file ethanol_rhf_gvb8_CASSCF_CASPT2.out by yourself. It is not in `automr`
output file currently.

