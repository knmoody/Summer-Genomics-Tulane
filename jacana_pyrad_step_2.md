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
cp -Rv --backup=existing --suffix=_plate3 w249134_GBS_Params_S1.1/fastq/*.gz step_2/fastq/
cp -Rv --backup=existing --suffix=_plate3 w249135_GBS_Params_S1.2/fastq/*.gz step_2/fastq/
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

* Run step 2!

```bash
sbatch pyrad_2.srun
```

* Remember you can check the status of your job submission with
```bash
squeue -u eenbody
```
