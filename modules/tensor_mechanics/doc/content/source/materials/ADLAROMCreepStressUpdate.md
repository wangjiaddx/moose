# AD LAROM Creep Stress Update

!syntax description /Materials/ADLAROMCreepStressUpdate

## Description

`ADLAROMCreepStressUpdate` computes the creep rate of materials by sampling a Reduced Order Model (ROM) formulated
via calibration with lower-length scale simulations. This method utilizes the exact same techniques
utilized in [ADPowerLawCreepStressUpdate](/ADPowerLawCreepStressUpdate.md) including the radial
return method implemented in [ADRadialReturnStressUpdate](/ADRadialReturnStressUpdate.md), however
in place of a traditional power-law creep model, a ROM is sampled to determine the creep rate as a
function of temperature, defect concentrations, the von Mises trial stress, and an environmental
factor. In addition, lower-length scale information, here mobile defect dislocations and immobile
defect dislocations, are evolved as determined by the ROM.

### Theory

The creep behavior of metals is governed by physical deformation mechanisms, such as dislocation
glide, climb, and Coble creep [!cite](Asaro:1983kf). If these physical mechanisms are not
explicitly captured with the creep model formulation, transient creep behaviors cannot be readily
captured [!cite](Wen:2017hu) [!cite](Wen:wv). Typical constitutive modeling frameworks are too
computationally costly to be used effectively in engineering applications, so use of a simplified
regression model in place of physics-based simulations, i.e. a reduced order model (ROM) is
necessary.

The ROM method employed in this model depends on a database of physics-based simulation results. The
formulation of that database must include the list of inputs which are needed to distinguish the
desired output(s). In the case of the stainless steel alloy 316H, the degrees of freedom present in
the full-fidelity model are too numerous to be included individually in a ROM database. They are
reduced through instead relying on isotropic effective measures. For example, thermal creep in
randomly textured polycrystalline 316H can be approximated with a J2 material response, i.e., a von
Mises stress criterion [!cite](Hutchinson:1976dr). This reduced list of inputs includes temperature,
von Mises stress, accumulated effective von Mises strain (as a history tracking parameter), average
dislocation density in the cell, and average dislocation density in the cell wall.

The physics-based simulations are expressed in a visco-plastic self-consistent (VPSC) framework
[!cite](Lebensohn:1993gn), [!cite](Lebensohn:2007iq). VPSC model describes the polycrystal as a
collection of orientations (grains) with associated volume fractions chosen to represent the initial
texture of the aggregate. Each grain is regarded as a visco-plastic inclusion embedded in, and
interacting with, a "homogeneous effective medium" (HEM), which has the average properties of the
aggregate. The macroscopic response of the polycrystal results from the contribution of each grain.
The visco-plastic compliance of the HEM is given by a self-consistent condition applied on the grain
averages. The constitutive laws relating strain-rate and stress for the aggregate are written in a
linearized form as,
\begin{equation}
  \dot{\epsilon}=\ \bar{\mathbf{M}}:\bar{{\sigma}} + \bar{\dot{\epsilon}}_0,
\end{equation}
where $\bar{\mathbf{M}}$ and $\bar{\dot{\epsilon}}_0$ are the macroscopic visco-plastic compliance tensor
and back-extrapolated terms for the aggregate, respectively. The inclusion formalism couples stress
and strain-rate in the grain $(\sigma ^g,\dot{\epsilon}^g)$ with the average stress and strain-rate in the effective medium $(\bar{\sigma},\bar{\dot{\epsilon}})$ through the interaction equation,
\begin{equation}
  \dot{\epsilon}^g - \bar{\dot{\epsilon}} = \left(\eta^{eff}(1-\mathbf{E})^{-1}:\mathbf{E}:\bar{\mathbf{M}}^{eff})\right):(\sigma ^g-\bar{\sigma}),
\end{equation}
where $\mathbf{E}$ is the visco-plastic Eshelby tensor, $\bar{\mathbf{M}}^{eff}$ is the macroscopic
visco-plastic compliance tensor and the parameter $\eta^{eff}$ "tunes" the stiffness of the
inclusion-matrix interaction.

The constitutive laws describe the relative contributions of dislocation glide climb and Coble creep to inform
deformation at the single crystal level. The total creep rate can be expressed as,
\begin{equation}
  {\dot{\epsilon}}^p=\ {\dot{\epsilon}}^d+\ {\dot{\epsilon}}^c+\ {\dot{\epsilon}}^{coble}.
\end{equation}
Here, ${\dot{\epsilon}}^d$, ${\dot{\epsilon}}^c$ and  ${\dot{\epsilon}}^{coble}$ refer to the plastic
deformation accumulated through dislocation glide, climb and Coble creep, respectively. The strain
rates due to dislocation motion can be written as the sum of the mean shear/climb rates over all active systems in the grain:
\begin{equation}
  \begin{aligned}
    {\dot{\epsilon}}^d=\ \sum_{s}{m_{ij}^s{\bar{\dot{\gamma}}}^s}\\
    {\dot{\epsilon}}^c=\ \sum_{s}{c_{ij}^s{\bar{\dot{\beta}}}^s}
  \end{aligned}
\end{equation}
Here $\bar{\dot{\gamma}}$  and $\bar{\dot{\beta}}$ denote the mean shear and climb rates, respectively, and
$m^{sis}$ the symmetric part of the Schmid tensor. $c=(b^s\bigotimes b^s)$ is  the climb tensor for
edge dislocations [!cite](Lebensohn:2010hr).

This model relies on the original work of [!cite](Wang:2017fo) and [!cite](Wang:2016fs) in which the
response of each material point (grain) is described in a statistical fashion (i.e. via the internal
stress distribution) to allow the quantification of type III stresses associated with dislocations.
With this approach, one must calculate the average strain rate in one grain accounting for the
mechanical response in all sub-material points. This treatment provides a connection between the
localized stress distribution within sub-material point and the average response of the material
point.   It is also required by the effective medium models, such as the VPSC framework, that assume
the strain rate and stress within each grain are homogenous. As per [!cite](Wang:2017fo) and
[!cite](Wang:2016fs), the mean shear rate of the slip system in the grain domain is expressed as:
\begin{equation}
  {\bar{\dot{\gamma}}}^s=\ \int_{-\infty}^{\infty}{{\dot{\gamma}}^s\left(\tau^s\right)}\ P\left(\tau^s-{\bar{\tau}}^s\right)d\tau^s,
\end{equation}
where ${\dot{\gamma}}^s$ is the shear rate of one sub-material point. $\tau^s$ is the local resolved shear stress.
${\bar{\tau}}^s=\sigma :m^s$ denotes the mean resolved shear stress in one grain, where $\sigma$ is
the deviatoric stress tensor. Notice that this law applies to the climb rate as well:
\begin{equation}
  {\bar{\dot{\beta}}}^s=\ \int_{-\infty}^{\infty}{{\dot{\beta}}^s\left(\tau_{climb}^s\right)}\ P\left(\tau_{climb}^s-\ {\bar{\tau}}_{climb}^s\right)d\tau_{climb}^s,
\end{equation}
with $\tau_{climb}^s$ and ${\bar{\tau}}_{climb}^s=\sigma: c^s$, being the local and global resolved climb stress, respectively.

As a first approximation, each component of the stress field can be described by a Gaussian
function. Therefore, the probability distribution function $P\left(\tau^s-{\bar{\tau}}^s\right)$,
representing the volume fraction of sub-material points with $\tau^s$, can be written as:
\begin{equation}
  P\left(\tau^s-{\bar{\tau}}^s\right)=\ \frac{1}{\sqrt{2\pi V}}\exp{\left(-\frac{\left(\tau^s-{\bar{\tau}}^s\right)^2}{2V^2}\right)},
\end{equation}
where $V$ is the distribution variance. [!cite](Wang:2016fs) suggest that the dispersion of intragranular stress
depends on the total dislocation density, and express $V$ as,
\begin{equation}
  V=\ \eta\sqrt{\rho_{cell}+\rho_{cw}},
\end{equation}
Where $\eta={10}^7$ MPa/m is an effective scaling coefficient, introduced here for the sake of simplicity. In practice, though, $\eta$ is
expected to be a function depending on the "Contrast factor" for each dislocation population and on
dislocation arrangements. The material parameters in use for the generation of the ROM database can
be found elsewhere [!cite](Tallman:YBweFy2x). For the meaning and implementation of the parameters,
readers are referred to the original work [!cite](Wen:wv).


The polynomial regression model used as the mathematical form of the ROM is formulated in terms of orthogonal polynomial
terms, defined using Legendre polynomials. These terms are expressed as $P_i\left(x\right)$,
polynomial of degree $i$, evaluated for input $x$, where the range of $x$ has been normalized to the
$\left[-1,\ 1\right]$ interval. The model form includes polynomial terms of each input parameter and all interaction terms thereof, i.e.,
\begin{equation}
	Output\ \sim \sum_{w=0}^{deg}{\ldots\sum_{z=0}^{deg}{\alpha_{w\ldots z}P_w\left(x_w\right)\ldots}}P_z\left(x_z\right),
\end{equation}
where $deg$ is the maximum degree of polynomial to be used in the model, $\alpha_{w\ldots z}$ is the regression coefficient for the term formed from the
 product of $w$-th degree polynomial of input $x_w$, etc.

Three output values are required from the ROM: $\dot{\epsilon}_{vm}$, ${\dot{\rho}}_{cell}$, and
${\dot{\rho}}_W$. While the strain rate is the desired output for predicting creep behavior, The
defect density rates are also needed, as they affect the evolution of the creep rate over time
through their own evolution. An entirely independent regression model is employed to predict each of
the three outputs required. For example, the regression model for $\dot{\epsilon}_{vm}$ is shown, i.e.,
\begin{equation}
  {\dot{\varepsilon}}_{vm}\ ~\ \sum_{n_\varepsilon}^{deg}\sum_{n_{\rho_{cell}}}^{deg}\sum_{n_{\rho_W}}^{deg}\sum_{n_\sigma}^{deg}\sum_{n_T=0}^{deg}{\alpha_{n_\varepsilon n_{\rho_{cell}}n_{\rho_W}n_\sigma n_T}^{{\dot{\varepsilon}}_{vm}}P_{n_\varepsilon}\left(\varepsilon_{vm}\right)P_{n_{\rho_{cell}}}\left(\rho_{cell}\right)P_{n_{\rho_W}}\left(\rho_W\right)P_{n_\sigma}\left(\sigma_{vm}\right)P_{n_T}\left(T\right)\ }
\end{equation}
where each $n_{sub}$ is an index corresponding to the degree of the Legendre polynomial of the input parameter in the subscript. The coefficients are denoted
as specific to the output $(\dot{\epsilon}_{vm})$ with a superscript. Additionally, mappings are
used to convert from engineering input units to terms less susceptible to error propagation for
informing the ROM. All inputs and outputs are handled internally such that the use of the ROM is
unchanged by the inclusion of these mappings.

The coefficients, $\alpha$, are evaluated by fitting the ROM to a database of results from the VPSC
simulations using the physics-based constitutive laws. The maximum degree of the polynomial terms is
selected to maximize fidelity to the data while avoiding an overfitting of the data, where
overfitting is indicated by a stark difference in the regression fit to training and testing data.
To validate the obtained regression coefficient values, initial conditions are given to the ROM and
VPSC, and the resulting simulations are compared.  

### Implementation

While `ADLAROMCreepStressUpdate` contains the necessary algorithms contained to evaluate the ROM,
the actual data that comprises the ROM is contained in a separate UserObject that inherets from
`LAROMData`. The data contained in `LAROMData` is the maximum and minimum values used to calibrate
the ROM, the transform used to prepare the input for the ROM, and the Legendre Polyonimal
coefficients.

`ADLAROMCreepStressUpdate` must be run in conjunction with an inelastic strain return mapping stress
calculator such as [ADComputeMultipleInelasticStress](ADComputeMultipleInelasticStress.md).

!listing modules/tensor_mechanics/test/tests/rom_stress_update/2drz.i block=Materials

In addition, the material specific `LAROMData` user object must also be coupled:

!listing modules/tensor_mechanics/test/tests/rom_stress_update/2drz.i block=UserObjects

!syntax parameters /Materials/ADLAROMCreepStressUpdate

!syntax inputs /Materials/ADLAROMCreepStressUpdate

!syntax children /Materials/ADLAROMCreepStressUpdate

!bibtex bibliography
