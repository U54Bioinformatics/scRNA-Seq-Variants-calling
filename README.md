# scRNA-Seq-Variants-calling
1.  Pull out the BAM files from the 10X data.
  betsy_run.py --num_cores 20 --network_png proc12.pdf \
    --input TenXCountResults --input_file proc01 \
    --output DemultiplexedTenXBamFolder --output_file proc11 \
    --mattr demux_cell_file=cell03.txt \
    --submit_to_cluster biocore1_proc11,48,128,proc14.log

Here, proc01 is the output from BETSY, e.g.:
legacy_batch1.10x.counts/
  <sample>/       (cell ranger output)
  <sample>.log
  summary.txt

2.  Create a sample sheet from output BAM files.

  betsy_run.py --network_png samp02.pdf \
    --input BamFolder --input_file proc11 \
    --dattr BamFolder.each_file_contains=original_data_and_name \
    --output DraftSampleGroupFile --output_file samp01.xls

3.  Do the variant calling on the BAM files with GATK.

 A=Mills_and_1000G_gold_standard
  betsy_run.py --num_cores 8 --network_png call02.pdf --receipt call03.txt \
    --input BamFolder --input_file proc11 \
    --dattr BamFolder.aligner=star \
    --input SampleGroupFile --input_file samp01.xls \
    --input ReferenceGenome --input_file genomes/Broad.hg19 \
    --output SimpleVariantMatrix --output_file call01.txt \
    --also_save_highest ManyCallerVCFFolders,call05 \
    --dattr BamFolder.split_n_trim=yes \
    --dattr VCFFolder.caller=gatk \
    --dattr VCFFolder.vartype=snp \
    --dattr SimpleVariantMatrix.caller_suite=single \
    --mattr wgs_or_wes=wes \
    --mattr filter_reads_with_N_cigar=yes \
    --mattr realign_known_sites1=v/$A.indels.b37.vcf.gz \
    --mattr realign_known_sites2=v/1000G_phase1.indels.b37.vcf.gz \
    --mattr recal_known_sites1=v/$A.indels.b37.vcf.gz \
    --mattr recal_known_sites2=v/1000G_phase1.indels.b37.vcf.gz \
    --mattr recal_known_sites3=v/dbsnp
    
    
    
# GATK pipeline used for scRNA-Seq mutation calls on Velocitron
## Please pull from github to make sure you have the latest BETSY 


1.  First step is to demultiplex the BAM file so that you have one BAM file for each cell.

ln -s /data3/U54/COH043/legacy_brst006.10x.counts/ cellranger.hg19

betsy_run.py --num_cores 40 --network_png proc12.pdf \
--input CellRangerCountResults \
--input_file cellranger.hg19/ \
--output DemultiplexedTenXBamFolder \
--output_file proc11 \
--mattr demux_cell_file=cell05.txt 
 
2.  Next step is to generate a sample sheet for the new directory of BAM files.
betsy_run.py  --network_png samp02.pdf \
--input BamFolder --input_file proc11 \
--dattr BamFolder.each_file_contains=original_data_and_name \
--output DraftSampleGroupFile --output_file samp01.xls

3.  This does the variant calling.

A=Mills_and_1000G_gold_standard
FA=genomes/Broad.hg19
ln -s /data/genomidata/genomes/Broad.hg19/ v

betsy_run.py  --num_cores 20 --network_png call02.pdf --receipt call03.txt \
--input BamFolder --input_file proc11 \
--dattr BamFolder.aligner=star \
--input SampleGroupFile --input_file samp01.xls \
--input ReferenceGenome --input_file ${FA} \
--output SimpleVariantMatrix --output_file call01.txt \
--also_save_highest ManyCallerVCFFolders,call05 \
--dattr BamFolder.split_n_trim=yes \
--dattr VCFFolder.caller=gatk \
--dattr VCFFolder.vartype=snp \
--dattr SimpleVariantMatrix.caller_suite=single \
--mattr wgs_or_wes=wes \
--mattr filter_reads_with_N_cigar=yes \
--mattr realign_known_sites1=v/$A.indels.hg19.sites.vcf.gz \
--mattr realign_known_sites2=v/1000G_phase1.indels.hg19.sites.vcf.gz \
--mattr recal_known_sites1=v/$A.indels.hg19.sites.vcf.gz \
--mattr recal_known_sites2=v/1000G_phase1.indels.hg19.sites.vcf.gz \
--mattr recal_known_sites3=v/dbsnp_138.b37.vcf.gz 


This should generate an SVM file (call01.txt) and a directory of VCFs (call05). 


4. Add coverage data
betsy_run.py --network_png mut42.pdf
--input SimpleVariantMatrix --input_file call01.txt
--dattr SimpleVariantMatrix.with_coverage=no
--input BamFolder --input_file proc11
--dattr BamFolder.aligner=bwa_mem
--dattr BamFolder.sorted=coordinate
--dattr BamFolder.indexed=yes
--dattr BamFolder.adapters_trimmed=yes
--dattr BamFolder.has_read_groups=yes
--dattr BamFolder.base_quality_recalibrated=yes
--dattr BamFolder.indel_realigned=yes
--input ReferenceGenome --input_file genomes/Broad.hg19
--input SampleGroupFile --input_file samp01.xls
--output SimpleVariantMatrix --output_file BRST007.gatk.svm.cov.txt
--dattr SimpleVariantMatrix.with_coverage=yes




 
 
