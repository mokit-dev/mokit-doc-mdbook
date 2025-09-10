# Dependency on QC Programs

## Minimal requirements

Dependencies on quantum chemistry packages are different for each executable or module. Here the minimum requirements for binary executables `automr`, `autosr`, `frag_guess_wfn` and other various are listed:
1. `automr`: PySCF, GAMESS
2. `autosr`: Gaussian
3. `frag_guess_wfn`: Gaussian
4. Most of the utilities do not depend on quantum chemistry packages except that the modules `py2gau`, `py2orca`, `py2molpro`, etc, work with PySCF installed.


### For people who cannot use Gaussian

The MOKIT developers understand that some people cannot (or do not want to) use Gaussian. You can set `HF_prog=PySCF` to call PySCF to perform the RHF/UHF calculations, e.g.
```
%mem=4GB
%nprocshared=2
#p CASSCF/cc-pVDZ

mokit{HF_prog=PySCF}

0 1
O      -0.23497692    0.90193619   -0.068688
H       1.26502308    0.90193619   -0.068688
H      -0.73568721    2.31589843   -0.068688

```
This does not require Gaussian installed on your computer.


### Setup Gaussian and PySCF

Usually there are no special requirements on setup Gaussian. 

When installing PySCF and MOKIT from conda, please read [this instruction](./chap2-2.md#use-mokit-with-default-channel) and [this](./chap2-2.md#use-mokit-with-conda-forge-channel).

Read [Section 2.3.7](./chap2-3.md#237-notes-on-quantum-chemistry-packages) and [FAQs](./chap_appdx.md#a1-frequently-asked-questions-faq) if you have trouble installing them or calling them with MOKIT.

### Modify and Setup GAMESS

Note that the original GAMESS code can only deal with GVB <=12 pairs. But nowadays we can do hundreds of pairs. It is required by MOKIT to modify GAMESS to go beyond 12 pairs. See instructions in [Section 4.4.10](./chap4-4.md#4410-gvb_prog).

Also, you need to set the environment variables as mentioned in [Section 2.3.4](./chap2-3.md#234-environment-variables).

## Additional requirements for using specific methods

Some methods supported by MOKIT cannot be used with minimal requirements above. Please read the documentation of "XXXX_prog" keywords of `automr` or `autosr` for more information. For example, [DMRGSCF_prog](./chap4-4.md#4414-dmrgscf_prog) of `automr` and [CC_prog](./chap4-7.md#cc_prog) of `autosr`. 

Here we provide a list for common used prog keywords and corresponding dependencies.

| `automr` methods | additional dependencies | instructions |
| --- | --- | --- |
| [DMRGCI/DMRGSCF](./chap4-4.md#4414-dmrgscf_prog) | pyscf[dmrgscf] and Block | [here](./chap2-3.md#installation-tips-and-instructions) |
| [CASPT2](./chap4-4.md#4415-caspt_prog) | OpenMolcas/Molpro/ORCA | [here](./chap2-3.md#installation-tips-and-instructions) |
| [MRCISD](./chap4-4.md#4417-mrcisd_prog) | See keyword documentation | |
| [MC-PDFT](./chap4-4.md#4420-mcpdft_prog) | pyscf-forge | |


| `autosr` methods | additional dependencies | instructions |
| --- | --- | --- |
| [DLPNO-CC](./chap4-7.md#cc_prog) | ORCA | |

## Setups for calling other QC programs

See [Section 2.3.4](./chap2-3.md#234-environment-variables) for setting environment variables for those programs. 
See [Section 2.3.7](./chap2-3.md#237-notes-on-quantum-chemistry-packages) for installation tips and instructions.


