# Convolutions with HWP within TOAST3

-------------------

Giuseppe Puglisi, Reijo Keskitalo, Davide Maino, Martin Reinecke





## 1. Introduction 

We present here a novel approach  within the 	`conviqt` algorithm, in presence of a spinning Half-Wave plate (HWP) polarization modulator  accounting for the polarisation properties of the beam.   In fact, it  produces a rotation of the beam which is no more stationary in the sky when traditional convolution methodologies are employed . Convolution methods working in harmonic space usually create data-cubes where for each (θ, ϕ, ψ)  position and orientation in the sky (with a suitable grids in the three dimensions) the convolution result is stored. The actual value of the convolved signal  for a generic pointing direction and orientation is obtained via interpolation of the data-cube. The HWP adds an extra-dimension of the data-cube and the required resolution and number of detectors of modern experiments make the problem almost unfeasible.

## 2. IQU convolution: `SimWeightedConviqt`  

With this approach we aim at including the polarized component of the beam that were neglected in the  implementation of the TOAST operator `SimWeightedConviqt` [link ](https://github.com/hpc4cmb/toast/blob/3ba6def76a8e8c6353ad994a979633207bccce58/src/toast/ops/conviqt.py#L569) ,  where the same perfect co-polar beam from total intensity is adopted also for the polarization part. This means that for total intensity the beam is made only of its $I$ (total intensity) component and for the two linear polarisation directions only the $Q\,(U)$ component is non-zero and it is equal to the $I$ component). 

In the following, we briefly describe the previous implementation. 

We firstly, convert the `GRASP` $\theta,\phi$  simulated grid files into the equivalent $IQU$ Stokes beam maps.  

We then generated 3 beam $IQU$ maps so that : 

-  $b^I= (b^I, 0,0 ) $
- $b^Q= (0,b^I, 0 )$  
- $b^U= (0,0, b^I )$  

with $b^I$ being the total intensity component of the initial `GRASP` beam. 

Each beam map is then expanded into $TEB$ spherical harmonic coefficients, $b^X_{\ell,m}$.  We remark here that  

 $b^{I, E} _{\ell m}=b^{I, B} _{\ell m} = b^{Q, T} _{\ell m}= b^{U, T} _{\ell m}=0$, 

which implies that for each beam map only the convolution for the non-zero component  is performed with   the relative sky harmonic coefficients, $a_{\ell m}$ . 

The final convolved TOD is thus  : 

$d_t \propto \sum _{\ell m s }  b^{I*}_{\ell s} a _{\ell m} + (b^{Q*}_{\ell s} a_{\ell m}  + b^{U*}_{\ell s} a_{\ell m} )e^{ 2i \psi +4i\alpha_{HWP} } {}_{\pm 2}Y_{\ell m }$



## 3. TEB convolution : `SimTEBConviqt`

The new convolution algorithm( [link](https://github.com/hpc4cmb/toast/pull/550)  ) assumes that  the unpolarized and polarized properties of the `GRASP` $IQU$ Stokes beam  maps are encoded into two beams: 

- $b_{\ell m}^T $, the total intensity beam 
- $b_{\ell m}^P = b_{\ell m}^E  + i b_{\ell m}^B $ , the _polarized_ beam obtained by expanding into $E$ and $B$ mode the $QU$ beam maps. 

The convolved TOD is then: 

$d_t \propto \sum _{\ell m s }  b^{T*}_{\ell s} a _{\ell m}^T + b^{P*}_{\ell s} (a^E_{\ell m} + ia^B_{\ell m} ) e ^{2i\psi + 4i \alpha_{HWP} }{}_{\pm 2}Y_{\ell m }$

It is immediate to show that the convolution of the polarization   signals is then: 

- $Q \propto b^{E}_{\ell s} a _{\ell m}^E +  b^{B}_{\ell s} a _{\ell m}^B $
- $U \propto b^{E}_{\ell s} a _{\ell m}^B -   b^{B}_{\ell s} a _{\ell m}^E $



## 4. Validation

For both the convolutions described in section 2 and 3 we firstly check that the two methodologies produce similar result when compared with the traditional `SimConviqt` approach in absence of HWP modulation. 



 







