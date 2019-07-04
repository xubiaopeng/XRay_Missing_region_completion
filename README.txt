Recovering the missing regions in crystal structures from the NMR measurement data using matrix completion method

*****************************

1. Run Recover_missing_str.m (In code L22-31, select a test protein), and a raw structure    (.pdb) will be produced under the directory;

2. Energy minimization.

   1) Install Gromacs-5.0;
   2) Put the file 'ions.mdp' and 'minim.mdp' under the directory;
   3) running following command step by step:

   echo 6 | gmx pdb2gmx -f $1 -o $1_processed.gro -water tip3p -ignh -missing # $1 is your        input pdb name 
   gmx editconf -f $1_processed.gro -o $1_newbox.gro -c -d 1.0 -bt cubic
   gmx solvate -cp $1_newbox.gro -cs spc216.gro -o $1_solv.gro -p topol.top
   gmx grompp -f ions.mdp -c $1_solv.gro -p topol.top -o ions.tpr
   echo 13 | gmx genion -s ions.tpr -o $1_solv_ions.gro -p topol.top -pname NA -nname CL -        neutral -conc 0.1
   gmx grompp -f minim.mdp -c $1_solv_ions.gro -p topol.top -o em.tpr 
   gmx mdrun -deffnm em
   echo 1 | gmx trjconv -f em.gro -s em.tpr -pbc mol -ur compact -o protein_$1.gro


   For example, if the input name is '1sy9.pdb', the commands are as following:

   echo 6 | gmx pdb2gmx -f 1sy9.pdb -o 1sy9_processed.gro -water tip3p -ignh -missing 
   gmx editconf -f 1sy9_processed.gro -o 1sy9_newbox.gro -c -d 1.0 -bt cubic
   gmx solvate -cp 1sy9_newbox.gro -cs spc216.gro -o 1sy9_solv.gro -p topol.top
   gmx grompp -f ions.mdp -c 1sy9_solv.gro -p topol.top -o ions.tpr
   echo 13 | gmx genion -s ions.tpr -o 1sy9_solv_ions.gro -p topol.top -pname NA -nname CL -      neutral -conc 0.1
   gmx grompp -f minim.mdp -c 1sy9_solv_ions.gro -p topol.top -o em.tpr 
   gmx mdrun -deffnm em
   echo 1 | gmx trjconv -f em.gro -s em.tpr -pbc mol -ur compact -o protein_1sy9.gro
   gmx editconf -f protein_1sy9.gro -o protein_EM_1sy9.pdb -label A


   # Then a new structure (protein_EM_1sy9.pdb) will produced.

3. Run Matlab function file: CombineMissX(ComPDBID,XrayPDBID,OutPDBID)

   For example: CombineMissX('protein_EM_1sy9.pdb','1ctr.pdb','1ctrout.pdb')

4. Energy minimization in the missing regions

   echo 6 | gmx pdb2gmx -f $1.pdb -o $1_processed.gro -water tip3p -ignh -missing
   gmx editconf -f $1_processed.gro -o $1_newbox.gro -c -d 1.0 -bt cubic
   gmx solvate -cp $1_newbox.gro -cs spc216.gro -o $1_solv.gro -p topol.top
   make_ndx -f $1_processed.gro -o index.ndx
   ri start_index-end_index        # Known residue index
   q       #  quit
   gmx genrestr -f $1_processed.gro -n index.ndx -o posres_1.itp -fc 1000 1000 1000
   10
   gmx grompp -f ions.mdp -c $1_solv.gro -p topol.top -o ions.tpr
   # Add 'define=-DPOSRES' at the beginning of the file 'minim.mdp', and replace 'posre.itp' with 'posres_1.itp'
   echo 13 | gmx genion -s ions.tpr -o $1_solv_ions.gro -p topol.top -pname NA -nname CL -   neutral -conc 0.1
   gmx grompp -f minim.mdp -c $1_solv_ions.gro -p topol.top -o em.tpr
   gmx mdrun -deffnm em
   echo 1 | gmx trjconv -f em.gro -s em.tpr -pbc mol -ur compact -o protein_$1.gro
   gmx editconf -f protein_$1.gro -o protein_EM_$1.pdb -label A  # 'protein_EM_$1.pdb' is the final structure.


   For example, if the input name is '1ctrout.pdb', the commands are as following:
 
   echo 6 | gmx pdb2gmx -f 1ctrout.pdb -o 1ctrout_processed.gro -water tip3p -ignh -missing
   gmx editconf -f 1ctrout_processed.gro -o 1ctrout_newbox.gro -c -d 1.0 -bt cubic
   gmx solvate -cp 1ctrout_newbox.gro -cs spc216.gro -o 1ctrout_solv.gro -p topol.top
   make_ndx -f 1ctrout_processed.gro -o index.ndx
   ri 1-74 | ri 81-147  
   q
   gmx genrestr -f 1ctrout_processed.gro -n index.ndx -o posres_1.itp -fc 1000 1000 1000
   10
   gmx grompp -f ions.mdp -c 1ctrout_solv.gro -p topol.top -o ions.tpr
   # Add 'define=-DPOSRES' at the beginning of the file 'minim.mdp', and replace 'posre.itp' with 'posres_1.itp'
   echo 13 | gmx genion -s ions.tpr -o 1ctrout_solv_ions.gro -p topol.top -pname NA -nname CL -neutral -conc 0.1
   gmx grompp -f minim.mdp -c 1ctrout_solv_ions.gro -p topol.top -o em.tpr
   gmx mdrun -deffnm em
   echo 1 | gmx trjconv -f em.gro -s em.tpr -pbc mol -ur compact -o protein_1ctrout.gro
   gmx editconf -f protein_1ctrout.gro -o protein_EM_1ctrout.pdb -label A  
   
   # 'protein_EM_1ctrout.pdb' is the final structure. 

      

  