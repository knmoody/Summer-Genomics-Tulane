###Running steps 3 through 7

* These steps largely use similar parameters to run and slight variations in how to run each step are a bit difficult to grasp (for me at this point). So, we can pretty quickly run through them.

*Navigate back to your working folder and make a new directory for step 3 and copy in the old job file and open it for editing. 

```bash
cd /lustre/project/jk/Jacana
mkdir step_3
cp step_2/pyrad_2.srun ./step_3/pyrad_3.srun
cd step_3
nano pyrad_3.srun
```

* The most effective way I found was to edit the job file to make a directory for edits (a directory made last step), then make a local
copy (using 'ln') of all the contents of that folder. Make your .srun file look like mine. Here are kep places I edited the step 2 job
submission file.
   * change job name to _s3
   * change output name to .3
   * change error name to .3
   * After the command `cd w$SLURM_JOBID, I made a directory and made a natural copy to my step 2 results edits folder.
   so what you would do here is change my path to lead to your own step_2/edits directory. 

```bash
#!/bin/bash
#SBATCH --qos=normal
#SBATCH --time=1-0
#SBATCH --verbose    ###        Verbosity logs error information into the error file
#SBATCH --job-name=jacana_pyrad_s3 ### Job Name
#SBATCH --nodes=1             ### Node count required for the job
#SBATCH --ntasks-per-node=20   ### Number of tasks to be launched per Node
#SBATCH --output=jacanas.3.output.out
#SBATCH --error=jacanas.3.error.err
#SBATCH --mail-type=ALL
#SBATCH --mail-user=eenbody@tulane.edu

date
pwd

mkdir w$SLURM_JOBID

cat <<EOF > w$SLURM_JOBID/params.txt
==** parameter inputs for pyRAD version 3.0.66  **======================== affected step ==
./                        ## 1. Working directory                                 (all)
./*C6RL0ANXX_1_fastq.gz   ## 2. Loc. of non-demultiplexed files (if not line 18)  (s1)
./jacana1.barcodes        ## 3. Loc. of barcode file (if not line 18)             (s1)
vsearch                   ## 4. command (or path) to call vsearch (or usearch)    (s3,s6)
muscle                    ## 5. command (or path) to call muscle                  (s3,s7)
TGCAG                     ## 6. Restriction overhang (e.g., C|TGCAG -> TGCAG)     (s1,s2)
20                        ## 7. N processors (parallel)                           (all)
6                         ## 8. Mindepth: min coverage for a cluster              (s4,s5)
4                         ## 9. NQual: max # sites with qual < 20 (or see line 20)(s2)
.85                       ## 10. Wclust: clustering threshold as a decimal        (s3,s6)
gbs                       ## 11. Datatype: rad,gbs,pairgbs,pairddrad,(others:see docs)(all)
4                         ## 12. MinCov: min samples in a final locus             (s7)
3                         ## 13. MaxSH: max inds with shared hetero site          (s7)
c85m4p3                 ## 14. Prefix name for final output (no spaces)         (s7)
==== optional params below this line ===================================  affected step ==
                       ## 15.opt.: select subset (prefix* only selector)            (s2-s7)
                       ## 16.opt.: add-on (outgroup) taxa (list or prefix*)         (s6,s7)
                       ## 17.opt.: exclude taxa (list or prefix*)                   (s7)
/lustre/project/jk/Jacana_pyRAD/step_2/fastq/  ## 18.opt.: loc. of de-multiplexed data                      (s2)
                       ## 19.opt.: maxM: N mismatches in barcodes (def= 1)          (s1)
                       ## 20.opt.: phred Qscore offset (def= 33)                    (s2)
1                       ## 21.opt.: filter: def=0=NQual 1=NQual+adapters. 2=strict   (s2)
                       ## 22.opt.: a priori E,H (def= 0.001,0.01, if not estimated) (s5)
                       ## 23.opt.: maxN: max Ns in a cons seq (def=5)               (s5)
8                       ## 24.opt.: maxH: max heterozyg. sites in cons seq (def=5)   (s5)
                       ## 25.opt.: ploidy: max alleles in cons seq (def=2;see docs) (s4,s5)
                       ## 26.opt.: maxSNPs: (def=100). Paired (def=100,100)         (s7)
                       ## 27.opt.: maxIndels: within-clust,across-clust (def. 3,99) (s3,s7)
                       ## 28.opt.: random number seed (def. 112233)              (s3,s6,s7)
                       ## 29.opt.: trim overhang left,right on final loci, def(0,0) (s7)
*                       ## 30.opt.: output formats: p,n,a,s,v,u,t,m,k,g,* (see docs) (s7)
                       ## 31.opt.: maj. base call at depth>x<mindepth (def.x=mindepth) (s5)
50                       ## 32.opt.: keep trimmed reads (def=0). Enter min length.    (s2)
                       ## 33.opt.: max stack size (int), def= max(500,mean+2*SD)    (s3)
                       ## 34.opt.: minDerep: exclude dereps with <= N copies, def=1 (s3)
                       ## 35.opt.: use hierarchical clustering (def.=0, 1=yes)      (s6)
                       ## 36.opt.: repeat masking (def.=1='dust' method, 0=no)      (s3,s6)
                       ## 37.opt.: vsearch max threads per job (def.=6; see docs)   (s3,s6)
==== optional: list group/clade assignments below this line (see docs) ==================
EOF

cd w$SLURM_JOBID
mkdir edits

ln /lustre/project/jk/Jacana_pyRAD/step_2/w252439_GBS_Params_S2_33Line20/edits/*.edit ./edits

module load pyrad

pyrad -p params.txt -s 3

date
```

* Now run step three using sbatch!

* Steps 4-7 seem like they could be molded into one step, but I am sure there is a reason they are not. Nevertheless, I would just make
a directory called step_4_to_7 or something, and do all your last 4 steps here. As above, use cp to copy in your .srun file from your
step 3 directory into your new directory. Give it a new name and edit is so that you make a new directory called clust.85 (if you used my
params) than has a ln to the clust.85 folder from step 3. For each step 4,5,6,7 you'll have to edit this line to link to the results from
the previous step. As an example, here is my job submission file for step 4:

```bash
!/bin/bash
#SBATCH --qos=normal
#SBATCH --time=1-0
#SBATCH --verbose    ###        Verbosity logs error information into the error file
#SBATCH --job-name=jacana_pyrad_s4 ### Job Name
#SBATCH --nodes=1             ### Node count required for the job
#SBATCH --ntasks-per-node=20   ### Number of tasks to be launched per Node
#SBATCH --output=jacanas.4.output.out
#SBATCH --error=jacanas.4.error.err
#SBATCH --mail-type=ALL
#SBATCH --mail-user=eenbody@tulane.edu

date
pwd

mkdir w$SLURM_JOBID

cat <<EOF > w$SLURM_JOBID/params.txt
==** parameter inputs for pyRAD version 3.0.66  **======================== affected step ==
./                        ## 1. Working directory                                 (all)
./*C6RL0ANXX_1_fastq.gz   ## 2. Loc. of non-demultiplexed files (if not line 18)  (s1)
./jacana1.barcodes        ## 3. Loc. of barcode file (if not line 18)             (s1)
vsearch                   ## 4. command (or path) to call vsearch (or usearch)    (s3,s6)
muscle                    ## 5. command (or path) to call muscle                  (s3,s7)
TGCAG                     ## 6. Restriction overhang (e.g., C|TGCAG -> TGCAG)     (s1,s2)
20                        ## 7. N processors (parallel)                           (all)
6                         ## 8. Mindepth: min coverage for a cluster              (s4,s5)
4                         ## 9. NQual: max # sites with qual < 20 (or see line 20)(s2)
.85                       ## 10. Wclust: clustering threshold as a decimal        (s3,s6)
gbs                       ## 11. Datatype: rad,gbs,pairgbs,pairddrad,(others:see docs)(all)
4                         ## 12. MinCov: min samples in a final locus             (s7)
3                         ## 13. MaxSH: max inds with shared hetero site          (s7)
c85m4p3                 ## 14. Prefix name for final output (no spaces)         (s7)
==== optional params below this line ===================================  affected step ==
                       ## 15.opt.: select subset (prefix* only selector)            (s2-s7)
                       ## 16.opt.: add-on (outgroup) taxa (list or prefix*)         (s6,s7)
                       ## 17.opt.: exclude taxa (list or prefix*)                   (s7)
/lustre/project/jk/Jacana_pyRAD/step_2/fastq/  ## 18.opt.: loc. of de-multiplexed data                      (s2)
                       ## 19.opt.: maxM: N mismatches in barcodes (def= 1)          (s1)
                       ## 20.opt.: phred Qscore offset (def= 33)                    (s2)
1                       ## 21.opt.: filter: def=0=NQual 1=NQual+adapters. 2=strict   (s2)
                       ## 22.opt.: a priori E,H (def= 0.001,0.01, if not estimated) (s5)
                       ## 23.opt.: maxN: max Ns in a cons seq (def=5)               (s5)
8                       ## 24.opt.: maxH: max heterozyg. sites in cons seq (def=5)   (s5)
                       ## 25.opt.: ploidy: max alleles in cons seq (def=2;see docs) (s4,s5)
                       ## 26.opt.: maxSNPs: (def=100). Paired (def=100,100)         (s7)
                       ## 27.opt.: maxIndels: within-clust,across-clust (def. 3,99) (s3,s7)
                       ## 28.opt.: random number seed (def. 112233)              (s3,s6,s7)
                       ## 29.opt.: trim overhang left,right on final loci, def(0,0) (s7)
*                       ## 30.opt.: output formats: p,n,a,s,v,u,t,m,k,g,* (see docs) (s7)
                       ## 31.opt.: maj. base call at depth>x<mindepth (def.x=mindepth) (s5)
50                       ## 32.opt.: keep trimmed reads (def=0). Enter min length.    (s2)
                       ## 33.opt.: max stack size (int), def= max(500,mean+2*SD)    (s3)
                       ## 34.opt.: minDerep: exclude dereps with <= N copies, def=1 (s3)
                       ## 35.opt.: use hierarchical clustering (def.=0, 1=yes)      (s6)
                       ## 36.opt.: repeat masking (def.=1='dust' method, 0=no)      (s3,s6)
                       ## 37.opt.: vsearch max threads per job (def.=6; see docs)   (s3,s6)
==== optional: list group/clade assignments below this line (see docs) ==================
EOF

cd w$SLURM_JOBID
mkdir clust.85

ln /lustre/project/jk/Jacana_pyRAD/step_3/w253012_GBS_Params_Default/clust.85/* ./clust.85

module load pyrad

pyrad -p params.txt -s 4

date
```

* Good luck running this!
