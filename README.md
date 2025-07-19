CROCO dev tracers
-----------

Notes on recent developments for tracer budgets corresponding to the following options:

==========

 ```
 #  define MLD_RHO 
 #  define DIAGNOSTICS_TS
 #  define DIAGNOSTICS_TS_MLD
 #  define DIAGNOSTICS_TRACER_ISO 
 ```


## Definition of MLD

 ```
 #  define MLD_RHO 
 ```
 
If MLD\_RHO is activated, the definition of the MLD is provided in **mld_rho.F** (by default a potential density difference of 0.03 kg m$$^{-3}$$) instead of the KPP based MLD. It corresponds to the vertical index *kbl_rho(i,j)* (instead of *kbl(i,j)*).
 
## MLD integrated tracer budget

 ```
 #  define DIAGNOSTICS_TS
 #  define DIAGNOSTICS_TS_MLD 
 #  define MLD_RHO 
 ```
 
Outputs tracer divergence (for active and passive tracers) integrated over the ML depth (defined using the MLD_RHO option). All terms are volume integrated.


It provides both integration of the 3D terms (provided in define DIAGNOSTICS\_TS):

- **tpas01\_xadv\_mld** = "MLD xi advection term" ;
- **tpas01\_yadv\_mld** = "MLD eta advection term" ;
- **tpas01\_vadv\_mld** = "MLD vertical advection term" ;
- **tpas01\_hmix\_mld** = "MLD xi mixing term" ;
- **tpas01\_vmix\_mld** = "MLD vertical mixing term" ;
- **tpas01\_forc\_mld** = "MLD Forcing term (Q & Nudg)" ;
- **tpas01\_rate\_mld** = "MLD time rate of change" ;

as well as additional terms specific to the MLD, such as the entrainment rate:

- **tpas01\_entr\_mld** = "MLD entrainment rate" ;

which accounts for MLD time-variation

And other additional terms:

- **tpas01\_xout\_mld**  = "MLD xi advection outside MLD" ;
- **tpas01\_yout\_mld** = "MLD eta advection outside MLD" ;

These correspond to advective fluxes directly outside the ML (unlike tpas01\_xadv\_mld and tpas01\_yadv\_mld, which include all horizontal advection fluxes, whether inside or outside the ML).

The point-wise budget is:

**tpas01\_rate\_mld = 
tpas01\_xadv\_mld + tpas01\_yadv\_mld + tpas01\_vadv\_mld + tpas01\_hmix\_mld + tpas01\_vmix\_mld + tpas01\_forc\_mld + tpas01\_entr\_mld**


And, when integrated spatially over a full patch of tracer, we should get: 

- **tpas01\_xout\_mld = tpas01\_xadv\_mld**
- **tpas01\_yout\_mld = tpas01\_yadv\_mld**


Computation (integration of 3d tracer terms) is done first in **step3d_t.F** for advective, vertical mixing, and forcing terms. Horizontal mixing (as well as updates to vertical mixing, entrainment and rate) are then done in **t3dmix.F** and **t3dmix\_spg.F**.

## Isopycnal/Diapycnal terms


 ```
 #  define DIAGNOSTICS_TRACERS_ISO
 ```
 
 Separate the tracer fluxes into iso and diapycnal contributions.
 
### option 1 (currently partially implemented)

Save 3d tracer fluxes (in **step3d_t.F**, **t3dmix.F**, and **t3dmix\_spg.F**):

- TF\_xHmix
- TF\_yHmix
- TF\_zHmix
- TF\_zVmix
- TF\_Vadv
- TF\_Xadv
- TF\_Yadv

Then project them on isopycnal/diapycnal directions in **set\_diags\_tracer_iso.F**, which is called in **step.F** just before writing outputs.

Note that density gradients are computed in **compute\_buoyancy\_gradient.h**, which is called in **step.F** after the predictor stage, to be consistent with the density gradients used in the advective schemes.

### option 2 (to be implemented)

Directly project and vertically integrate iso/diapycnal fluxes in **step3d_t.F**, **t3dmix.F**, and **t3dmix\_spg.F**. And just save them in the form of 2d variables.




 
 
 
 
 
 
 
 
 
 
 