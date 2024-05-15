# gen_final_proj

## Study Background

   This study used samples taken from multiple different sources such as the Lamprey river, Oyster River, and the Squamscott River. and analyzed through eDNA metabarcoding.
   Metabarcoding is taking samples from the environment where DNA can be extracted and then amplifying it to readable levels using PCR primers.
   This study compared fish species using the MiFish locus (or 12s) primers which are designed specifically for marine species.
   The data was analyzed using qiime2.

## Methods

code:

mkdir final-proj 

cp -R /tmp/gen711_project_data/eDNA-fqs/mifish/fastqs/ /home/users/maf1092/final-proj

cp /tmp/gen711_project_data/eDNA-fqs/mifish/GreatBay-Metadata.tsv /home/users/maf1092/final-proj

cp /tmp/gen711_project_data/eDNA-fqs/mifish/Wells-Metadata.tsv /home/users/maf1092/final-proj

cp /tmp/new_meta.tsv /home/users/maf1092/final-proj

cd final-proj/

mkdir ref-database

cp /tmp/gen711_project_data/eDNA-fqs/mifish/ref-database/mitofish-classifier.qza /home/users/maf1092/final-proj/ref-database/2mitofish-classifier.qza

mkir trimmed_fastqs

### Filter Raw Reads

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

### Denoising

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

mkdir taxonomy merged-data

### Taxonomy

qiime feature-table merge-seqs \
   --i-data /home/users/maf1092/final-proj/denoising/rep-seqs_GreatBay.qza \
   --i-data /home/users/maf1092/final-proj/denoising/rep-seqs_Wells.qza \
   --o-merged-data /home/users/maf1092/final-proj/merged-data/BOTH_rep-seqs.qza

qiime feature-table merge \
  --i-tables /home/users/maf1092/final-proj/denoising/feature_table_GreatBay.qza \
  --i-tables /home/users/maf1092/final-proj/denoising/feature_table_Wells.qza \
  --o-merged-table /home/users/maf1092/final-proj/merged-data/combined_feature_table.qza

qiime feature-classifier classify-sklearn \
  --i-classifier  /home/users/maf1092/final-proj/ref-database/2mitofish-classifier.qza \
  --i-reads /home/users/maf1092/final-proj/merged-data/BOTH_rep-seqs.qza \
  --o-classification /home/users/maf1092/final-proj/taxonomy/classify-sklearn-taxonomy

### Barplot and Charts

mkdir new-barplot new-phylo-tree

qiime taxa barplot \
     --i-table /home/users/maf1092/final-proj/merged-data/combined_feature_table.qza \
     --i-taxonomy /home/users/maf1092/final-proj/taxonomy/classify-sklearn-taxonomy.qza \
     --o-visualization /home/users/maf1092/final-proj/new-barplot/BOTH-barplot.qzv

qiime feature-table filter-samples \
  --i-table /home/users/maf1092/final-proj/merged-data/combined_feature_table.qza \
  --m-metadata-file /home/users/maf1092/final-proj/new_meta.tsv \
  --o-filtered-table /home/users/maf1092/final-proj/merged-data/combined_filter_feature_table.qza

qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences /home/users/maf1092/final-proj/merged-data/BOTH_rep-seqs.qza \
  --o-alignment /home/users/maf1092/final-proj/new-phylo-tree/alignments \
  --o-masked-alignment /home/users/maf1092/final-proj/new-phylo-tree/masked-alignment \
  --o-tree /home/users/maf1092/final-proj/new-phylo-tree/unrooted-tree \
  --o-rooted-tree /home/users/maf1092/final-proj/new-phylo-tree/rooted-tree \
  --p-n-threads 4

qiime diversity core-metrics-phylogenetic \
  --i-phylogeny /home/users/maf1092/final-proj/new-phylo-tree/rooted-tree.qza \
  --i-table /home/users/maf1092/final-proj/merged-data/combined_filter_feature_table.qza \
  --p-sampling-depth 500 \
  --m-metadata-file /home/users/maf1092/final-proj/new_meta.tsv  \
  --p-n-jobs-or-threads 4 \
  --output-dir core-metrics

qiime diversity alpha-phylogenetic \
  --i-table /home/users/maf1092/final-proj/merged-data/combined_filter_feature_table.qza \
  --i-phylogeny /home/users/maf1092/final-proj/new-phylo-tree/rooted-tree.qza \
  --p-metric faith_pd \
  --o-alpha-diversity /home/users/maf1092/final-proj/new-phylo-tree/core-metrics/faith_pd

qiime diversity alpha-rarefaction \
    --i-table /home/users/maf1092/final-proj/merged-data/combined_filter_feature_table.qza \
    --i-phylogeny /home/users/maf1092/final-proj/new-phylo-tree/rooted-tree.qza \
    --p-max-depth 150000 \
    --m-metadata-file /home/users/maf1092/final-proj/new_meta.tsv  \
    --p-min-depth 100 \
    --p-steps 15 \
    --o-visualization /home/users/maf1092/final-proj/new-phylo-tree/core-metrics/alpha-rarefaction

qiime diversity alpha-group-significance \
    --i-alpha-diversity /home/users/maf1092/final-proj/new-phylo-tree/core-metrics/faith_pd.qza \
    --m-metadata-file /home/users/maf1092/final-proj/new_meta.tsv  \
    --o-visualization /home/users/maf1092/final-proj/new-phylo-tree/core-metrics/alpha-group-significance

### Table in Excel

qiime tools export \
  --input-path /home/users/maf1092/final-proj/taxonomy/classify-sklearn-taxonomy.qza \
  --output-path /home/users/maf1092/final-proj/tables/

qiime tools export \
  --input-path /home/users/maf1092/final-proj/merged-data/combined_filter_feature_table.qza \
  --output-path /home/users/maf1092/final-proj/tables/

biom add-metadata \
  --input-fp /home/users/maf1092/final-proj/tables/feature-table.biom \
  -o /home/users/maf1092/final-proj/tables/table-with-taxonomy.biom \
  --observation-metadata-fp /home/users/maf1092/final-proj/tables/taxonomy.tsv \
  --observation-header "taxonomy" \
  --sc-separated taxonomy

biom convert \
-i /home/users/maf1092/final-proj/tables/table-with-taxonomy.biom \
-o /home/users/maf1092/final-proj/tables/otu-table.tsv \
--to-tsv --header-key taxonomy

## Results

These are the results of some of the data

<details>
<summary>Barplot</summary>
This shows the abundance of DNA per sample of each species that was identified.
<img src="https://github.com/meredithgenet1cs/gen_final_proj/blob/main/barplot-full.png">
</details>

<details>
<summary>Alpha Rarefaction</summary>
This shows the if the sequencing depth of the samples is deep enough for the analysis we used. This was made using the Shannon diversity index.
<img src="https://github.com/meredithgenet1cs/gen_final_proj/blob/main/alpha-rarefaction.png">
</details>

<details>
<summary>Alpha Significance</summary>
This shows the differences between the Great Bay and Wells data sets.
<img src="https://github.com/meredithgenet1cs/gen_final_proj/blob/main/alpha-significance.png">
</details>

### Emperor Plots

<details>
<summary>Jaccard</summary>
Shows the difference between data based on the presence or absence of data. Three different angles are shown emphasizing the different axes.
<img src="https://github.com/meredithgenet1cs/gen_final_proj/blob/main/jaccard-emperor.png">
<img src="https://github.com/meredithgenet1cs/gen_final_proj/blob/main/jaccard-emperor-2.png">
<img src="https://github.com/meredithgenet1cs/gen_final_proj/blob/main/jaccard-emperor-3.png">
</details>

### Jaccard

Shows the difference between data based on the presence or absence of data. Three different angles are shown emphasizing the different axes.
![image](jaccard-emperor.png)
![image](jaccard-emperor-2.png)
![image](jaccard-emperor-3.png)

### Bray-Curtis

Shows the difference between data based on the abundance of data. Three different angles are shown emphasizing the different axes.
![image](Bray-curtis1.png)
![image](Bray-curtis2.png)
![image](Bray-curtis3.png)

### Weighted Unifrac

Shows the diffference between data based on the abuncance and the phylogeny of data. Three different angles are shown emphasizing the different axes.
![image](weighted-unifrac-1.png)
![image](weighted-unifrac-2.png)
![image](weighted-unifrac-3.png)

## Bibliography

Bliek Tijs, Frans van der Kloet and Marc Galland (eds): "RNA-seq lesson."  Version 2020.04. https://github.com/ScienceParkStudyGroup/rnaseq-lessonhttps://scienceparkstudygroup.github.io/microbiome-lesson/06-beta-diversity/index.html

Bolyen E, Rideout JR, Dillon MR, Bokulich NA, Abnet CC, Al-Ghalith GA, Alexander H, Alm EJ, Arumugam M, Asnicar F, Bai Y, Bisanz JE, Bittinger K, Brejnrod A, Brislawn CJ, Brown CT, Callahan BJ, Caraballo-Rodríguez AM, Chase J, Cope EK, Da Silva R, Diener C, Dorrestein PC, Douglas GM, Durall DM, Duvallet C, Edwardson CF, Ernst M, Estaki M, Fouquier J, Gauglitz JM, Gibbons SM, Gibson DL, Gonzalez A, Gorlick K, Guo J, Hillmann B, Holmes S, Holste H, Huttenhower C, Huttley GA, Janssen S, Jarmusch AK, Jiang L, Kaehler BD, Kang KB, Keefe CR, Keim P, Kelley ST, Knights D, Koester I, Kosciolek T, Kreps J, Langille MGI, Lee J, Ley R, Liu YX, Loftfield E, Lozupone C, Maher M, Marotz C, Martin BD, McDonald D, McIver LJ, Melnik AV, Metcalf JL, Morgan SC, Morton JT, Naimey AT, Navas-Molina JA, Nothias LF, Orchanian SB, Pearson T, Peoples SL, Petras D, Preuss ML, Pruesse E, Rasmussen LB, Rivers A, Robeson MS, Rosenthal P, Segata N, Shaffer M, Shiffer A, Sinha R, Song SJ, Spear JR, Swafford AD, Thompson LR, Torres PJ, Trinh P, Tripathi A, Turnbaugh PJ, Ul-Hasan S, van der Hooft JJJ, Vargas F, Vázquez-Baeza Y, Vogtmann E, von Hippel M, Walters W, Wan Y, Wang M, Warren J, Weber KC, Williamson CHD, Willis AD, Xu ZZ, Zaneveld JR, Zhang Y, Zhu Q, Knight R, and Caporaso JG. 2019. Reproducible, interactive, scalable and extensible microbiome data science using QIIME 2. Nature Biotechnology 37: 852–857. https://doi.org/10.1038/s41587-019-0209-9

Faith, D. P. Phylogenetic diversity https://danielpfaith.wordpress.com/phylogenetic-diversity/ 

Froese, R. and D. Pauly. Editors. 2024.FishBase. World Wide Web electronic publication.
www.fishbase.org, ( 02/2024 )
https://www.fishbase.de/summary/Pholis-crassispina

Miller, J.
https://github.com/jthmiller/eDNA-metabarcoding-intro/blob/Gen711-811/metadata.md

Miya, M., Sato, Y., Fukunaga, T., Sado, T., Poulsen, J. Y., Sato, K., Minamoto, T., Yamamoto, S., Yamanaka, H., Araki, H., Kondoh, M., & Iwasaki, W. (2015, July 1). MiFish, a set of universal PCR primers for metabarcoding environmental DNA from fishes: Detection of more than 230 subtropical Marine Species | Royal Society Open Science. Royal Society Open Science. https://royalsocietypublishing.org/doi/10.1098/rsos.150088

Xiong, F., Shu, L., Gan, X., Zeng, H., He, S., & Peng, Z. (2022). Methodology for fish biodiversity monitoring with environmental DNA metabarcoding: The primers, databases and bioinformatic pipelines. Water Biology and Security, 1(1), 100007. https://doi.org/10.1016/j.watbs.2022.100007   
