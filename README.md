# Trifunctional macromolecules isomerization


Simulations were run on The Pawsey Centre's Setonix HPE Cray EX supercomputer using the Setonix install of using GROMACS 2023 in conjunction with the GROMOS 54A7 forcefield. 

The mdp input for the production simulations is provided. Folders T0, T1, T3 and T5 contain the topology file for each system, the inital and final frame from each production simulation, and one representative trajectory file. The topology for the solvent, acetonitrile, was downloaded from the Automated Topology Builder (https://atb.uq.edu.au) and is ATB molid: 913062. 

Analysis was performed on trajectory frames spaced at 200 ps intervals. Trajectory visualization, analysis and image rendering were done using VMD version 1.9.4 (https://www.ks.uiuc.edu/Research/vmd). 

# Detailed Procedure

Simulation parameters for the central pyrene-chalcone (PyChal, ATB molid: 1423694), terminal PyChal (ATB molid: 1423693) and methyl 6-hydroxyhexanoate linker (ATB molid: 1423695) subunits of T0, T1, T3 and T5 were developed individually using the Automated topology Builder and compatible with the GROMOS 54A7 molecular dynamics force field for each subunit. Each subunit was capped at its functionalization points to incorporate an overlap region of neighboring units, at least 3 bonds deep. Atomic coordinates and molecular topologies for the complete macromolecule were developed in Chimera 1.17.1 by computationally ligating the appropriate number of monomer units at their overlapping functionalization points to reproduce each experimental trifunctional molecule. In T1, T3 and T5, the two terminal PyChal are connected to the central PyChal through linker fragments. In contrast, the central PyChal of T0 is directly connected to the two terminal PyChal units.
All simulations were prepared and performed using GROMACS 2023 in conjunction with the GROMOS 54A7 forcefield. Four different simulations systems were prepared, corresponding to the macromolecules T0, T1, T3 and T5. In each simulation, a single macromolecule was placed in a cubic box and solvated in acetonitrile (ATB molid: 913062). The box was of sufficient size such that the polymer was at least 1.0 nm from any box edge at the start of the simulation. Each simulation system was energy minimized using a steepest descent algorithm and equilibrated under NPT condition using 1 fs and 2 fs timestep for 1 ns in each case. During the equilibration process, the pressure was maintained at 1 bar using a Berendsen barostat with an isothermal compressibility of 4.5x10-5 bar; and the temperature was maintained at 300 K using the Bussi-Donadio-Parrinello velocity-rescaling thermostat with a coupling constant of 0.1 ps. Non-covalent interactions were calculated with a 1.0 nm cutoff in all simulations. Following equilibration, 500 ns production simulations were carried out in triplicate using a 2 fs timestep.
Analysis was performed on trajectory frames spaced at 200 ps intervals. The pairwise distance between photoactive groups in each arm were calculated using the first carbon atom of the double bond in each PyChal arm (i.e., the double bond carbon atom closest to the carbonyl moiety, shown in SI Figure 8.2)  and the corresponding carbon atom in the opposing PyChal arms.
To determine the relative populations of specific conformations, the replicate trajectories for the T0, T1, T3 and T5 simulations, respectively, were clustered using the Gromos clustering algorithm. The Gromos algorithm uses an RMSD cut-off value to count the number of neighbors for the conformation sampled in each frame of the trajectory. The structure with the largest number of neighbors is then chosen as the central conformation of the first cluster, and all of its neighbors are included in this cluster. This first cluster is then removed from the pool of conformations and clustering continues until all conformations have been assigned to a conformational cluster. The most populated cluster contains the conformation with the largest number of neighbors, and is the predominant cluster sampled, representing at least 86 % of the trajectory. Due to the size differences between T0, T1, T3 and T5, the RMSD cut-off values were adjusted to maintain the cluster population size of the predominant conformation for each macromolecule. This corresponded to RMSD cutoff values of 0.70, 0.85, 0.98 and 1.15 nm for the T0, T1, T3 and T5 systems, respectively. 
Plots of the simulation data were generated using Matplotlib 3.5.2. Trajectory visualization, analysis and image rendering were done using VMD version 1.9.4.

# Commands used for one representative simulation T0.

;Preparation of topology for entire polymer structure using pdb2gmx Gromacs topology builder
gmx pdb2gmx -f T0.pdb

;Defining simulation box
gmx editconf -f conf.gro -o newbox.gro -c -d 1.0 -bt cubic

;Vacuum simulation to condense the extended polymer structure 
gmx grompp -f vac_NVE_300K.mdp -c newbox.gro -p topol.top -o NVE.tpr -maxwarn 2
gmx mdrun -v -deffnm NVE

;Creating a box with dimension 12*12*12 nm3 around polymer structure
gmx editconf -f NVE.gro -c -box 12 12 12 -o T0_box.gro

;Solvation of the system according to the density of acetonitrile (786 kg/m3 which is equivalent to 7.86e-22 g/nm3)
gmx insert-molecules -f T0_box.gro -ci ACN_atb.gro -o T0_ACN.gro -nmol 20000 

;Energy minimize the system
gmx grompp -f minimise.mdp -c  T0_ACN.gro -p T0_ACN.top -o T0_ACN_em.tpr -maxwarn 1
gmx mdrun -v -deffnm T0_ACN_em

;Preparation of an index file
gmx make_ndx -f  T0_ACN_em.gro -o  T0_ACN.ndx

;Equilibration of the system (1fs equilibration followed by 2 fs equilibration)
gmx grompp -f eq_1fs.mdp -c  T0_ACN_em2.gro -r  T0_ACN_em2.gro -n  T0_ACN.ndx -p  T0_ACN.top -o T0_ACN_eq1.tpr  -maxwarn 2  2>&1 | tee eq1_grompp.txt
gmx mdrun -v -deffnm T0_ACN_eq_1000
gmx grompp -f eq_2fs.mdp -c  T0_ACN_eq1.gro -r T0_ACN_eq1.gro -n  T0_ACN.ndx -p  T0_ACN.top -o T0_ACN_eq2.tpr  -maxwarn 2  2>&1 | tee eq2_grompp.txt
gmx mdrun -v -deffnm T0_ACN_eq2

Production run
gmx grompp -f prod_run.mdp -c  T0_ACN_eq2.gro -r T0_ACN_eq2.gro -n  T0_ACN.ndx -p  T0_ACN.top -o T0_ACN_md_run.tpr  -maxwarn 2  2>&1 | tee md_run_grompp.txt
gmx mdrun -v -deffnm T0_ACN_md_run
