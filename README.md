# gen_final_proj

## Study Background

eDNA

## Methods

pictures:
upload then
![](./ name of file.png)

link:
[study name](link)

code:

mkdir final-proj 

cp -R /tmp/gen711_project_data/eDNA-fqs/mifish/fastqs/ /home/users/maf1092/final-proj

cp /tmp/gen711_project_data/eDNA-fqs/mifish/GreatBay-Metadata.tsv /home/users/maf1092/final-proj

cp /tmp/gen711_project_data/eDNA-fqs/mifish/Wells-Metadata.tsv /home/users/maf1092/final-proj

cd final-proj/

mkir trimmed_fastqs

qiime tools import \
   --type "SampleData[PairedEndSequencesWithQuality]"  \
   --input-format CasavaOneEightSingleLanePerSampleDirFmt \
   --input-path /home/users/maf1092/final-proj/fastqs/GreatBay \
   --output-path /home/users/maf1092/final-proj/trimmed_fastqs/trimmed_GreatBay

qiime tools import \
   --type "SampleData[PairedEndSequencesWithQuality]"  \
   --input-format CasavaOneEightSingleLanePerSampleDirFmt \
   --input-path /home/users/maf1092/final-proj/fastqs/Wells \
   --output-path /home/users/maf1092/final-proj/trimmed_fastqs/trimmed_Wells

qiime cutadapt trim-paired \
    --i-demultiplexed-sequences /home/users/maf1092/final-proj/trimmed_fastqs/trimmed_GreatBay.qza \
    --p-cores 4 \
    --p-front-f GTCGGTAAAACTCGTGCCAGC \
    --p-front-r CATAGTGGGGTATCTAATCCCAGTTTG \
    --p-discard-untrimmed \
    --p-match-adapter-wildcards \
    --verbose \
    --o-trimmed-sequences /home/users/maf1092/final-proj/trimmed_fastqs/clean-trimmed_GreatBay.qza

qiime cutadapt trim-paired \
    --i-demultiplexed-sequences /home/users/maf1092/final-proj/trimmed_fastqs/trimmed_Wells.qza \
    --p-cores 4 \
    --p-front-f GTCGGTAAAACTCGTGCCAGC \
    --p-front-r CATAGTGGGGTATCTAATCCCAGTTTG \
    --p-discard-untrimmed \
    --p-match-adapter-wildcards \
    --verbose \
    --o-trimmed-sequences /home/users/maf1092/final-proj/trimmed_fastqs/clean-trimmed_Wells.qza

qiime demux summarize \
    --i-data /home/users/maf1092/final-proj/trimmed_fastqs/clean-trimmed_GreatBay.qza \
    --o-visualization  /home/users/maf1092/final-proj/trimmed_fastqs/demux-summ_GreatBay.qzv 

qiime demux summarize \
    --i-data /home/users/maf1092/final-proj/trimmed_fastqs/clean-trimmed_Wells.qza \
    --o-visualization  /home/users/maf1092/final-proj/trimmed_fastqs/demux-summ_Wells.qzv

mkdir denoising

qiime dada2 denoise-paired \
    --i-demultiplexed-seqs /home/users/maf1092/final-proj/trimmed_fastqs/clean-trimmed_GreatBay.qza  \
    --p-trunc-len-f  120 \
    --p-trunc-len-r 115 \
    --p-trim-left-f 0 \
    --p-trim-left-r 0 \
    --p-n-threads 4 \
    --o-denoising-stats /home/users/maf1092/final-proj/denoising/denoising_GreatBay.qza \
    --o-table /home/users/maf1092/final-proj/denoising/feature_table_GreatBay.qza \
    --o-representative-sequences /home/users/maf1092/final-proj/denoising/rep-seqs_GreatBay.qza


qiime dada2 denoise-paired \
    --i-demultiplexed-seqs /home/users/maf1092/final-proj/trimmed_fastqs/clean-trimmed_Wells.qza  \
    --p-trunc-len-f  120 \
    --p-trunc-len-r 115 \
    --p-trim-left-f 0 \
    --p-trim-left-r 0 \
    --p-n-threads 4 \
    --o-denoising-stats /home/users/maf1092/final-proj/denoising/denoising_Wells.qza \
    --o-table /home/users/maf1092/final-proj/denoising/feature_table_Wells.qza \
    --o-representative-sequences /home/users/maf1092/final-proj/denoising/rep-seqs_Wells.qza

qiime metadata tabulate \
    --m-input-file /home/users/maf1092/final-proj/denoising/denoising_GreatBay.qza \
    --o-visualization /home/users/maf1092/final-proj/denoising/denoising_GreatBay.qzv

qiime metadata tabulate \
    --m-input-file /home/users/maf1092/final-proj/denoising/denoising_Wells.qza \
    --o-visualization /home/users/maf1092/final-proj/denoising/denoising_Wells.qzv

qiime feature-table tabulate-seqs \
        --i-data /home/users/maf1092/final-proj/denoising/rep-seqs_GreatBay.qza \
        --o-visualization /home/users/maf1092/final-proj/denoising/rep-seqs_GreatBay.qzv

qiime feature-table tabulate-seqs \
        --i-data /home/users/maf1092/final-proj/denoising/rep-seqs_Wells.qza \
        --o-visualization /home/users/maf1092/final-proj/denoising/rep-seqs_Wells.qzv

cd /home/users/maf1092/final-proj/

mkdir taxonomy

qiime feature-classifier classify-sklearn \
  --i-classifier /home/users/maf1092/final-proj/ref-database/mitofish-classifier.qza \
  --i-reads /home/users/maf1092/final-proj/denoising/rep-seqs_GreatBay.qza \
  --o-classification /home/users/maf1092/final-proj/taxonomy/GreatBay_taxonomy.qza

qiime feature-classifier classify-sklearn \
  --i-classifier /home/users/maf1092/final-proj/ref-database/mitofish-classifier.qza \
  --i-reads /home/users/maf1092/final-proj/denoising/rep-seqs_Wells.qza \
  --o-classification /home/users/maf1092/final-proj/taxonomy/Wells_taxonomy.qza

qiime metadata tabulate \
  --m-input-file /home/users/maf1092/final-proj/taxonomy/GreatBay_taxonomy.qza \
  --o-visualization /home/users/maf1092/final-proj/taxonomy/GreatBay_taxonomy.qzv

qiime metadata tabulate \
  --m-input-file /home/users/maf1092/final-proj/taxonomy/Wells_taxonomy.qza \
  --o-visualization /home/users/maf1092/final-proj/taxonomy/Wells_taxonomy.qzv

###Barplot

mkdir barplot

qiime feature-table filter-samples \
  --i-table /home/users/maf1092/final-proj/denoising/feature_table_GreatBay.qza \
  --m-metadata-file /home/users/maf1092/final-proj/mifish-metadata.tsv \
  --o-filtered-table /home/users/maf1092/final-proj/barplot/GreatBay_feature_table_filtered.qza

qiime feature-table filter-samples \
  --i-table /home/users/maf1092/final-proj/denoising/feature_table_Wells.qza \
  --m-metadata-file /home/users/maf1092/final-proj/mifish-metadata.tsv \
  --o-filtered-table /home/users/maf1092/final-proj/barplot/Wells_feature_table_filtered.qza

mkdir merge-rep-seqs

qiime feature-table merge-seqs \
   --i-data /home/users/maf1092/final-proj/denoising/rep-seqs_GreatBay.qza \
   --i-data /home/users/maf1092/final-proj/denoising/rep-seqs_Wells.qza \
   --o-merged-data /home/users/maf1092/final-proj/merge-rep-seqs/BOTH_rep-seqs.qza

qiime feature-table merge \
  --i-tables /home/users/maf1092/final-proj/barplot/GreatBay_feature_table_filtered.qza \
  --i-tables /home/users/maf1092/final-proj/barplot/Wells_feature_table_filtered.qza \
  --o-merged-table /home/users/maf1092/final-proj/merge-rep-seqs/BOTH_feature_table.qza

qiime feature-classifier classify-sklearn \
  --i-classifier /home/users/maf1092/final-proj/ref-database/mitofish-classifier.qza \
  --i-reads /home/users/maf1092/final-proj/merge-rep-seqs/BOTH_rep-seqs.qza \
  --o-classification /home/users/maf1092/final-proj/merge-rep-seqs/BOTH_taxonomy.qza

mv merge-rep-seqs/ merged-data/

qiime taxa barplot \
     --i-table /home/users/maf1092/final-proj/merged-data/BOTH_feature_table.qza \
     --m-metadata-file /home/users/maf1092/final-proj/mifish-metadata.tsv \
     --i-taxonomy /home/users/maf1092/final-proj/merged-data/BOTH_taxonomy.qza \
     --o-visualization /home/users/maf1092/final-proj/merged-data/BOTH_barplot.qzv

qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences /home/users/maf1092/final-proj/merged-data/BOTH_rep-seqs.qza \
  --o-alignment /home/users/maf1092/final-proj/merged-data/BOTH_alignments \
  --o-masked-alignment /home/users/maf1092/final-proj/merged-data/BOTH_masked-alignment \
  --o-tree /home/users/maf1092/final-proj/merged-data/BOTH_unrooted-tree \
  --o-rooted-tree /home/users/maf1092/final-proj/merged-data/BOTH_rooted-tree \
  --p-n-threads 4

