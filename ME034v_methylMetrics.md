I've grabbed some metrics for the enzymatic methylSEQ samples - these were output \
by FastQC and the bsmap log file

![image](https://user-images.githubusercontent.com/43852873/159178658-16b9313f-53d9-4dbe-aafc-bc0132e2839f.png)

To compile additional metrics I need to run the next few steps in the pipeline and \
scrape from the log files, BUT I'm getting stuck at step 04.  I think something is going \
wrong in one of the sam/bam formatting steps in step 03 (?)

Here are steps 03 and 04 as run

03_fix_sort_bsmap_DRM.sh
````
#!/bin/bash -l
#SBATCH --time=72:00:00
#SBATCH --ntasks=1
#SBATCH --mem=32g
#SBATCH --tmp=10g
#SBATCH --job-name=2021_02_05_04_filter
#SBATCH --mail-type=ALL
#SBATCH --mail-user=read0094@umn.edu

########## Modules #################

#bsmap requires samtools < 1.0.0
# module load samtools/0.1.18 # no longer an available module
module load samtools
#PATH=~/software/bsmap-2.74/samtools:$PATH


########## Set up dirs #################
cd bsmapped

########## Run #################

# make bam
samtools view -b PHJ01.sam -o PHJ01.bam
samtools view -b PHJ02.sam -o PHJ02.bam
samtools view -b PHJ03.sam -o PHJ03.bam
samtools view -b PHJ04.sam -o PHJ04.bam
samtools view -b PHJ05.sam -o PHJ05.bam
samtools view -b PHJ06.sam -o PHJ06.bam
samtools view -b PHJ07.sam -o PHJ07.bam


# sort by read name (needed for fixsam)
samtools sort -n PHJ01.bam -o PHJ01_nameSrt.bam
samtools sort -n PHJ02.bam -o PHJ02_nameSrt.bam
samtools sort -n PHJ03.bam -o PHJ03_nameSrt.bam
samtools sort -n PHJ04.bam -o PHJ04_nameSrt.bam
samtools sort -n PHJ05.bam -o PHJ05_nameSrt.bam
samtools sort -n PHJ06.bam -o PHJ06_nameSrt.bam
samtools sort -n PHJ07.bam -o PHJ07_nameSrt.bam

# fix mate pairs
samtools fixmate PHJ01_nameSrt.bam  PHJ01_nameSrt_fixed.bam
samtools fixmate PHJ02_nameSrt.bam  PHJ02_nameSrt_fixed.bam
samtools fixmate PHJ03_nameSrt.bam  PHJ03_nameSrt_fixed.bam
samtools fixmate PHJ04_nameSrt.bam  PHJ04_nameSrt_fixed.bam
samtools fixmate PHJ05_nameSrt.bam  PHJ05_nameSrt_fixed.bam
samtools fixmate PHJ06_nameSrt.bam  PHJ06_nameSrt_fixed.bam
samtools fixmate PHJ07_nameSrt.bam  PHJ07_nameSrt_fixed.bam



# co-ordinate sort
#samtools sort PHJ07_nameSrt_fixed.bam -o PHJ07_sorted.bam

samtools sort PHJ01_nameSrt_fixed.bam  -o PHJ01_sorted.bam
samtools sort PHJ02_nameSrt_fixed.bam  -o PHJ02_sorted.bam
samtools sort PHJ03_nameSrt_fixed.bam  -o PHJ03_sorted.bam
samtools sort PHJ04_nameSrt_fixed.bam  -o PHJ04_sorted.bam
samtools sort PHJ05_nameSrt_fixed.bam  -o PHJ05_sorted..bam
samtools sort PHJ06_nameSrt_fixed.bam  -o PHJ06_sorted.bam
samtools sort PHJ07_nameSrt_fixed.bam  -o PHJ07_sorted.bam

# index
samtools index PHJ01_sorted.bam
samtools index PHJ02_sorted.bam
samtools index PHJ03_sorted.bam
samtools index PHJ04_sorted.bam
samtools index PHJ05_sorted.bam
samtools index PHJ06_sorted.bam
samtools index PHJ07_sorted.bam
`````

Outputs from step03 (only PHJ01 shown)
`````
-rw-------. 1 read0094 springer  17G Mar  9 12:13 PHJ01.bam
-rw-------. 1 read0094 springer  17G Mar 10 16:51 PHJ01_nameSrt.bam
-rw-------. 1 read0094 springer  17G Mar 11 02:30 PHJ01_nameSrt_fixed.bam
-rw-------. 1 read0094 springer  79G Mar  8 15:10 PHJ01.sam
-rw-------. 1 read0094 springer  11G Mar 11 12:34 PHJ01_sorted.bam
-rw-------. 1 read0094 springer 1.2M Mar 11 18:38 PHJ01_sorted.bam.bai
`````

Step04 as called
`````
#!/bin/bash -l
#SBATCH --time=40:00:00
#SBATCH --ntasks=1
#SBATCH --mem=32g
#SBATCH --tmp=10g
#SBATCH --job-name=DRM_filter
#SBATCH --mail-type=ALL
#SBATCH --mail-user=read0094@umn.edu



########## Modules #################

#module load python2/2.7.8
module load java
#module load bedtools
module load samtools
module load bamtools
#bsmap requires samtools < 1.0.0
#module load samtools/0.1.18
#PATH=~/software/bsmap-2.74/samtools:$PATH

########## Set up dirs #################

#get job ID
#use sed, -n supression pattern space, then 'p' to print item number {PBS_ARRAYID} eg 2 from {list}
echo sample being mapped is PHJ02

#cd analysis
mkdir -p bsmapped_filtered

#PHJ01
java -jar /home/springer/read0094/Software/picard.jar MarkDuplicates \
I=bsmapped/PHJ01_sorted.bam \
O=bsmapped_filtered/PHJ01_sorted_MarkDup.bam \
METRICS_FILE=bsmapped_filtered/PHJ01_MarkDupMetrics.txt \
ASSUME_SORTED=true \
REMOVE_DUPLICATES=true

# keep properly paired reads using bamtools package
# note that some reads marked as properly paired by bsmap actually map to different chromosomes
bamtools filter \
-isMapped true \
-isPaired true \
-isProperPair true \
-in bsmapped_filtered/PHJ01_sorted_MarkDup.bam \
-out bsmapped_filtered/PHJ01_sorted_MarkDup_pairs.bam

# clip overlapping reads using bamUtils package
bam clipOverlap \
--in bsmapped_filtered/PHJ01_sorted_MarkDup_pairs.bam \
--out bsmapped_filtered/PHJ01_sorted_MarkDup_pairs_clipOverlap.bam \
--stats

samtools index bsmapped_filtered/PHJ01_sorted_MarkDup_pairs_clipOverlap.bam
`````

Error in step04
`````
picard.sam.markduplicates.MarkDuplicates done. Elapsed time: 0.01 minutes.
Runtime.totalMemory()=519110656
To get help, see http://broadinstitute.github.io/picard/index.html#GettingHelp
Exception in thread "main" htsjdk.samtools.SAMFormatException: SAM validation error: ERROR::INVALID_FLAG_MATE_UNMAPPED:Record 1, Read name A00223:416:HTMM3DRXX:2:1110:7798:28526, Mate unmapped flag should not be set for unpaired read.
`````

I ran 'flagstat' on the PHJ01.bam file and it does look like there are recognized pairs here -- there are \
also additional metrics that I could add...
`````
samtools flagstat PHJ01.bam 
229752068 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
229752068 + 0 mapped (100.00% : N/A)
229752068 + 0 paired in sequencing
117741501 + 0 read1
112010567 + 0 read2
200611020 + 0 properly paired (87.32% : N/A)
211615064 + 0 with itself and mate mapped
18137004 + 0 singletons (7.89% : N/A)
1678540 + 0 with mate mapped to a different chr
1678540 + 0 with mate mapped to a different chr (mapQ>=5)
`````

I'm running the flagstat on what should be the step04 first input - and things still look good \
at least it looks like pairs are still recognized - 
`````
samtools flagstat PHJ01_sorted.bam
229752068 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
229752068 + 0 mapped (100.00% : N/A)
211615064 + 0 paired in sequencing
105807532 + 0 read1
105807532 + 0 read2
200610898 + 0 properly paired (94.80% : N/A)
211615064 + 0 with itself and mate mapped
0 + 0 singletons (0.00% : N/A)
1678648 + 0 with mate mapped to a different chr
1678648 + 0 with mate mapped to a different chr (mapQ>=5)
`````

But the head of these files doesn't look right
`````
samtools view -H PHJ01.sam
@HD	VN:1.0
@SQ	SN:Chr01	LN:42132932
@SQ	SN:Chr02	LN:48726069
@SQ	SN:Chr03	LN:49814079
@SQ	SN:Chr04	LN:39642072
@SQ	SN:Chr05	LN:46382547
@SQ	SN:Chr06	LN:36113639
@SQ	SN:Chr07	LN:34998066
@SQ	SN:Chr08	LN:42437421
@SQ	SN:Chr09	LN:56635340
@SQ	SN:ChrCP	LN:134903
@SQ	SN:ChrUn1	LN:149356
@SQ	SN:ChrUn2	LN:523348
@SQ	SN:phix	LN:5386
@PG	ID:BSMAP_2.74

samtools view -H PHJ01.bam
@HD	VN:1.0
@SQ	SN:Chr01	LN:42132932
@SQ	SN:Chr02	LN:48726069
@SQ	SN:Chr03	LN:49814079
@SQ	SN:Chr04	LN:39642072
@SQ	SN:Chr05	LN:46382547
@SQ	SN:Chr06	LN:36113639
@SQ	SN:Chr07	LN:34998066
@SQ	SN:Chr08	LN:42437421
@SQ	SN:Chr09	LN:56635340
@SQ	SN:ChrCP	LN:134903
@SQ	SN:ChrUn1	LN:149356
@SQ	SN:ChrUn2	LN:523348
@SQ	SN:phix	LN:5386
@PG	ID:BSMAP_2.74

samtools view -H PHJ01_sorted.bam
@HD	VN:1.0	SO:coordinate
@SQ	SN:Chr01	LN:42132932
@SQ	SN:Chr02	LN:48726069
@SQ	SN:Chr03	LN:49814079
@SQ	SN:Chr04	LN:39642072
@SQ	SN:Chr05	LN:46382547
@SQ	SN:Chr06	LN:36113639
@SQ	SN:Chr07	LN:34998066
@SQ	SN:Chr08	LN:42437421
@SQ	SN:Chr09	LN:56635340
@SQ	SN:ChrCP	LN:134903
@SQ	SN:ChrUn1	LN:149356
@SQ	SN:ChrUn2	LN:523348
@SQ	SN:phix	LN:5386
@PG	ID:BSMAP_2.74
`````
