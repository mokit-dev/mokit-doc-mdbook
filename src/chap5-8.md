# 5.8 Examples of `autosr`
The utility `autosr` is designed for advanced electron correlation methods like CCSD(T), CCSDT, and their density-fitting (DF, also called RI) approximations, local correlation approximations, etc (see the full list in [Section 4.7.1](./chap4-7.md#471-methods-in-autosr)). The RI technique is turned on by default. See the supported programs for `MP2_prog` and `CC_prog` in [Section 4.7.2.1](./chap4-7.md#4721-sepcify-programs-for-each-step). By default, `HF_prog=Gaussian`.

## 5.8.1 DF-CCSD(T) calculation using Molpro
See an example
```
%mem=20GB
%nprocshared=4
#p CCSD(T)/cc-pVTZ

mokit{CC_prog=Molpro}

0 1
O   2.72506079   -0.54744525    0.0
H   3.68506079   -0.54744525    0.0
H   2.40460620    0.35749058    0.0

```

By default, the density fitting (DF, also called RI) technique is turned on and the auxiliary basis set of cc-pVTZ are automatically dealt with, so this is in fact a DF-CCSD(T) calculation. If you do not want to use RI, you can specfidy `mokit{CC_prog=Molpro,noRI}`.

The T1 diagnostic will be printed if you perform CCSD or CCSD(T) calculations. The `Force` keyword is supported for calling programs to calculate the analytical nuclear gradients. For example, the CCSD(T) force can be obtained by
```
#p CCSD(T)/cc-pVTZ

mokit{CC_prog=Molpro,force}
```

Currently only `CC_prog=Molpro` or `CC_prog=PSI4` is supported for the CCSD(T) force.

## 5.8.2 DLPNO-CCSD(T1) calculation using ORCA
The input file of `autosr` is just the Gaussian .gjf file. For example, the DLPNO-CCSD(T1) calculation of a water molecule is shown as follows (h2o.gjf)
```
%mem=20GB
%nprocshared=4
#p DLPNO-CCSD(T1)/cc-pVTZ

mokit{}

0 1
O   2.72506079   -0.54744525    0.0
H   3.68506079   -0.54744525    0.0
H   2.40460620    0.35749058    0.0

```

Submit the job
```
autosr h2o.gjf >h2o.out 2>&1 &
```

By default, the HF calculation will be performed by calling Gaussian, and the DLPNO-CCSD(T1) calculation will be performed by calling ORCA. The auxiliary basis set of cc-pVTZ are automatically dealt with by `autosr`. You are supposed to write the memory as large as possible since post-HF jobs usuallty needs a lot of memory. The `%maxcore` in ORCA input file will be the total memory multiplied by 0.8 and divided by the number of processors.

