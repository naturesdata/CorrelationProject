# Evaluating Feature Correlation in the Alzheimer's Disease Neuroimaging Initiative
## Table of Contents
1. [Command Line Tools Required For This Project](#table1)
2. [Downloadable ADNI Files Needed](#table2)
3. [Required Computational Resources For Supercomputer](#table3)
4. [Cloning The Project Repository](#header1)
5. [Creating The Raw ADNIMERGE Data Domain](#header2)
6. [Creating The Python Virtual Environment](#header3)
7. [Creating The Gene Expression Data Domain](#header5)
8. [Creating The MRI Data Domain](#header6)
9. [Completing The ADNIMERGE Data Domain](#header4)
10. [Creating The Final Data Set](#header7)
11. [Preparing For The Correlation Analysis](#header8)
12. [Performing The Correlation Analysis](#header9)
13. [Feature Frequency Of Significant Comparisons](#header10)
14. [Feature Frequency For The Confounding Variable Sub Sets](#header11)
15. [Counting Comparisons](#header12)
## Command Line Tools Required For This Project <a name="table1"></a>
| Tool   | Version |
|--------|---------|
| Python | 3.7     |
| R      | 3.6     |
| gcc    | 8       |
| Perl   | 5.28    |
## Downloadable ADNI Files Needed <a name="table2"></a>
|               Data               | Disk Space (Gigabytes) | Download Time (seconds) |
|:--------------------------------:|:----------------------:|:-----------------------:|
| ADNI_Gene_Expression_Profile.zip |          0.075         |           3.0           |
|          Enrollment.zip          |         0.0016         |          0.064          |
|   Image Collections Directories  |          1,200         |         48,000.0        |
|          Study_Info.zip          |          0.272         |          10.88          |
## Required Computational Resources For Supercomputer <a name="table3"></a>
| Resource                                      | Amount         |
|-----------------------------------------------|----------------|
| Minimum Hard Drive Space                      | 3 Terabytes    |
| Minimum RAM For Highest Memory Node           | 1600 Gigabytes |
| Minimum Number Of Cores per 128 Gigabyte Node | 4              |
| Minimum Number Of 128 Gigabyte Nodes          | 50             |
| Minimum Number of 64 Gigabyte Nodes           | 1              |
| Minimum Number of 32 Gigabyte Nodes           | 1              |
| Minimum Number of 16 Gigabyte Nodes           | 10             |
| Minimum  GPU Memory per 16 Gigabyte Node      | 16 Gigabytes   |
## Cloning The Project Repository <a name="header1"></a>
The repository can be cloned using the following commands:
```
git clone --recurse-submodules https://github.com/naturesdata/CorrelationProject.git
cd CorrelationProject/
```
Within the repository are scripts for constructing the gene expression data domain, constructing the MRI data domain, constructing the ADNIMERGE data domain, and combining the three data domains into a final data set. Then there are scripts for performing the correlation analysis after the data set is complete.
## Creating The Raw ADNIMERGE Data Domain <a name="header2"></a>
Due to its complexity, the processing for the ADNIMERGE data domain occurs within its own submodule, accessible by executing the following from the root of the repository:
```
cd DataClean/MergeTables/
```
The compressed file `Enrollment.zip` can be downloaded at http://adni.loni.usc.edu and contains the ADNI registry table in the form of a CSV file called `REGISTRY.csv`. This file must be moved into the `Dates` directory of the `MergeTables` submodule. Information extracted from the registry table is used to create the dates table. This table includes the dates when measurements were taken in order to select data values which were measured at the latest date. The dates table is stored at `./Dates/dates.rds` and can be created by executing the following commands in the submodule:
```
cd Dates/
Rscript dates.r 2> /dev/null
cd ../
```
The compressed file `Study_Info.zip` can be downloaded at http://adni.loni.usc.edu and contains the `ADNIMERGE_0.0.1.tar.gz` file. After placing this `.tar.gz` file into the root of the `MergeTables` submodule, the `ADNIMERGE` package can be installed. The `ADNIMERGE` package contains several of the tables, in the form of R data frames, which are accessible in the ADNI website, all in one R environment. It can be installed along with all the other necessary R packages by executing the following:
```
Rscript installPackages.r
```
(Estimated Time: 10 minutes). More information on the `ADNIMERGE` package can be found at https://groups.google.com/forum/#!forum/adni-data. Once the necessary R packages are installed, the tables in `ADNIMERGE` can be stored in a list by executing the following:
```
cd AddTables
Rscript add-tables.r
cd ..
```
(Estimated Time: 2 minutes). This script will store the list of tables in `added-tables.rds`, rather than the tables existing separately in an R environment as the default. Additionally, the tables are processed and selected, meaning not all tables make it to the list. So the next steps in the process only use the tables that qualify and those tables are properly processed before they’re used in the next step. Processing involves capitalizing all headers for consistency. Columns that have only one unique value are removed, as they are not informative. If not enough columns remain, then the table is not included. Additionally, tables without a column with a RID header, which acts as a patient ID column, are not included because we can’t merge these tables with the others without a primary key. The names of the tables that couldn’t be added are stored in `not-added.rds`. The measurements that are available for each patient are collected by executing the following:
```
cd KnownValues
Rscript known-values.r
cd ..
```
*See Table 1 in the supplemental materials under "Getting The Known Values From ADNIMERGE" for resource usage information for this command.* The above command creates a mapping from RID, which is a unique identifier for a patient, to a list of known measurements for that patient. This information is obtained from the list of tables created previously. The dates table, also created previously, is used to obtain the value that was measured on the latest date in attempt to have as many of the most recent measurements as possible. An arbitrary value is selected in the case a date is not available for a given measurement of a given individual. The mapping is saved in `known-vals.rds`.
<br>
Once the lists of available measurements are made, one list for each RID, they are ready to potentially become columns in the combined ADNIMERGE data domain, so the type of measurement will be referred to as a column name or header for the remainder of the process. The headers for the soon-to-be ADNIMERGE data domain can be sorted by executing the following.
```
cd SortColNames
Rscript sort-col-names.r
cd ..
```
(Estimated Time: 14 minutes). This script uses the `known-vals.rds` file to create a list of sorted column names, saved in `sorted-col-names.rds`. They are sorted by the number of RIDs that have a known value for the corresponding column that will be created in the ADNIMERGE data domain. The list is sorted in descending order. This list enables the creation of a table where the leftmost column has the most known values while the rightmost column has the least known values.
<br>
Now using both `known-vals.rds` and `sorted-col-names.rds`, the raw ADNIMERGE data domain can be made. It’s called raw because further processing will occur later using the python language rather than R.
```
cd RawDataSet
Rscript raw-data-set.r
cd ..
```
(Estimated Time: 12 minutes). This script iterates though each column name, each iteration of which iterates through each RID. If an RID has a known value for a given column, it is added. Otherwise, a value of `NA` (not available) is added to the column. The resulting data frame is saved in `raw-data-set.rds`.
<br>
Next, the type of each column must be known to determine which statistical test is appropriate for each pair of columns. The data frame of column types has the same headers as the raw data domain except for RID and only has a single row that contains an indicator of whether the corresponding column is numeric or nominal. This data frame is stored in `col-types.rds` and is created by executing the following:
```
cd ColTypes
Rscript col-types.r
cd ..
```
(Estimated Time: 4 minutes). Column types are determined by the contents of the column itself. If the values are numerical and there’s enough unique values (more than 10), the column is deemed numeric. If the values are not of a numeric type, it’s deemed nominal. But headers are not included in the column types data frame if the column has only one unique value. Headers are also not included if the corresponding column is not numeric but has too many unique values (at least 20), and therefore couldn’t reasonably represent a categorical variable. Such columns typically didn’t have useful information for statistical analysis, but were often notes taken by the data recorders which were full English sentences, clearly not variables, but still made their way into the `ADNIMERGE` tables. In this way, the column types data frame is used to filter the raw data domain, excluding those columns that are unfit.
<br>
The final step is converting the column types data frame and the raw data domain data frame into CSV files to be combined with the gene expression domain and MRI domain after further processing. These CSV files, stored as `col-types.csv` and `raw-data-set.csv`, are created by executing the following:
```
cd ToCSV
Rscript to-csv.r
cd ..
```
(Estimated Time: several seconds). This script uses the headers of the column types data frame to select the acceptable columns from the raw data domain, then both data frames are saved as CSV files. This step concludes the process for putting together the raw ADNIMERGE data domain. Return to the root of the repository by executing:
```
cd ../..
```
## Creating The Python Virtual Environment <a name="header3"></a>
To set up the virtual environment for the python scripts, execute the following:
```
virtualenv -p `which python3` env
source env/bin/activate
pip3 install -r requirements.txt
deactivate
```
(Estimated Time: several minutes).
## Creating The Gene Expression Data Domain <a name="header5"></a>
The file `ADNI_Gene_Expression_Profile.zip` can be found at https://ida.loni.usc.edu/ under Download -> Genetic Data. It must be moved to `./data/gene_expression`. The contents can be extracted by executing the following:
```
cd data/gene_expression
unzip ADNI_Gene_Expression_Profile.zip
cd ../..
```
The gene expression data domain can then be created by executing the following:
```
cd DataClean/
bash jobs/adni-expression.sh
```
(Estimated Time: 3 minutes)
## Creating The MRI Data Domain <a name="header6"></a>
To begin creating the MRI data domain, the directories containing the image files can be found at https://ida.loni.usc.edu/ under Download -> Image Collections. There are 2,432 directories, the name of each corresponds to a patient ID. All of these directories must be moved to `./data/mri/raw-adni/` for later steps. Most of these images will be filtered by taking the intersect of patient IDs with the patient IDs of the other domains (i.e. ADNIMERGE and gene expression). This intersect can be accomplished by first obtaining the intersecting patient IDs just between the ADNIMERGE data domain and gene expression data domain. 744 patient IDs intersect between these two domains, meaning they have 744 individuals in common. To create a file containing these intersecting patient IDs, execute the following:
```
bash jobs/ptids.sh adni
```
(Estimated Time: 1 minute). Once the raw MRI data is in the correct data directory and the intersecting patient IDs are saved, the MRIs that have patient IDs that intersect with the prior saved patient IDs can be extracted and organized by executing the following:
```
bash jobs/med-adni-mri.sh
```
*See Table 1 in the supplemental materials under "Collecting The Medical Images Of Intersecting Patient IDs" for resource usage information for this command.* A total of 743 patient IDs (only 1 less than before) intersect between all three data domains, so there will be 743 individuals in the final data set. Since we are filtering the non-intersecting individuals, there will therefore also be 743 individuals in the MRI data domain. We don’t bother filtering the patients in the other data domains firstly because we couldn’t know what the MRI patient IDs are beforehand and secondly creating the other domains is not nearly as computationally intensive as creating the MRI data. Besides, those non-intersecting individuals in the ADNIMERGE and gene expression domains will necessarily be filtered when the three data domains are merged into one.
<br>
The MRIs are of the DICOM file format, a common file format for medical images, which contains metadata in addition to the images themselves. These DICOM medical images are handled using the `Pydicom` python package. More information about DICOM files and the Pydicom package can be found at https://pydicom.github.io. The medical images, however, are not in a format that’s ideal for deep learning. They can be converted to PNG format by executing the following:
```
bash jobs/png-mri.sh adni
```
*See Table 1 in the supplemental materials under "Converting The DICOM Images To PNG Format" for resource usage information for this command.* The medical images are converted to PNG format using a python package called `med2image`. More information about this package can be found at https://pypi.org/project/med2image/. After the images are organized and converted into a usable format, they then need to be further processed by executing the following:
```
bash jobs/txt-mri.sh adni
```
*See Table 1 in the supplemental materials under "Processing The PNG Images And Converting Them To TXT Files" for resource usage information for this command.* This processing includes normalizing, resizing, and splicing. The normalizing step is a simple min-max normalization of the pixel values. The resizing is performed using the OpenCV python package. The images are processed as multi-dimensional arrays provided by the `NumPy` python package. The splicing is necessary because there is not a single image per individual, but rather, a sequence of images. Sagital MRIs were captured in a sequence of images ranging from one side of the skull to the other. The first images in each sequence are of layers on one side, later images are of layers in the middle, and the last images are of the other side. These layers are known as slices. Different individuals in the data have different numbers of slices, so they have different numbers of images in their sequence. The smallest sequence length is 124 slices, and those individuals with larger sequence lengths have their sequences spliced to 124 for consistency. After being processed, the images are saved as txt files and are finally ready for the deep convolutional neural network.
<br>
Since the individual samples have sequences of images rather than single images, a separate convolutional autoencoder is trained for each slice index. Since there are 124 slices per individual, there are 124 slice indices, ranging from 0 to 123, where 0 refers to the first slice in the sequence and 123 refers to the last (zero-based numbering). As stated above, there is 743 individuals in the MRI data domain, each of which has their sequence from slices 0 to 123, so there are 743 index 0 images, 743 index 1 images, all the way to 743 index 123 images for a total of 743 * 124 = 92,132 images. Since there are 743 images for each slice index, and there is a separate trained autoencoder per slice index, each autoencoder is trained on 743 images.
<br>
To train the convolutional auto-encoders, execute the following:
```
bash jobs/conv-autoencoder.sh 0 adni
```
*See Table 1 in the supplemental materials under "Training The Autoencoders" for resource usage information for this command.* The 0 argument in the above command refers to the first slice index, meaning the model corresponding to the 0 slice index is trained. This same command must additionally be ran using a slice index of 1, 2, and all the way to 123.
<br>
Once the auto-encoders are trained, they can be used to compress the images into one-dimensional latent vectors which are concatenated together to become the rows in the table for the MRI domain. Execute the following to do this step:
```
bash jobs/mri-table.sh adni
```
*See Table 1 in the supplemental materials under "Creating The MRI Domain" for resource usage information for this command.*
## Completing The ADNIMERGE Data Domain <a name="header4"></a>
Before the ADNIMERGE data domain can be completed, it must have its instances filtered by intersecting patient IDs between its raw form, the gene expression data domain, and the MRI data domain. This filtering is possible now that the MRI domain is complete and is necessary to impute missing values correctly. Run the patient IDs script again by executing the following:
```
bash jobs/ptids.sh adni
```
(Estimated Time: 1 minute). The ADNIMERGE data domain's final processing, including imputing missing values, can be completed by executing the following:
```
cd DataClean/
bash jobs/phenotypes.sh adni
cd ..
```
*See Table 1 in the supplemental materials under "Completing The ADNIMERGE Data Domain" for resource usage information for this command.* The command above additionally creates mappings from patient ID to confounding variable values to be used later.
## Creating The Final Data Set <a name="header7"></a>
After the MRI data domain is complete, it can be merged with the ADNIMERGE domain and gene expression domain by executing the following:
```
bash jobs/combine.sh adni combined processed-data/datasets/adni/mri.csv
```
*See Table 1 in the supplemental materials under "Combining The Three Domains Into The Final Data Set" for resource usage information for this command.* The final data set is now complete and ready for correlation analysis. Enter the data directory of the `CorrelationAnalysis` sub module by executing the following:
```
cd ../
cd CorrelationAnalysis/data
```
Give the `CorrelationAnalysis` repo access to the cleaned data by using symbolic links to the `DataClean` sub module by executing the following:
```
ln -s ../../DataClean/processed-data/datasets/adni/combined-col-types.csv col-types.csv
ln -s ../../DataClean/processed-data/datasets/adni/phenotypes-col-types.csv adnimerge-col-types.csv
ln -s ../../DataClean/processed-data/datasets/adni/combined.csv data.csv
cd ../
```
## Preparing For The Correlation Analysis <a name="header8"></a>
To get the column types in a form usable by the correlation analysis script, execute the column types script:
```
bash jobs/col-types.sh
```
(Estimated Time: 1 minute). The column types need to be known for the analysis because the type of each feature determines which statistical test to perform. We don't want to include the insignificant tests in the analysis. To determine significance, we first must obtain a Bonferroni corrected alpha. This alpha can be found by executing the following:
```
bash jobs/bonferroni.sh
```
(Estimated Time: 1 minute). This script corrects the original alpha value of 0.05 simply by dividing the original alpha by the total number of tests performed which is 355,229,668,828. The correlation script, due to its big O squared complexity and the large input size, needs to be split up among a total of 6,418 jobs. The inputs to those jobs are computed in a complicated manner and therefore are computed in their own script. Execute that script with the following command:
```
bash jobs/col-comparison-inputs.sh
```
## Performing The Correlation Analysis <a name="header9"></a>
This script will create a CSV table containing the correct inputs for the correlation analysis ranging from job number 0 to job number 6417. The correlation analysis script uses that CSV file and a given job number to get its inputs for a given job. Below is the command for running the correlation analysis for job 0.
```
bash jobs/col-comparison-dict.sh data/data.csv data/col-comp-inputs.csv 0 4 comp-dicts
```
*See Table 1 in the supplemental materials under "Correlation Analysis On The Entire Data Set" for resource usage information for this command.* The 0 argument in the above command refers to the job number. This same command must be ran for all job numbers all the way to 6417. All these jobs together perform a total of 355,229,668,828 statistical tests, each of which produces a p-value for a feature pair that’s compared. But only the comparisons with a p-value below the bonferroni threshold are stored. Every feature in the data set is compared to every other feature after all 6,418 jobs are complete. The output of each job is a mapping from the comparison pair (two headers) to the p-value obtained performing the correlation test between that pair of features.
<br>
The mappings that are output from these jobs will not all be the same size since different parts of the total set of comparisons had more significant comparisons than others. So some of the output files of the above 6,418 jobs will have different sizes than others, which will make determining resource allocation of future analysis difficult. This issue can be resolved by rearranging the file contents such that they all have approximately equal size by executing the following:
```
bash jobs/even-comp-dicts.sh data/comp-dicts 13200000
```
*See Table 1 in the supplemental materials under "Equalize The Size Of The Files Containing The Comparisons" for resource usage information for this command.*
## Feature Frequency Of Significant Comparisons <a name="header10"></a>
The next part of the analysis is to obtain the mapping of features to the frequency which they appear in significant comparisons separated by domain. Before such a mapping can be created for features which appeared in *maximally* significant comparisons, we must filter the comparisons, selecting only the maximally significant ones:
```
bash jobs/alpha-filter.sh data/comp-dicts 5e-324 0 5 data/maximum-filtered
```
*See Table 1 in the supplemental materials under "Filtering Comparisons By Maximum Alpha" for resource usage information for this command.* The above command needs to be ran from index 0 to 852. The feature-to-frequency mappings for both Bonferroni significance and maximum significance can be made into CSV format by executing the following:
```
bash jobs/sig-freqs.sh data/comp-dicts data/bonferroni-sig-freqs.p
```
*See Table 1 in the supplemental materials under "Getting The Significant Comparison Frequencies For Features By Bonferroni Alpha" for resource usage information for this command.*
```
bash jobs/sig-freqs-table.sh data/bonferroni-sig-freqs.p data/bonferroni-sig-freqs.csv
```
*See Table 1 in the supplemental materials under "Creating The Significant Comparison Frequencies Table For Bonferroni Alpha" for resource usage information for this command.*
```
bash jobs/sig-freqs.sh data/maximum-filtered/ data/maximum-sig-freqs.p
```
(Estimated Time: 4 minutes).
```
bash jobs/sig-freqs-table.sh data/maximum-sig-freqs.p data/maximum-sig-freqs.csv
```
(Estimated Time: 9 minutes). These results can be summarized by executing the following:
```
bash jobs/sig-freqs-summary.sh maximum 100 data/maximum-sig-freqs.csv
```
(Estimated Time: 4 seconds).
```
bash jobs/sig-freqs-summary.sh bonferroni 100 data/bonferroni-sig-freqs.csv
```
(Estimated Time: 12 seconds).
## Feature Frequency For The Confounding Variable Sub Sets <a name="header11"></a>
Next link the maps of patient ID to feature value to the correlation analysis data directory:
```
cd data/
ln -s ../../DataClean/processed-data/feat-maps/ ./feat-maps
cd ../
```
This directory includes a mapping from patient ID to sex and one from patient ID to clinical dementia rating. These mappings will be used to create the sub sets of the data set which only contain the patients which have a specified value for these two potentially confounding variables. There will be a sub set which only contains male patients and one which only contains female patients. Create these two subsets by executing the following:
```
bash jobs/create-subset.sh ptgender
```
(Estimated Time: two minutes). Clinical dementia rating is also a potential confounding variable. The following command will split the data set into a subset for healthy controls, a subset for patients with mild cognitive impairment, and a subset for AD patients:
```
bash jobs/create-subset.sh cdglobal
```
(Estimated Time: two minutes). 
Now that the sub sets have been created, the same analysis as before can be performed on the sub sets only for those comparisons which passed the maximumally significant alpha. First, we need to make those files all the same size by running the following:
```
bash jobs/even-comp-dicts.sh data/maximum-filtered 30000
```
(Estimated Time: 40 minutes). Now the sub set analysis can be performed.
```
bash jobs/col-comparison-subset.sh 0 0.0 data/maximum-filtered
```
*See Table 1 in the supplemental materials under "Re-Run Only The Most Significant Comparisons On The Subsets" for resource usage information for this command.* In the data set, clinical dementia rating has numbers representing the categorical values with 0.0, 0.5, and 1.0 representing controls, mild cognitive impairment, and AD respectively. The 0.0 argument in the above command represents the healthy controls sub set that was created earlier. The above command must be executed using an argument of 0.5 and 1.0 as well. Additionally, the above command must be executed for each sex (i.e. male and female). Finally, each of those five commands must be ran for every index. The 0 argument in the above command represents the 0 index and the commands must be ran up to index 2955 for a total of 5 * 2,956 = 14,780  jobs.
<br>
After the analysis has been performed on each of the sub sets, the mappings of features to the frequency that they appear in significant comparisons (same mapping as above but for the sub set comparisons) can be made by executing the following:
```
bash jobs/sig-freqs.sh data/male-comp-dicts/ data/male-sig-freqs.p
bash jobs/sig-freqs.sh data/female-comp-dicts/ data/female-sig-freqs.p
bash jobs/sig-freqs.sh data/0.0-comp-dicts/ data/0.0-sig-freqs.p
bash jobs/sig-freqs.sh data/0.5-comp-dicts/ data/0.5-sig-freqs.p
bash jobs/sig-freqs.sh data/1.0-comp-dicts/ data/1.0-sig-freqs.p
```
Each of the above commands might take up to 4 minutes. Next the respective tables can be made:
```
bash jobs/sig-freqs-table.sh data/male-sig-freqs.p data/male-sig-freqs.csv
bash jobs/sig-freqs-table.sh data/female-sig-freqs.p data/female-sig-freqs.csv
bash jobs/sig-freqs-table.sh data/0.0-sig-freqs.p data/0.0-sig-freqs.csv
bash jobs/sig-freqs-table.sh data/0.5-sig-freqs.p data/0.5-sig-freqs.csv
bash jobs/sig-freqs-table.sh data/1.0-sig-freqs.p data/1.0-sig-freqs.csv
```
Each of the above commands might take up to 10 minutes. Finally, the results can be summarized:
```
bash jobs/sig-freqs-summary.sh male 100 data/male-sig-freqs.csv
bash jobs/sig-freqs-summary.sh female 100 data/female-sig-freqs.csv
bash jobs/sig-freqs-summary.sh 0.0 100 data/0.0-sig-freqs.csv
bash jobs/sig-freqs-summary.sh 0.5 100 data/0.5-sig-freqs.csv
bash jobs/sig-freqs-summary.sh 1.0 100 data/1.0-sig-freqs.csv
```
Each of the above commands might take up to 5 seconds.
## Counting Comparisons <a name="header12"></a>
The test results can now be counted by category. The category that a test result is counted in depends on its significance level and the comparison type. There are two kinds of comparison types. One comparison type category is based on the data types of the two features being compared. The categories of this comparison type are numeric being compared to numeric, nominal to nominal, and numeric to nominal. The different significance levels are no significance (not even below 0.05), below 0.05, below the Bonferroni corrected alpha of 1.4075400899075716e-13, below the super alpha of 1e-100, and maximum significance (so significant that the p-value is lower than the minimum floating point value that can be displayed in the python programming language and as a result is reported as 0.0). Since there are hundreds of billions of test results, they need to be counted by several jobs. Below is the command for job 0:
```
bash jobs/inter-counts-table.sh comp-dicts 1e-100 0 5 data-type
```
*See Table 1 in the supplemental materials under "Counting All The Comparisons By Data Type" for resource usage information for this command.* The 0 argument refers to the job index similar to training the MRI autoencoders and performing all the statistical tests. This time there are a total of 1,294 jobs that need to be ran so the above command must be ran from job index 0 to job index 1293. The same thing must additionally be done for the domain counts table as compared to the data-type counts table. Those steps can be done by executing the above command for each job index but this time replacing data-type with domain as seen below:
```
bash jobs/inter-counts-table.sh comp-dicts 1e-100 0 5 domain
```
*See Table 1 in the supplemental materials under "Counting All The Comparisons By Domain" for resource usage information for this command.* Rather than comparing counts by data type (numeric or nominal), this command will create intermediate counts tables by domain (ADNIMERGE, Expression, or MRI). The counts in all the intermediate tables can be added together into one by executing the following:
```
bash jobs/counts-table.sh data-type
bash jobs/counts-table.sh domain
```
The above commands might take about four minutes to complete.
<br>
After the analysis has been performed on each of the sub sets, the intermediate counts tables need to be created for each of them by executing the following:
```
bash jobs/inter-counts-table.sh 0.0-comp-dicts 1e-100 0 1 data-type 0.0
```
*See Table 1 in the supplemental materials under "Count The Comparisons Of The Subset Analysis" for resource usage information for this command.* New "comp-dicts" directories will have been created in the `data` directory, including `0.0-comp-dicts`, `0.5-comp-dicts`, `1.0-comp-dicts`, `male-comp-dicts`, and `female-comp-dicts`. The above command uses `0.0-comp-dicts` as an argument but that command must be ran for each of the new "comp-dicts" directories. The 0 (fourth argument) is again an index and the above command must be ran from indices 0 to 1293 for each of the sub sets. The last argument in the command specifies the sub set where 0.0 is in the above command but the sub sets 0.5, 1.0, male, and female must also be ran. And again the data-type argument is used in the above command. All of the jobs mentioned before must additionally be ran using the domain argument for a total of 5 * 1294 * 2 = 12,940 jobs. Once all the intermediate counts tables are complete for all of the sub sets and for both the data-type and domain tables, they each can be added up into a final counts table by executing the following:
```
bash jobs/counts-table.sh data-type 0.0
bash jobs/counts-table.sh domain 0.0
```
The 0.0 in the above commands again represents the 0.0 subset and the same commands must be ran for all the other sub sets as well. This step concludes the counting and should result in ten additional counts tables.
