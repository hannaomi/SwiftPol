--- 
title: ‘SwiftPol:A Python package for building and parameterizing <i>in silico<i> polymer systems' 
tags: 
  - Python 
  - polymer 
  - force field 
  - molecular dynamics 
  - polydispersity 
authors: 
  - name: Hannah N. Turney 
    orcid: 0009-0002-3298-0309 
    affiliation: 1 
  - name: Micaela Matta 
    orcid: 0000-0002-9852-3154
    affiliation: 1 
affiliations: 
 - name: Department of Chemistry, King’s College London 
   index: 1 
   ror: 0220mzb33

date:  DD Month YYYY 
bibliography: paper.bib 
 
 
--- 
 
# Summary 

A polymer sample contains a natural degree of variation in its structure and non-uniformity between its chains, which influences the bulk material properties of the sample. This innate heterogeneity is often disregarded in the *in silico* study of a polymer system, resulting in divergence from its *in vitro* counterpart. This paper presents ‘SwiftPol,’ a user-guided Python software for the automated generation of polydisperse polymer systems which are representative of experimental samples. 

# Statement of need 
Polymer MD simulations are often performed with uniform idealized systems that do not capture the heterogeneity of their experimental counterparts. The result of this misalignment is non-convergence between MD-derived polymer properties and experimental data, and these MD simulations can overlook key components of polymer physics such as polydispersity and semi-crystallinity  [@schmid_understanding_2023]. Studies have demonstrated that polymer material properties are highly sensitive to variations in substructure, making it essential to account for this bulk diversity in order to capture the true physics of polymer systems [@wan_effect_2021]. 

Existing polymer MD studies showcase an assortment of approaches to manually incorporate polydispersity into their polymer chain builds [@andrews_structure_2020; @kawagoe_construction_2019; @stipa_molecular_2021]. Although effective for their associated applications, these manual approaches are not universally applicable methods for introducing polydispersity into molecular dynamics systems or are performed using proprietary software.
Open source software packages designed to build *in silico* polymer systems are focused on the design of polymers at the monomer and single-chain scale [@davel_parameterization_2024; @klein_hierarchical_2016; @santana-bonilla_modular_2023]. However, there is not currently a software package available that integrates these smaller scale characteristics into *in silico* polymer design, whilst also effectively capturing the naturally occurring heterogeneity and polydispersity in real-life polymer systems. The development of SwiftPol was driven by the need to fill this gap in multi-scale building functionality of existing polymer building packages, to enable our work modelling of the behavior of bulk polymer systems.

Here, we will detail the development of SwiftPol - a user-guided python tool for building representative polymer ensembles, and subsequent studies to show its relevance and performance. 

SwiftPol uses open-source Python libraries ‘RDkit’, OpenFF-interchange, and OpenFF-toolkit to promote reproducibility and portability [@landrum_rdkitrdkit_2024; @thompson_openff_2024; @wagner_openforcefieldopenff-toolkit_2024; @wang_open_2024].  We have ensured that SwiftPol can be seamlessly integrated into existing open-source software built for parameterization and simulation, to allow the user to select their preferred force field, topology format and engine.


# Package Overview 

The SwiftPol build module contains Python functions to build both single polymer chains, and polydisperse polymer chain ensembles. 

SwiftPol takes as an input the simplified molecular-input line-entry system (SMILES) string of all monomer components in a polymer, as well as a list of parameters representing the target average properties of the ensemble; monomer % composition (for co-polymers), length, number of chains, blockiness (for blocky co-polymers), terminals, residual monomer. The user must define the reaction SMARTS which describes the polymerization reaction associated with their polymer chemistry. 

As depicted in \autoref{Figure 1}, SwiftPol generates an initial polymer sequence that fits the chain length and terminal parameters exactly, using a probability function to determine the ratio of monomers in the chain. In the case of a block co-polymer, the chain is passed to a second function which tests whether the values for blockiness and % monomer are within 10% of the input variable. The +/- 10% acceptance margin introduces polydispersity into the ensemble by ensuring a certain level of non-uniformity between polymer chains, without straying too far from the input value. 

If all tests are passed, the chain is appended to the Python polymer ensemble build object, and the associated properties of the chain are calculated and added as ensemble attributes. Otherwise, the chain is discarded, and the process is repeated. Once the ensemble size is satisfied, average properties are calculated using in-built SwiftPol functions. 

![Flowchart showing the process of building a polymer ensemble using SwiftPol.\label{Figure 1}](Fig_1_Swiftpol.png) 

This approach allows for the generation of a polydisperse chain ensemble, meaning each chain displays different properties but the ensemble matches the target properties and distribution, as is observed in experimental polymer samples. 

SwiftPol also contains functions to generate conformers, pack periodic boxes, and assign force field parameters to the polydisperse ensembles. 

# Applications using poly(lactide-co-glycolide)

Using SwiftPol, we have successfully constructed polydisperse systems of poly(lactide-co-glycolide) (PLGA), a widely used biodegradable polymer. We used the molecular structures and properties of experimental PLGA products as input for SwiftPol building functions to create representative PLGA systems to be used for molecular dynamics simulations. By integrating experimental data, such as chain terminals, copolymer ratios of lactic and glycolic acid, and blockiness, we have been able to replicate the bulk characteristics of various commercial polymer products, namely polydispersity. 

A full example implementation of SwiftPol for building PLGA systems can be found in the [building a PLGA system example notebook](Example_Notebooks/PLGA_demo.ipynb).

We used SwiftPol to build ‘product X’, a commercially available 75:25 LA:GA ester-terminated PLGA. Following the chain build, another SwiftPol function was used to calculate the appropriate box size for the unit cell, number of water molecules, salt molecules, and residual monomer molecules to include in the complete condensed polymer ensemble.
The input values for the SwiftPol builder, seen in \autoref{Table 1}, were taken from quality assurance documents provided by the manufacturer of product X, except the value for blockiness which was measured experimentally by Sun et al [@sun_characterization_2022].


Input parameters for SwiftPol PLGA builder function, for the building of product X.[]{label=Table 1}

| INPUT                                     | VALUE       |
|-------------------------------------------|-------------|
| SYSTEM SIZE                               | 3           |
| TARGET LACTIDE PROPORTION (%)             | 75          |
| DEGREE OF POLYMERIZATION (MONOMER)       | 50          |
| TARGET CHAIN BLOCKINESS                   | 1.7         |
| TERMINAL                                  | Ester       |
| RESIDUAL MONOMER (% W/W)                 | 0.05        |
| NACL CONCENTRATION (M)                   | 0.1         |


The system attributes assigned by SwiftPol to the completed condensed PLGA unit cell are in seen in \autoref{Table 2},


[SwiftPol system build attributes. x̄n = mean value of attribute across n chains.]{label=Table 2}

| ATTRIBUTE                               | X̄N         |
|-----------------------------------------|-------------|
| SYSTEM SIZE (CHAINS)                   | 3           |
| ACTUAL LACTIDE PROPORTION (%)           | 68.9        |
| AVERAGE CHAIN BLOCKINESS                | 1.65        |
| AVERAGE MOLECULE WEIGHT (DALTON)       | 3370        |
| AVERAGE CHAIN LENGTH (MONOMERS)        | 50          |
| POLYDISPERSITY INDEX                    | 1.68        |
| BUILD TIME (S)                          | 1.4         |



# Speed Benchmarking

We determined whether SwiftPol can build polymer ensembles and chains with sizes that are relevant to the system scales of interest by performing a stress test. \autoref{Figure 2}, shows measurements of the time benchmarking results, illustrating that SwiftPol can build large-scale systems in a realistic time frame, and will not create a bottleneck in an MD workflow.


![ A) Time, t, taken to build systems with a single-chain, ranging from a 10-mer to a 1000-mer. B) Time, t, taken to 50-mer chain build systems ranging from 10 chains to 250 chains.\label{Figure 2}](Fig_2_Swiftpol.png) 


# Conclusion 

Considering key experimental properties of bulk polymer materials during *in silico* system building creates representative simulations which are more characteristic of experimental systems 


# Defining Polymer Properties 

SwiftPol uses the following expressions to define key polymer properties. 

Monomer ratio, *R~m*, is the ratio of monomer A to monomer B in an AB co-polymer, shown in \autoref{equation 1} 

\begin{equation}\label{equation 1} 
\mathit{R_{m}} = \frac{n(A)}{n(A+B)} 
\end{equation} 

Degree of polymerization, DOP, is the mean polymer chain length in the system, shown in \autoref{equation 2}. 

\begin{equation}\label{equation 2} 
DOP = \overline{x}(nA+nB) 
\end{equation} 

Number of chains, n~chains, is the total number of chains built by SwiftPol and appended to the object, shown in \autoref{equation 3}. 

\begin{equation}\label{equation 3} 
n_{chains} = total\,number\,of\,chains\,built 
\end{equation} 

Blockiness, *b*, is a measurement of the distribution of monomers in an AB co-polymer, shown in \autoref{equation 4}. 

\begin{equation}\label{equation 4} 
\mathit{b} = \frac{nB-B\,bonds}{nA-B\,bonds} 
\end{equation} 

Residual monomer, *M~resid*, is the % of residual monomer molecules in the system, shown in \autoref{equation 5}. 

\begin{equation}\label{equation 5} 
\mathit{M_{resid}} = \frac{M_{w}(M_(resid))}{M_{w}(Carbon-containing\,compounds)} 
\end{equation} 


# Acknowledgements 

Hannah Turney is supported by funding contributions from UKRI Biotechnology and Biological Sciences Research Council (grant ref. BB/T008709/1) and Johnson&Johnson Innovative Medicine. 
We acknowledge contributions and feedback from Jeffrey Wagner at the Open Force field consortium and Anusha Lalitha, David Hahn and Gary Tresadern at Johnson&Johnson Innovative Medicine.
We acknowledge the use of King’s College London e-research Computational Research, Engineering and Technology Environment (CREATE) high performance computing facility in the development and testing of SwiftPol.


# References 

Andrews, J., Handler, R.A. and Blaisten-Barojas, E. (2020) ‘Structure, energetics and thermodynamics of PLGA condensed phases from Molecular Dynamics’, Polymer, 206, p. 122903. Available at: https://doi.org/10.1016/j.polymer.2020.122903.
Davel, C.M. et al. (2024) ‘Parameterization of General Organic Polymers within the Open Force Field Framework’, Journal of Chemical Information and Modeling [Preprint]. Available at: https://doi.org/10.1021/acs.jcim.3c01691.
Kawagoe, Y. et al. (2019) ‘Construction of polydisperse polymer model and investigation of heat conduction: A molecular dynamics study of linear and branched polyethylenimine’, Polymer, 180, p. 121721. Available at: https://doi.org/10.1016/j.polymer.2019.121721.
King's College London. (2024). King's Computational Research, Engineering and Technology Environment (CREATE). Retrieved October 28, 2024, from https://doi.org/10.18742/rnvf-m076
Klein, C. et al. (2016) ‘A Hierarchical, Component Based Approach to Screening Properties of Soft Matter’, in R.Q. Snurr, C.S. Adjiman, and D.A. Kofke (eds) Foundations of Molecular Modeling and Simulation: Select Papers from FOMMS 2015. Singapore: Springer, pp. 79–92. Available at: https://doi.org/10.1007/978-981-10-1128-3_5.
Landrum, G. et al. (2024) ‘rdkit/rdkit: 2024_09_2 (Q3 2024) Release’. Zenodo. Available at: https://doi.org/10.5281/zenodo.13990314.
ROCS 3.6.2.0. OpenEye, Cadence Molecular Sciences, Santa Fe, NM. http://www.eyesopen.com.
Santana-Bonilla, A. et al. (2023) ‘Modular Software for Generating and Modeling Diverse Polymer Databases’, Journal of Chemical Information and Modeling, 63(12), pp. 3761–3771. Available at: https://doi.org/10.1021/acs.jcim.3c00081.
Schmid, F. (2023) ‘Understanding and Modeling Polymers: The Challenge of Multiple Scales’, ACS Polymers Au, 3(1), pp. 28–58. Available at: https://doi.org/10.1021/acspolymersau.2c00049.
Stipa, P. et al. (2021) ‘Molecular dynamics simulations of quinine encapsulation into biodegradable nanoparticles: A possible new strategy against Sars-CoV-2’, European Polymer Journal, 158, p. 110685. Available at: https://doi.org/10.1016/j.eurpolymj.2021.110685.
Thompson, M. et al. (2024) ‘OpenFF Interchange’. Zenodo. Available at: https://doi.org/10.5281/zenodo.11389943.
Wagner, J. et al. (2024) ‘openforcefield/openff-toolkit: 0.16.0 Minor feature and bugfix release’. Zenodo. Available at: https://doi.org/10.5281/zenodo.10967071.
Wan, B. et al. (2021) ‘Effect of polymer source on in vitro drug release from PLGA microspheres’, International Journal of Pharmaceutics, 607, p. 120907. Available at: https://doi.org/10.1016/j.ijpharm.2021.120907.
Wang, L. et al. (2024) ‘The Open Force Field Initiative: Open Software and Open Science for Molecular Modeling’, The Journal of Physical Chemistry B, 128(29), pp. 7043–7067. Available at: https://doi.org/10.1021/acs.jpcb.4c01558.
