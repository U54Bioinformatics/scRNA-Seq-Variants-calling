# scRNA-Seq-Variants-calling
1.  Pull out the BAM files from the Ten X data.
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
