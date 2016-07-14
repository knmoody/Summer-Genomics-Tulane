###pyRAD - Step 1 

* First, we need to create a new params file. You will also need the proper barcode files from Sara copied to your working folder on cypress. You can use scp to do this if you get them on your home computer (see example of using scp to copy to cypress above).

```bash
module load pyrad
pyrad -n
```

* You should see a new file, params.txt. We need to edit this params file (nano params.txt) to read our fastq file for plate 1 and provide it with a barcodes file. In our directory, there are three fastq files that correspond to the three plates of DNA that Sara ran. The barcodes overlap in these three plates, so we will have to run step 1 (demultiplexing) three times. The change to line two reflects this, where we only want it to consider the first plate. Line three specifies the one barcode file we are using (jacana1 corresponds to plate 1). 

* Here is what I have in my params file by copying the commands used in the pyRAD GBS tutorial.

```bash
==** parameter inputs for pyRAD version 3.0.66  **======================== affected step ==
./                        ## 1. Working directory                                 (all)
./C6RL0ANXX_1.fastq.gz    ## 2. Loc. of non-demultiplexed files (if not line 18)  (s1)
./jacana1.barcodes              ## 3. Loc. of barcode file (if not line 18)             (s1)
vsearch                   ## 4. command (or path) to call vsearch (or usearch)    (s3,s6)
muscle                    ## 5. command (or path) to call muscle                  (s3,s7)
TGCAG                     ## 6. Restriction overhang (e.g., C|TGCAG -> TGCAG)     (s1,s2)
20                         ## 7. N processors (parallel)                           (all)
6                         ## 8. Mindepth: min coverage for a cluster              (s4,s5)
4                         ## 9. NQual: max # sites with qual < 20 (or see line 20)(s2)
.85                 ## 10. lowered clust thresh...
gbs                 ## 11. changed datatype to gbs
4                         ## 12. MinCov: min samples in a final locus             (s7)
3                         ## 13. MaxSH: max inds with shared hetero site          (s7)
c85m4p3             ## 14. outprefix...
==== optional params below this line ===================================  affected step ==
                       ## 15.opt.: select subset (prefix* only selector)            (s2-s7)
                       ## 16.opt.: add-on (outgroup) taxa (list or prefix*)         (s6,s7)
                       ## 17.opt.: exclude taxa (list or prefix*)                   (s7)
                       ## 18.opt.: loc. of de-multiplexed data                      (s2)
                       ## 19.opt.: maxM: N mismatches in barcodes (def= 1)          (s1)
                       ## 20.opt.: phred Qscore offset (def= 33)                    (s2)
1                    ## 21. set filter to 1
                       ## 22.opt.: a priori E,H (def= 0.001,0.01, if not estimated) (s5)
                       ## 23.opt.: maxN: max Ns in a cons seq (def=5)               (s5)
8                    ## 24. increased maxH
                       ## 25.opt.: ploidy: max alleles in cons seq (def=2;see docs) (s4,s5)
                       ## 26.opt.: maxSNPs: (def=100). Paired (def=100,100)         (s7)
                       ## 27.opt.: maxIndels: within-clust,across-clust (def. 3,99) (s3,s7)
                       ## 28.opt.: random number seed (def. 112233)              (s3,s6,s7)
                       ## 29.opt.: trim overhang left,right on final loci, def(0,0) (s7)
*                    ## 30. all output formats
                       ## 31.opt.: maj. base call at depth>x<mindepth (def.x=mindepth) (s5)
50                   ## 32. keep fragments longer than 50
                       ## 33.opt.: max stack size (int), def= max(500,mean+2*SD)    (s3)
                       ## 34.opt.: minDerep: exclude dereps with <= N copies, def=1 (s3)
                       ## 35.opt.: use hierarchical clustering (def.=0, 1=yes)      (s6)
                       ## 36.opt.: repeat masking (def.=1='dust' method, 0=no)      (s3,s6)
                       ## 37.opt.: vsearch max threads per job (def.=6; see docs)   (s3,s6)
==== optional: list group/clade assignments below this line (see docs) ==================
```

* During the workshop, there may have been a slightly different params file used, I am not sure. But I was able to use the one above to work and it follows the default GBS parameters listed on the pyRAD tutorial for GBS SE reads.  

####This is a really useful time to review which of these parameters could be adjusted at this step! You can figure this out be looking at which of these parameters affect which step.
   * Lines 2 and 3 of course are used here, because they specify the location of the files being used. 
   * Line 5 specifies the restriction overhang. This differs between different types of protocols used for digesting genomic DNA.
   * Line 19 is the most important to pay attention to. This sets a threshold for how many mismatches the software will allow between the barcode you give it and the sequences it finds in your data. Mismatches exist, because the sequencer can make errors leading to an incorrect base read or not calling any base at all. The default, set to 1, shouuld only include barcodes the algorithm finds with up to 1 base pair mis matches. You could increase this and see how your output changes. 

* In running step 1, we found an issue with the barcodes file, since it was copied from excel and still has tabs instead of spaces. Let's replace tabs and then rename the files with the following commands:

```bash
cat jacana1.barcodes | sed -e 's/['$'\011'' ]\+/ /' | tee jacana1new.barcodes
mv jacana1.barcodes jacana1old.barcodes
mv jacana1new.barcodes jacana1.barcodes
```

* You will need to do this for each of the three barcodes files.

* Now, because we are on the cluster, we need to create a job script to run step one. I've called my script pyrad_1.1.srun. We also modified this script so that we can run step 1 on each plates without overwriting the output stats and fastq folders: 

```bash
#!/bin/bash
#SBATCH --qos=normal
#SBATCH --time=1-0
#SBATCH --verbose    ###        Verbosity logs error information into the error file
#SBATCH --job-name=jacana_pyrad_s1.1 ### Job Name
#SBATCH --nodes=1             ### Node count required for the job
#SBATCH --ntasks-per-node=20   ### Number of tasks to be launched per Node
#SBATCH --output=jacanas.1.1.output.out
#SBATCH --error=jacanas.1.1.error.err
#SBATCH --mail-type=ALL
#SBATCH --mail-user=slipshut@tulane.edu

date
pwd

mkdir w$SLURM_JOBID
cp params1.1.txt w$SLURM_JOBID/params.txt
ln C6RL0ANXX_1_fastq.gz w$SLURM_JOBID
ln jacana1.barcodes w$SLURM_JOBID
cd w$SLURM_JOBID


module load pyrad

pyrad -p params.txt -s 1

date (END) 
```

Summary of what we wrote into this script below the general sbatch commands:

   * In short, we create a working directory labeled with the slurm job ticket ID
   * Next, we copy the params file for plate one into this new folder and call it params.txt
   * Create a hard link to the fastq file in that new folder. We aren't taking up extra space by copying the file itself. 
   * Same for the barcodes file
   * Using cd navigate into the 
   * Run pyrad commands

* Continue running step 1 for plates 2 and 3 by making new params.txt files (e.g. params1.1.txt, params1.2.txt) and running new .srun scripts to include this new files. So you should have three params files and three .srun scripts. The result will be three new directories for each job that you ran. Within these directories are the results of de-multiplexing performed in pyRAD step 1. 

* We were able to successfully run step 1 of pyrad using the abpve script and params.txt files. After this finished we explored the outputs.
```bash
#navigate into the working directory labeled with the slurm ID
ls fastq/ | wc –l 	#tells you how many files you have in this fastq folder
wc -l jacana1.barcodes 	#tells you how many lines are in this file (95 is ok, 96 is ok too – depends on how computer is counting lines)
```

* Once a job is submitted you will have a new folder for that job – you want to keep the right folder, rename it (using the mv command) and get rid of old ones. I renamed this using the "mv" command to have a short description of what params I used (e.g. w249134_GBS_Params_S1.3). Below are a couple tips for exploring the directories we made.   
```bash
ls w[folder number] 	#this should show all the inputs and outputs from this job including a stats and fastq

du --si w[folder number] 	#this is another way to see if it worked, it shows you what’s in the folder and how much space it takes up
```

* You can explore the successful job output folders and navigate into them to explore the fastq and stats outputs

* We saw that line 19 (maxM)in the params file sets levels of more or les stringency – the default is 1 (one mismatch), but you can change this (see above). You can explore how well this method performed by looking at pyRAD out put found in the stats folder generated in step 1. 

```bash
cat stats/s1.sorting.txt | column –t | head 	#This will show you the beginning of the file with number of reads, etc in columns
```

* Your output should look something like this. You can see there is a discrepency between how many reads were found and how many were matched. That difference is how many reads were discarded in this step due to low confidence regarding that read's barcode match. 

```bash
file                  Nreads      cut_found   bar_matched
C6RL0ANXX_1_fastq.gz  267132531   257022443   254647086
sample                true_bar    obs_bars    N_obs
SL_161                AACGCACATT  AACGCACATT  3064528
SL_161                AACGCACATT  AACGCGCATT  33632
SL_161                AACGCACATT  ATCGCACATT  27169
SL_161                AACGCACATT  AACGGACATT  25965
SL_161                AACGCACATT  AACGCACTTT  20967
SL_161                AACGCACATT  AACGAACATT  18723
SL_161                AACGCACATT  AACGCACATG  18034
```