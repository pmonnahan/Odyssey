# Odyssey 

---------
Version: 2.1.0

Odyssey is semi-autonomous workflow that aids in the preparation, phasing, and imputation of genomic data. Odyssey also provides support for custom reference panel merging, population structure (admixture) analysis, phenotype normalization, genetic data quality control, genome-wide-association studies (GWAS), and visualization of analysis results.

If you use Odyssey in any published work, please cite the following paper:

Eller, Ryan J., Sarath C. Janga, and Susan Walsh. "Odyssey: a semi-automated pipeline for phasing, imputation, and analysis of genome-wide genetic data." BMC bioinformatics 20.1 (2019): 364. https://doi.org/10.1186/s12859-019-2964-5

# General Overview
Odyssey relies primarily on the idea that the creation of imputed genomic data must go through 4 basic steps: pre-imputation QC, phasing, imputation, and post imputation cleanup and QC. Once Odyssey is setup, users can automate these 4 steps on a High Performance System (HPS) running some form of Linux or a desktop (if you are not concerned about time requirements as phasing/imputation is normally resource intensive). Odyssey contains 6 main folders. The '0_DataPrepModule' directory is where the user places his or her target data that needs to be remapped and/or 'fixed' to a reference genome (to assure proper genomic positions and allele 'flippige'). The '1_Target' directory is where the cleaned data from '0_DataPrepModule' is automatically put following the data being cleaned. The 'Reference' directory is where the reference data that will be used by Shapeit2/Eagle2 and Impute4/Minimac4. The '2_Phasing' directory is where phasing scripts, logs, and phased data will be housed while the '3_Impute' directory is where an imputed dosage VCF will be stored. The '4_GWAS' directory is where GWAS analyses are conducted and also contains the Phenotype sub-directory where phenotype normalization may be performed. A 'SubModule' directory contains the 'PopStratModule,' which performs admixture analysis on a given dataset, 'Remapping_Made_Easy_Submodule, which allows for semi-autonomous genome remapping on a given dataset, and the 'HGDP_Starter-SubModule', which formats a version of the HGDP dataset for Odyssey use. Lastly the '5_QuickResultsis' directory populates metadata collected over the range of analysis for a given imputation or GWAS project. Each imputation run and GWAS analysis are stored in discrete project folders within the main 4 directories (e.g. 1_Target, 2_Phasing, 3_Impute, and 4_GWAS).

Odyssey is setup based on a pre-made downloadable Singularity container, which contains all but 2 of Odyssey's dependencies. Impute4 must be downloaded separtely due to licensing restrictions, and GNU-Parallel (which is an optional dependency that speeds up runtime) must be configured separtely. All of Odyssey's dependencies are housed within the 'Configuration' directory.

Users should also make note of the Settings.conf file, which guides users in setting Odyssey's variables, which controls the workflow. This method of user control was used instead of the more traditional command line arguments in order to simplify the workflow while allowing the users to visualize all the options Odyssey has to offer.

Once Odyssey is setup and the workflow settings are configured in Settings.conf, the user can execute the various scripts in the main Odyssey folder which cleans, phases, imputes, and performs a GWAS on the data. Odyssey is optimized for human data, but it is theoretically possible to analyze data from other organisms using Odyssey as well.

# Odyssey Tutorial
As an added reference, an Odyssey Tutorial has been provided which contains a 100 sample HGDP dataset (provided in the 'Tutorial_Data' directory) as well as an 'Odyssey Tutorial [v#]' document, which walks you through the data prep, phasing, imputation, post imputation QC that is needed to create a dosage VCF, as well as some of the sub modules. The tutorial as well as configuration instructions can be found within the 'Odyssey_Literature' directory. I would highly recommend new users to utilize the detailed explanation that can be found in the tutorial in order to become more familiar with Odyssey. Running through the tutorial covers all the essential steps, offers some tips, and provides a look at more advanced settings that will hopefully make data prep, QC, phasing, imputation, post Imputation QC, GWAS and admixture analysis, and phenotype normalization very simple and easy. It will only take around 3-6 hour to complete the tutorial given adequate computational resources. One can even truncate the analysis to a few chromosomes to decrease the amount of time to completion even further.


---------
Quick Setup
==============

Note: It is best to use Odyssey on a system that has Singularity already installed. There are non-Singularity setup methods of setting up Odyssey, but doing so is not recommended. A detailed explanation of Odyssey setup can be found in 'Odyssey Installation Instructions' found within the Readme folder

Download Odyssey via Github:
------------------------
1) On the Odyssey Github page navigate to the Github Release tab

	The release should have 3 files for download: OdysseyContainer.sif.gz + Source code (zip) + Source code (tar.gz)

2) Download and extract the Source code to your working directory

		cd /Directory/You/Want/To/Work/From
		unzip Odyssey-[#.#.#]
		
4) Download and extract the OdysseyContainer.sif.gz to the Singularity directory: 
		
		cd ./Odyssey-[#.#.#]/Configuration/Singularity 
		unzip OdysseyContainer.sif.gz
		
5) Due to licensing restrictions, you will need to download and extract the Impute4 (if you choose to use that imputation software over Minimac4) executable to the Impute4 directory within Configuration

	a) Request access to Impute4 here: https://jmarchini.org/impute-4/
	
	b) After downloading Impute4 to the Impute4 directory you will need to alter the following line in ./Configuration/Setup/Programs-Singularity.conf:
	
		Impute_Exec4="${WorkingDir}Configuration/Impute4/impute4.1.1_r294.2";
	
	d) Replace impute4.1.1_r294.2 with the appropriate version you just downloaded

6) You’re done. Start using Odyssey

Extra Note: While the pre-made releases are an easy way to download everything you need to run Odyssey, I am constantly updating Odyssey's Github page so you may use Git to pull changes from Github between releases, if you want the latest and greatest improvements. 


Setup Reference Data: 
--------------------------
Reference data to be used with phasing/imputation is installed by the workflow by default, but can be changed in Settings.conf so that you can use your own reference data. To do so, simply populate the Reference Data Folder with your preferred reference data. By default reference data is downloaded from the IMPUTE2/Minimac3 site under their reference data for "1,000 Genomes haplotypes -- Phase 3 integrated variant set release in NCBI build 37 (hg19) coordinates" (Updated 3 Aug 2015)
		
If you choose to use a custom reference dataset, then several adjustments may need to be made to the naming of the reference files (.legend, .map, and .hap) and also to the Settings.conf file. These adjustments are explained in greater detail in the Settings.conf file. Users should pay particular attention to make sure that custom reference data is sync'ed to the target data (i.e. don't try and use a Target dataset mapped to GRCH 37 to a reference dataset mapped to GRCH 38).

Setup Target Data: 
-----------------------
As an optional first step to cleanup your genomic data (hereafter referred to as "Target" data) you may put your Plink formatted data (.bed/.bim/.fam) in ./0_DataPrepModule/PLACE_DATA_2B_FIXED_HERE/. Running the '0_DataPrepCleanup.sh' script from Odyssey's main directory will give you the option of fixing your target data to a GRCH 37 reference build and will remove positional duplicates if they exist. This step is not required, but highly recommended as positional duplicates will cause the pipeline to error out. Fixing the data to a reference genome also helps reduce the chances of encountering a workflow error caused by 'dirty' data. This optional step will deposit your data in the ./1_Target/PLACE_NEW_PROJECT_TARGET_DATA_HERE/.

Additional Note:
As data that will be imputed should match the genomic build of the reference dataset, it may be helpful to utilize the Remapping_Made_Easy_SubModule found within the 'SubModules' Directory. It utilizes NCBI's Remap Service to semi-automate the remapping of your data to a build of your choice (that is supported by NCBI of course). More information on running this sub-module can be found within the Odyssey Tutorial.

The first required step of Odyssey is placing data in the ./Target/PLACE_NEW_PROJECT_TARGET_DATA_HERE/ directory. Data placed here undergo basic quality control prior to phasing (e.g. filtering individuals and variants based on missingness, minor allele frequency, and Hardy-Weinbergy Equilibrium). The cutoffs are set by default to industry standards, however users may choose to adjust the thresholds in Settings.conf. In addition, at this step users may opt to interactively visualize their dataset exploring the quality control measures mentioned above, as well as evaluate sample relatedness via IBD. Dataset QC plots (e.g. IBD, HWE, and Missingness) are saved automatically to a results folder within ./Target/PLACE_NEW_PROJECT_TARGET_DATA_HERE/.


Fill out Settings.conf: 
-------------------------------------------
Settings.conf file is responsible for setting the variables that will be used to execute the scripts in the home directory. Essentially, all the main scripts on the home directory "phone home" back to the Settings file to lookup their variables. Because of this, unless additional customization is needed, you should never have to modify any of the main scripts in the home directory (however, each script within Odyssey Modules and Submodules are heavily commented to allow for easy navigation when attempting more advanced customization not supported by the Settings file). Most of these variables are relating to toggling steps of Odyssey (to allow for more user control and to help with troubleshooting), specifying the home directory (i.e. './Odyssey/'), etc. Step-by-step instructions on how to setup Settings.conf variables can be found in the Settings file itself and a more detailed explanation can be found in the Odyssey Tutorial.

A Note on Odyssey File Organization:
--------------------------------------
Odyssey has an organization scheme to keep all imputation results separate from each other so the user does not have to "reset" the Odyssey folder after each imputation run. Odyssey does this by organizing files into 'Imputatation Projects'. Each project will create a folder that is identified by a BaseName, or a name that is specified in Settings.conf at the beginning of the analysis to identify the imputation run. This will allow for the creation of identifiable folders within the Target, Phase, Impute, GWAS, and QuickResults folders. For example, if I have a dataset of Homo sapien target DNA that I want imputed, I will setup an Imputation Project named "Human_Impute1" (the name must not contain whitespaces). Odyssey will then create a target folder (within the Target directory) specific to my imputation run and move my data into it. Odyssey will then deposit phased and imputated scripts, outputs, and results within the Phase and Impute folders respectively. If I then want to impute a different set of data, I simply create a new Imputation project, which will separately house the target, phase, and impute data from my second imputation run.


 
---------
Running Odyssey
==============

Step 1: Pre-Imputation QC and Setup
-----------------------

1) Once the target data has been cleaned and is deposited in the 'PLACE_NEW_PROJECT_TARGET_DATA_HERE' folder within the Target folder directory, the first script, "1_ImputeProjectSetup-QC-Split.sh" can be run from the home directory. Simply use a command prompt to navigate to the home directory (e.g. $ cd /path/to/Odyssey-v[#.#.#]/) and execute the script (e.g. $ bash 1_ImputeProjectSetup-QC-Split.sh) which will setup an Imputation Project Folder, move your Target Data into this Project Folder, and will provide a small amount of pre-imputation QC which includes (by default):

	a) Filtering for individual missingness (removes individuals missing more than 5% of genomic content)
	
	b) Filtering for genetic variant missingness (removes variants missing in more than 5% of individuals)
	
	c) Filtering for minor allele frequencies (removes variants that contain a minor allele frequency of 2.5% or less)
	
	d) Filtering for Hardy-Weinberg Equilibrium (removes variants that have a HWE p-value of 1e-5 or below). This test is very lenient to allow for diverse target data.
	
2) SHAPEIT/EAGLE requires data to be split by chromosome so the last step is splitting the dataset into their respective chromsomes
	By default the script looks for chromosomes 1-26 (the default for human samples) into their respective chromosomes.

Step 2: Phasing
-----------------------

1) Odyssey organizes phased data into an Imputation Project Folder created within the Phase folder. The name of this folder is specified by the Imputation Project Name variable and will contain subdirectories that house the phasing scripts, outputs, and results.

2) No additional files outside of those created in Step 1 need to be created to run the Phasing step. Each step builds on the next and contains all the files necessary to run the next step. 

3) Phasing is carried out using SHAPEIT/EAGLE recommended settings (shown below) and a reference data map provided by IMPUTE2 (by default) or the user.		
	
	a) The SHAPEIT command has the following general form: shapeit --thread [# threads] --input-bed [PlinkTargetBedFile] --input-map [ReferenceMapFile] --output-max [OutputPhasedName] --output-log [OutputPhasedLogName]
	
	b) The EAGLE command has the following general form: 
	eagle --numThreads [# threads] --bfile [PlinkTargetBedFile] --geneticMapFile=[ReferenceMapFile] --outPrefix=[OutputPhasedName] --chrom [Chrom #]

4) Phasing customization can be set via altering settings found in Settings.conf

5) Phased output, logs, and scripts are deposited within the Imputation Project directory placed within the Phase directory

Step 3a: Imputation
-----------------------

1) Odyssey organizes imputed data into an Imputation Project Folder created within the Impute folder. The name of this folder is specified by the Imputation Project Name variable and will contain subdirectories that house the imputation scripts, outputs, and results. Note that either Impute or Minimac can be used with a pre-phased dataset run through either Shapeit or Eagle.

2) Imputation is carried out using IMPUTE/Minimac recommended settings using reference data (genetic, hap, and map files) provided by either IMPUTE/Minimac (by default) or the user. The General IMPUTE/Minimac commands are listed below. Note that the reason why Impute4 is used is because it has superior speed in comparison to Impute2. Minimac is another imputation solution and is setup by default for users who do not need to utilize Impute's abilities to more accurately impute admixed datasets.
		
	a) impute4 -g [PhasedHapsFile]  -s [PhasedSampleFile] -m [ReferenceGeneticMapFile] -h [ReferenceHapsFile] -l [ReferenceLegendFile] -int [StartChromosomeChunkSegment EndChromosomeChunkSegment] -maf_align -Ne [20000] -o [OutputName]
	
	b) minimac4 --cpus [# cpus] --allTypedSites --minRatio 0.00001 --refHaps [ReferenceHapsFile] --haps [PhasedHapsFileFromSHAPEIT] --prefix [OutputName]
	

3) Imputation customization can be set via altering settings found in Settings.conf.

4) Imputed output, logs, and scripts are deposited within the Imputation Project directory placed within the Raw_Imputation folder within the Impute directory


Step 3b: Post Imputation Cleaning and Concatenation
-----------------------

Post imputation cleaning is performed differently depending on whether Impute or Minimac was used.

For Impute Workflows:

1) Since imputted files are divided by chromosome and by segment, these files must be concatenated. Odyssey does this through 3b_ConcatConvert.sh which does a simple concat command with all the imputed chromsomal segments housed within the Raw Imputation folder and re-assigns their chromosome number (which isn't explicitly assigned during imputation)

2) SNPTEST then creates a SNP Report which calculates the INFO imputation QC metric. This will later be used to filter the VCF file (the INFO cutoff is set to 0.3 by default, but may be adjusted in Settings.conf)

3) Concatenated chromsomal .GEN files are converted to a dosage VCF file (.VCF) using Plink 2.0 and filtered by INFO score

4) The dosage VCF files are concatenated via BCFTools

5) The final output within the 'ConcatImputation' Folder contains the following:
		
	a)	.snpstat (SNPTEST snp report that contains several metrics on the imputed chromosome including the INFO score)
	
	b)	.snpstatOut (is a log file for SNPTEST which contains the run results from SNPTEST AND a count of the total number of variants imputed for the particular chromosome and how many are left after INFO filtering)
	
	c)	Most importantly is the 1Imputed_[BaseName].vcf.gz file which is the final imputation product. This dosage VCF file can be inputted into analysis programs such as Plink 2.0, SNPTEST, GenAble, etc. for further analysis
	

For Impute Workflows:

1) By default Minimac concatenates imputed chromosomal segments into imputed chromosomal.vcf.gz files which already contains the Minimac R2 (very similar to Impute's INFO scores) imputation QC metric.

2) These chromosomal.vcf.gz files are then filtered via Plink

3) Filtered vcf.gz imputation files are then concatenated via BCFTools

5) The final output within the 'ConcatImputation' Folder contains the following:
		
	a)	.log and .out files detailing the actions performed on individual dosage chromosomal vcf.gz files

	b)	Most importantly is the 1Imputed_[BaseName].vcf.gz file which is the final imputation product. This dosage VCF file can be inputted into analysis programs such as Plink 2.0, SNPTEST, GenAble, etc. for further analysis. Unlike the Impute vcf.gz product, the Minimac vcf.gz has the R2 INFO metrics baked into the vcf.gz file itself for all imputed and typed variants
	



Step 4: Setup GWAS Project and Run GWAS Analysis
-----------------------
The last step in Odyssey will be to setup a GWAS Analysis Project where a dosage VCF can either be manually imported into Plink or an Imputation Project can be specified, which will allow Odyssey to automatically lookup the dosage VCF file and corresponding sex sample file (the .fam file for the dataset which contains sex information) and perform an analysis. How Plink imports the data can be setup via Settings.conf.
The GWAS analysis is designed to perform and visualize a genotype:phenotype analysis (i.e. a Genome-Wide-Association Study). Specifically, the genotypic data and inputted phenotypic data is fit on a general linear model or a logistic model, and R is used to visualize the output.

1) Users will need to setup a GWAS Project for the GWAS analysis by completing the GWAS Project Variables Section of Settings.conf. More specific details on how to fill out the variables are included within the Settings file itself and the tutorial, but briefly:
	
	a) Specify the GWAS Project Name
	
	b) List the Imputation Project the user wishes to analyze or manually list the path of a dosage VCF
	
	c) Specify the name of the covariate/phenotype file that correspond to the imputation files (which is placed in ./4_GWAS/Phenotype). 
		
	Note: It is important that the user read the instructions in the Plink manual regarding the formatting of the phenotype and covariate files.
	
	d) List any additional Plink options to be run during the analysis withing the "Plink_Options" variable

2) Run the '5_AutomatePlink.sh' script from the Odyssey directory

3) A new GWAS Project directory should be created which upon analyis completion should contain the following:
	
	a) An analysis log file
	
	b) A QQPlot
	
	c) A Plotly interactive Manhattan Plot (with its Plotly dependency folder)
	
	d) A summary table that contains the top 10000 variants with the lowest unadjusted p-values (as well as multiple comparison corrections, effect sizes, etc.)
	
	e) A gzipped file that contains the raw results from Plink
	


---------
Additional Odyssey Tasks
==============

Population Stratification Analysis
-----------------------------

Odyssey is capable of performing an Eigensoft-like analysis to assess a datasets ancestry structure using reference datasets such as the 1000 Genomes or HGDP datasets. Follow the instructions found within 00_PopStrat_Helper.xlsx, which is an Excel file that already has 1000 Genomes and HGDP datasets pre-populated and organized. Briefly, a Population Stratification workflow follows the following steps:

1) 2 datasets, a reference dataset that contains known ancestry and a target dataset that contains ancestries that will be determined, are placed in the ./SubModules/PopStratModule/PLACE_Target-Ref_Datasets_HERE/ directory
	
2) Common genotypes are found between the 2 datasets in order to perform an inner-merge (i.e. the 2 datasets are merged so that only the common genotypes are retained)

3) The resulting dataset undergoes some basic quality control measures (i.e. missingness and HWE) and then pruned based on linkage disequilibrium

4) A PCA analysis is performed on the combined, QC'ed, and prunned dataset

5) Files that contain the PCA eigenvectors and eigenvalues are used in tandem with an ancestry file (which is a Plink formatted ID list of reference samples that contain the ancestry the user wishes to keep) that the user uploads

6) An R script is then used to construct an X-dimensional centroid based on the eigenvectors of the samples that are of the ancestry the user wishes to retain. Outliers that fall outside of the X-dimensional centroid are determined based on a specified standard deviation or inter quartile range cutoff.

6) The output of the analysis is an interactive 3D Plotly scatterplot that color coordinates individuals who are deemed outliers, reference samples, and samples that should be included in the analysis. A text document recommending which individuals should be removed based on ancestry is also provided.


GWAS Phenotype Normalization
-----------------------------

Non-normal phenotypes occasionally negatively affect GWAS analyses. In response Odyssey contains an Analyze/Transform Phenotypes script housed within the ./4_GWAS/Phenotype directory. Running this script will prompt for the path to a Plink formatted phenotype file to analyze and will then proceed to show the distribution of the phenotype by column (with each column, m>2, being a different phenotype). See https://www.cog-genomics.org/plink/1.9/input#pheno for more information on phenotype file formatting. This interactive script will then ask the user whether to perform a Yeo-Johnson, Rank-Order Inverse Normalization, or no normalization on the selected phenotype. If normalizing the script will visualize the before and after, ask whether the transformation is to be accepted, and if accepted will append the transformed phenotype onto the end of the phenotype file.

Custom Imputation Ref Panel Creation (BETA)
-----------------------------

As more sequence data is being collected it is now possible for users to create highly customizable imputation reference panels that better suits their needs. If a user has several reference datasets (e.g. custom collected data in addition to 1000 Genomes data) and wishes to merge them into a single reference dataset, then this can be accomplished using the MergeRefSets Submodule found within ./Reference/MergeRefSets/. In general, the workflow is as follows:

1) Put Reference dataset 1 in ./Reference/MergeRefSets/Reference1

2) Put Reference dataset 2 in ./Reference/MergeRefSets/Reference2

3) Add the proper genetic map files in ./Reference/MergeRefSets/GeneticMaps

4) Configure settings in Merge.conf

5) From ./Reference/MergeRefSets run
	
		$ bash ./MergeRef.sh

6) The two reference datasets will be merged using Impute2's merging function (explained here: https://mathgen.stats.ox.ac.uk/impute/merging_reference_panels.html) and deposited in ./Reference/MergeRefSets/MergedRefPanels

Note: Depending on the size of the reference panels, this operation may take an incredibly long time since the operation is essentially imputing the reference sets to each other, which is essentially a dual-imputation procedure.







