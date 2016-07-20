###Step 2

####*Consolidating results from step 1 into one directory*

* Alright so now that we have three directories that contain fastq folder containing all of our de-multiplexed reads, we have to move these all into one directory. There is almost certainly some easy automated way of doing this, but I went for brute force. 

* The first thing I did make a new folder called step_2. Then I copied in the params file into it and rename it just params.txt (because we will only need one from here on out). Also copy in one of the job script files and call it step 2. 

```bash
mkdir step_2
mkdir step_2/fastq
cp params1.1.txt step_2/params.txt
cp pyrad_1.1.srun step_2/pyrad_2.srun
```

* What I did next is probably not the best way to do it, but it is described below. Graham offered an alternative in the next chunk.

####*Option one for copying files*

```bash
cp -Rv --backup=existing --suffix=_plate1 w249134_GBS_Params_S1.1/fastq/*.gz step_2/fastq/
cp -Rv --backup=existing --suffix=_plate2 w249135_GBS_Params_S1.2/fastq/*.gz step_2/fastq/
cp -Rv --backup=existing --suffix=_plate3 w249136_GBS_Params_S1.3/fastq/*.gz step_2/fastq/
```

* This copies everything that was in my output directory from running step 1 for plate one into the fastq directory in the step_2 folder.
   * -R is recursive, so it copies everythign within that directory. Notice how I used *.gz to copy all .gz files
   * v is verbose so it will tell me exactly what is copying
   * --backup=existing says to do something with existing files
   * --suffix adds on a custom string to the end of any files that are duplicates in the destination directory

* Now, this is a poor mans solution, becuase now I have several files that end in .fq.gz_plate3 and every file needs to end in .fq.gz. So Rather than try to solve this problem and do it different, I just used mv to change the name of those repeats (all of this in my new setup_2/fastq). Examples below:

```bash
  mv blank_R1.fq.gz_plate3 blank_R1_plate3.fq.gz
  mv LSUMZ_25287_R1.fq.gz_plate3 LSUMZ_25287_R1_plate3.fq.gz 
  mv MJM_7661_R1.fq.gz_plate3 MJM_7661_R1_plate3.fq.gz
  mv MJM_8247_R1.fq.gz_plate3 MJM_8247_R1_plate3.fq.gz
  mv SEL_STRI_25_R1.fq.gz_plate3 SEL_STRI_25_R1_plate3.fq.gz
  mv SL_161_R1.fq.gz_plate3 SL_161_R1_plate3.fq.gz
```

####*Option two for copying files*

* Here is Graham's solution. This will search all of the working folders you made on cypress and pump out all the duplicates it finds.

```bash
ls w24*/fastq/* fastq/* | sed -e 's%.*/%%'|sort|uniq -c |sed -e '/^ \+1/ d'
```

* And here is the result

```bash
 3 blank_R1.fq.gz
      2 LSUMZ_25287_R1.fq.gz
      2 MJM_7661_R1.fq.gz
      2 MJM_8247_R1.fq.gz
      2 SEL_STRI_25_R1.fq.gz
      2 SL_161_R1.fq.gz
```

* Great now we know which files have duplicates. Now we need to rename them. First gives you the directory where the file is, then you use mv to rename that file where it is. Do this for every file that is duplicated. Note, your directory names will differ from that below. 

```bash
find w24*/fastq/ -name LSUMZ_25287_R1.fq.gz 
w243517/fastq/LSUMZ_25287_R1.fq.gz #output
mv w243517/fastq/LSUMZ_25287_R1.fq.gz w243517/fastq/LSUMZ_25287.3_R1.fq.gz
```

* Once every repeat is renamed, you can copy them over. Graham suggested using a hardlink here, rather than using cp. This means they will only phyiscally be in one place.

```bash
ln w24*/fastq/* fastq/* step2/fastq
```

####*Setting up the job script file*

* Yay! Now we have a fastq folder filled with all the de multiplexed files! I am sure there was a better way to do this...

* Next, lets make a new job script. While preparing this, I noted that I could not make a hard link to the fastq directory (just cant do this with directories). Rather than spending time trying to find an alternative, I just altered the params.txt slightly (using nano params.txt). On line 18 you can specify the location of the directory where all your demultiplexed files are. Here is what that line looks like for me. 

```bash
/lustre/project/jk/Jacana_pyRAD/step_2/fastq/  ## 18.opt.: loc. of de-multiplexed data                      (s2)
```
* You don't have to change anything else in the params file at this point. However, this is a great opportunity to see what there is that could be changed!
   * Line 15: You could only select a subset of your data, for some reason, and this would help you do that
   * Line 20: In this step will will be filtering low quality data based on the phred score. This is an important line where you specify how high you want to set this bar for filtering. Good place to go back through and run again to see how this changes your data!
   * Line 21: This gives you several options for filtering based on quality or quality and adapters. We have adapters (when wouldn't you?) so we want to filter by both NQUAL+adapters, so this should be set to 1. Mine was already set this way, but you may need to set it yourself. I am not sure what 2 would do, but I bet its in the general tutorial. 
   * Line 32: This allows you to set the minimum length of reads you want to keep after trimming has been done. The default in the GBS tutorial is 50, so you should set it as such. Another good place to tweek when exploring your data. 

* Next I did some small tweeking to the job script (nano pyrad_2.srun), which now looks like this:

```bash
#!/bin/bash
#SBATCH --qos=normal
#SBATCH --time=1-0
#SBATCH --verbose    ###        Verbosity logs error information into the error file
#SBATCH --job-name=jacana_pyrad_s2 ### Job Name
#SBATCH --nodes=1             ### Node count required for the job
#SBATCH --ntasks-per-node=20   ### Number of tasks to be launched per Node
#SBATCH --output=jacanas.2.output.out
#SBATCH --error=jacanas.2.error.err
#SBATCH --mail-type=ALL
#SBATCH --mail-user=eenbody@tulane.edu

date
pwd

mkdir w$SLURM_JOBID
cp params.txt w$SLURM_JOBID/params.txt
cd w$SLURM_JOBID

module load pyrad

pyrad -p params.txt -s 2

date
```

####*NEW BETTER JOB FILE TO USE*

* This job script file creates a params file within the jobscript itself, so you don't have to keep carrying the params file around when making new working directories (note: phred offset here set to 43, default is 33). 

```bash
#!/bin/bash
#SBATCH --qos=normal
#SBATCH --time=1-0
#SBATCH --verbose    ###        Verbosity logs error information into the error file
#SBATCH --job-name=jacana_pyrad_s2 ### Job Name
#SBATCH --nodes=1             ### Node count required for the job
#SBATCH --ntasks-per-node=20   ### Number of tasks to be launched per Node
#SBATCH --output=jacanas.2.output.out
#SBATCH --error=jacanas.2.error.err
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
43                       ## 20.opt.: phred Qscore offset (def= 33)                    (s2)
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

module load pyrad

pyrad -p params.txt -s 2

date
```

* Run step 2!

```bash
sbatch pyrad_2.srun
```

* Remember you can check the status of your job submission with
```bash
squeue -u eenbody
```
