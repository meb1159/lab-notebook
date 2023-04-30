# lab-notebook
lab notes for gen 711

Friday 4/14/23

repositories on Github is kind of like goodgle docs, way to share data to other people and either allow their changes or deny them 
- putting all results and information for final projects into respositories 
    make repo
        - 
- can edit in web interface or edit in vscode and upload changes to github 
# in vscode makes a new section 
- code format for plots in in provided example 

Project layout:
Title
Authors 
Background
Methods
Findings  

- Making our own repositorys 
 - hit '+' 
 - hit 'new repository' 
 - give it a name, description and set limits 

 git clone [http link]
  - pulls whole repository into folder on vscode 

github: jthmiller/gen711-811-example

Group: Nicole, Kim, Me

create a pull request to send something to be published on shared page... nicole has to approve it 

## Friday 4/21/2023 
link to main info repository: https://github.com/jthmiller/eDNA-metabarcoding-intro/tree/Gen711-811
    link to our data: /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-1
    
#Steps to complete this lab:
1. Run Fastp to trim the 'poly-g' tails of the reads
2. Import the directory of reads with qiime and save the output in your directory
3. Use the 'cutadapt' program in qiime to trim off the primers

made directory-  mkdir trimmedfastqs
activated conda - conda activate genomics

touch testfile.txt
    # made a tester file with nothing in it to play with 

cp /tmp/gen711_project_data/fastp.sh ~/fastp.sh
cp /tmp/meb1159/711-final-project/fastp.sh ~/fastp.sh
    # copied the fastp.sh to our home directory 
chmod +x ~/fastp.sh
    # changing the permissions of the file we just copied 

cp /tmp/meb1159/711-final-project/fastp.sh ~/fastp.sh

./fastp.sh 150 /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-2 trimmed_fastqs
    # 150 is thepoly g length cut off, with the directory of the reads to trim and the directory to store the reads in 
./fastp.sh 150 /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-1 trimmed_fastqs
     # 150 is thepoly g length cut off, with the directory of the reads to trim and the directory to store the reads in 

conda env list  
conda activate qiime-2022.8

qiime tools import \--type "SampleData[PairedEndSequencesWithQuality]"  \ --input-format CasavaOneEightSingleLanePerSampleDirFmt \ --input-path /home/users/meb1159/trimmed_fastqs \ --output-path <path to an output directory>/<a name for the output files> \
    # import the directory of poly-G trimmed FASTQ files into a single 'qiime file'

qiime cutadapt trim-paired \
    --i-demultiplexed-sequences <path to the file from step 2> \
    --p-cores 4 \
    --p-front-f <the forward primer sequence> \
    --p-discard-untrimmed \
    --p-match-adapter-wildcards \
    --verbose \
    --o-trimmed-sequences <path to an output directory>/<name for the output files>.qza
    # remove the primers and adapters of each pair of sequences

qiime demux summarize \
--i-data <path to the file from step above> \
--o-visualization  <path to an output directory>/<a name for the output files>.qzv 
    # make a summary of the summary.qzv file
  
## Friday 4/28/23

#activate genomics environment
conda activate genomics
cp /tmp/gen711_project_data/fastp-single.sh .
chmod +x fastp-single.sh
# run fastp
./fastp-single.sh 120 /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-2 trimmed_fastqs_FP

./fastp-single.sh 120 /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-1 trimmed_fastqs_FP1

-check the file sizes of the polyg trimmed fastq files
-Are any empty? Those will need to be removed from the directory
ls -shS

# run the following steps twice. Once for each output directory

# activate qiime
conda activate qiime2-2022.8

# import fastqs into a single qiime file
qiime tools import --type "SampleData[SequencesWithQuality]" --input-format CasavaOneEightSingleLanePerSampleDirFmt --input-path /home/users/kgb1008/711-final-project/trimmed_fastqs_FP --output-path qiime_trimmed_fastqs_2

qiime tools import --type "SampleData[SequencesWithQuality]" --input-format CasavaOneEightSingleLanePerSampleDirFmt --input-path /home/users/kgb1008/711-final-project/trimmed_fastqs_FP1 --output-path qiime_trimmed_fastqs_1

# cutadapt
qiime cutadapt trim-single --i-demultiplexed-sequences qiime_trimmed_fastqs_2.qza --p-front TACGTATGGTGCA --p-discard-untrimmed --p-match-adapter-wildcards --verbose --o-trimmed-sequences qiime_cutadaot_fastqa_2

qiime demux summarize --i-data qiime_cutadaot_fastqa_2.qza --o-visualization qiime_demux_fastqa_2.qza

qiime cutadapt trim-single --i-demultiplexed-sequences qiime_trimmed_fastqs_1.qza --p-front TACGTATGGTGCA --p-discard-untrimmed --p-match-adapter-wildcards --verbose --o-trimmed-sequences qiime_cutadaot_fastqa_1

qiime demux summarize --i-data qiime_cutadaot_fastqa_1.qza --o-visualization qiime_demux_fastqa_1.qza

# denoising
qiime dada2 denoise-single --i-demultiplexed-seqs qiime_cutadaot_fastqa_2.qza --p-trunc-len 50 --p-trim-left 13 --p-n-threads 4 --o-denoising-stats denoising-stats_2.qza --o-table feature_table_2.qza --o-representative-sequences rep-seqs_2.qza
        Saved FeatureTable[Frequency] to: feature_table_2.qza
        Saved FeatureData[Sequence] to: rep-seqs_2.qza
        Saved SampleData[DADA2Stats] to: denoising-stats_2.qza

qiime metadata tabulate --m-input-file denoising-stats_2.qza --o-visualization denoising-stats_2.qzv
        Saved Visualization to: denoising-stats_2.qzv

qiime feature-table tabulate-seqs --i-data rep-seqs.qza --o-visualization rep-seqs_2.qzv
        Saved Visualization to: rep-seqs_2.qzv

qiime dada2 denoise-single --i-demultiplexed-seqs qiime_cutadaot_fastqa_1.qza --p-trunc-len 50 --p-trim-left 13 --p-n-threads 4 --o-denoising-stats denoising-stats.qza --o-table feature_table.qza --o-representative-sequences rep-seqs.qza
        Saved FeatureTable[Frequency] to: feature_table.qza
        Saved FeatureData[Sequence] to: rep-seqs.qza
        Saved SampleData[DADA2Stats] to: denoising-stats.qza    

qiime metadata tabulate --m-input-file denoising-stats.qza --o-visualization denoising-stats.qzv
        Saved Visualization to: denoising-stats.qzv

qiime feature-table tabulate-seqs --i-data rep-seqs.qza --o-visualization rep-seqs.qzv
        Saved Visualization to: rep-seqs.qz
        
## .gza files saved at:
/home/users/meb1159/711-final-project/trimmed_fastqs


