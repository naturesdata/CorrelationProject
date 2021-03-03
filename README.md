# Evaluating Feature Correlation in the Alzheimer's Disease Neuroimaging Initiative
## Table of Contents
1. [Command Line Tools Required For This Project](#table1)
2. [Downloadable ADNI Files Needed](#table2)
3. [Cloning The Project Repository](#header1)
4. [Creating The Raw ADNIMERGE Data Domain](#header2)
5. [Creating The Python Virtual Environment](#header3)
6. [Completing The ADNIMERGE Data Domain](#header4)
7. [Creating The Gene Expression Data Domain](#header5)
8. [Creating The MRI Data Domain](#header6)
9. [Creating The Final Data Set](#header7)
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
The compressed file `Enrollment.zip` was downloaded at http://adni.loni.usc.edu and contains the ADNI registry table in the form of a CSV file called `REGISTRY.csv`. This file must be moved into the `Dates` directory of the `MergeTables` submodule. Information extracted from the registry table is used to create the dates table. This table includes the dates when measurements were taken in order to select data values which were measured at the latest date. The dates table is stored at `./Dates/dates.rds` and can be created by executing the following commands in the submodule:
```
cd Dates/
Rscript dates.r 2> /dev/null
cd ../
```
#### Run Time: 1.72 seconds
The compressed file `Study_Info.zip` was downloaded at http://adni.loni.usc.edu and contains the `ADNIMERGE_0.0.1.tar.gz` file. After placing this `.tar.gz` file into the root of the `MergeTables` submodule, the `ADNIMERGE` package can be installed. The `ADNIMERGE` package contains several of the tables, in the form of R data frames, which are accessible in the ADNI website, all in one R environment. It can be installed along with all the other necessary R packages by executing the following:
```
Rscript installPackages.r
```
#### Run Time: 9 minutes 47 seconds
More information on the `ADNIMERGE` package can be found at https://groups.google.com/forum/#!forum/adni-data. Once the necessary R packages are installed, the tables in `ADNIMERGE` can be stored in a list by executing the following:
```
cd AddTables
Rscript add-tables.r
cd ..
```
#### Run Time: 1 minute 8.413 seconds
This will store the list of tables in `added-tables.rds`, rather than the tables existing separately in an R environment as the default. Additionally, the tables are processed and selected, meaning not all tables make it to the list. This means that the next steps in the process only use the tables that qualify and those tables are properly processed before they’re used in the next step. Processing involves capitalizing all headers for consistency. Columns that have only one unique value are removed, as they are not informative. If not enough columns remain, then the table is not included. Additionally, tables without a column with a RID header, which acts as a patient ID column, are not included because we can’t merge these tables with the others without a primary key. The names of the tables that couldn’t be added are stored in `not-added.rds`. The measurements that are available for each patient are collected by executing the following:
```
cd KnownValues
Rscript known-values.r
cd ..
```
#### Run Time: 12 hours 19 minutes 50 seconds
The above command creates a mapping from RID, which is a unique identifier for a patient, to a list of known measurements for that patient. This information is obtained from the list of tables created previously. The dates table, also created previously, is used to obtain the value that was measured on the latest date in attempt to have as many of the most recent measurements as possible. An arbitrary value is selected in the case a date is not available for a given measurement of a given individual. The mapping is saved in `known-vals.rds`.
<br>
Once the lists of available measurements are made, one list for each RID, they are ready to potentially become columns in the combined ADNIMERGE data domain, so the type of measurement will be referred to as a column name or header for the remainder of the process. The headers for the soon-to-be ADNIMERGE data domain can be sorted by executing the following.
```
cd SortColNames
Rscript sort-col-names.r
cd ..
```
#### Run Time: 13 minutes 9.588 seconds
This uses the `known-vals.rds` file to create a list of sorted column names, saved in `sorted-col-names.rds`. They are sorted by the number of RIDs that have a known value for the corresponding column that will be created in the ADNIMERGE data domain. The list is sorted in descending order. This enables the creation of a table where the leftmost column has the most known values while the rightmost column has the least known values.
<br>
Now using both `known-vals.rds` and `sorted-col-names.rds`, the raw data set can be made. It’s called raw because further processing will occur later using the python language rather than R.
```
cd RawDataSet
Rscript raw-data-set.r
cd ..
```
#### Run Time: 11 minutes 27.046 seconds
This script iterates though each column name, each iteration of which iterates through each RID. If an RID has a known value for a given column, it is added. Otherwise, a value of `NA` (not available) is added to the column. The resulting data frame is saved in `raw-data-set.rds`.
<br>
Next, the type of each column must be known to determine which statistical test is appropriate for each pair of columns. The data frame of column types has the same headers as the raw data set except for RID and only has a single row that contains an indicator of whether the corresponding column is numeric or nominal. This data frame is stored in `col-types.rds` and is created by executing the following:
```
cd ColTypes
Rscript col-types.r
cd ..
```
#### Run Time: 3 minutes 40.904 seconds
Column types are determined by the contents of the column itself. If the values are numerical and there’s enough unique values (more than 10), the column is deemed numeric. If the values are not of a numeric type, it’s deemed nominal. But headers are not included in the column types data frame if the column has only one unique value. Headers are also not included if the corresponding column is not numeric but has too many unique values (at least 20), and therefore couldn’t reasonably represent a categorical variable. Such columns typically didn’t have useful information for statistical analysis, but were often notes taken by the data recorders which were full English sentences, clearly not variables, but still made their way into the `ADNIMERGE` tables. In this way, the column types data frame is used to filter the raw data set, excluding those columns that are unfit.
<br>
The final step is converting the column types data frame and the raw data set data frame into CSV files to be combined with the gene expression domain and MRI domain after further processing. These CSV files, stored as `col-types.csv` and `raw-data-set.csv`, are created by executing the following:
```
cd ToCSV
Rscript to-csv.r
cd ..
```
#### Run Time: 8.839 seconds
This script uses the headers of the column types data frame to select the acceptable columns from the raw data set, then both data frames are saved as CSV files. This concludes the process for putting together the raw data set for the ADNIMERGE domain. Return to the root of the repository by executing:
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
## Completing The ADNIMERGE Data Domain <a name="header4"></a>
The ADNIMERGE data domain goes through further processing (primarily imputing missing values) by executing the following:
```
cd DataClean/
bash jobs/phenotypes.sh adni
cd ..
```
#### Run Time: 1 hour 24 minutes 59 seconds
## Creating The Gene Expression Data Domain <a name="header5"></a>
The file `ADNI_Gene_Expression_Profile.zip` can be found at https://ida.loni.usc.edu/ under Download -> Genetic Data. It must be moved to `./data/gene_expression`. The contents can be extracted by executing the following:
```
cd data/gene_expression
unzip ADNI_Gene_Expression_Profile.zip
cd ../..
```
The gene expression data set can then be created by executing the following:
```
cd DataClean/
bash jobs/adni-expression.sh
```
#### Run Time: 2 minutes 25.821 seconds
## Creating The MRI Data Domain <a name="header6"></a>
To begin creating the MRI data domain, the directories containing the image files can be found at https://ida.loni.usc.edu/ under Download -> Image Collections. There are 2,432 directories, the name of each corresponds to a patient ID. All of these directories must be moved to `./data/mri/raw-adni/` for later steps. Most of these images will be filtered by taking the intersect of patient IDs with the patient IDs of the other domains (i.e. ADNIMERGE and gene expression). This can be accomplished by first obtaining the intersecting patient IDs just between the ADNIMERGE data domain and gene expression data domain. 744 patient IDs intersect between these two domains, meaning they have 744 individuals in common. To create a file containing these intersecting patient IDs, execute the following:
```
bash jobs/ptids.sh adni
```
#### Run Time: 52.114 seconds
Once the raw MRI data is in the correct data directory and the intersecting patient IDs are saved, the MRIs that have patient IDs that intersect with the prior saved patient IDs can be extracted and organized by executing the following:
```
bash jobs/med-adni-mri.sh
```
#### Run Time: 1 hour 1 minute 57 seconds
A total of 743 patient IDs (only 1 less than before) intersect between all three data domains, so there will be 743 individuals in the final data set. Since we are filtering the non-intersecting individuals, there will therefore also be 743 individuals in the MRI data domain. We don’t bother filtering the patients in the other data domains firstly because we couldn’t know what the MRI patient IDs are beforehand and secondly creating the other domains is not nearly as computationally intensive as creating the MRI data. Besides, those non-intersecting individuals in the ADNIMERGE and gene expression domains will necessarily be filtered when the three data domains are merged into one.
<br>
The MRIs are of the DICOM file format, a common file format for medical images, which contains metadata in addition to the images themselves. These DICOM medical images are handled using the `Pydicom` python package. More information about DICOM files and the Pydicom package can be found at https://pydicom.github.io. The medical images, however, are not in a format that’s ideal for deep learning. They can be converted to PNG format by executing the following:
```
bash jobs/png-mri.sh adni
```
#### Run Time: 12 hours 24 minutes 29 seconds
The medical images are converted to PNG format using a python package called `med2image`. More information about this package can be found at https://pypi.org/project/med2image/. After the images are organized and converted into a usable format, they then need to be further processed by executing the following:
```
bash jobs/txt-mri.sh adni
```
#### Run Time: 1 hours 14 minutes 3 seconds
This processing includes normalizing, resizing, and splicing. The normalizing step is a simple min-max normalization of the pixel values. The resizing is performed using the OpenCV python package. The images are processed as multi-dimensional arrays provided by the `NumPy` python package. The splicing is necessary because there is not a single image per individual, but rather, a sequence of images. Sagital MRIs were captured in a sequence of images ranging from one side of the skull to the other. The first images in each sequence are of layers on one side, later images are of layers in the middle, and the last images are of the other side. These layers are known as slices. Different individuals in the data have different numbers of slices, so they have different numbers of images in their sequence. The smallest sequence length is 124 slices, and those individuals with larger sequence lengths have their sequences spliced to 124 for consistency. After being processed, the images are saved as txt files and are finally ready for the deep convolutional neural network.
<br>
Since the individual samples have sequences of images rather than single images, a separate convolutional autoencoder is trained for each slice index. Since there are 124 slices per individual, there are 124 slice indices, ranging from 0 to 123, where 0 refers to the first slice in the sequence and 123 refers to the last (zero-based numbering). As stated above, there is 743 individuals in the MRI data domain, each of which has their sequence from slices 0 to 123, so there are 743 index 0 images, 743 index 1 images, all the way to 743 index 123 images for a total of 743 * 124 = 92,132 images. Since there are 743 images for each slice index, and there is a separate trained autoencoder per slice index, each autoencoder is trained on 743 images.
<br>
To train the convolutional auto-encoders, execute the following:
```
bash jobs/conv-autoencoder.sh 0 adni
```
The 0 argument in the above command refers to the first slice index, meaning the model corresponding to the 0 slice index is trained. This same command must additionally be ran using a slice index of 1, 2, and all the way to 123. **Note that each of these runs requires a 16 gigabyte GPU.**
<br>
Once the auto-encoders are trained, they can be used to compress the images into one-dimensional latent vectors which are concatenated together to become the rows in the table for the MRI domain. Execute the following to do this:
```
bash jobs/mri-table.sh adni
```
#### Run Time: 1 hour 52 minutes 36 seconds
## Creating The Final Data Set <a name="header7"></a>
**Creating the table for the MRI domain also uses a 16 gigabyte GPU but 128 gigabytes of CPU memory was allocated.** After the MRI data domain is complete, it can be merged with the ADNIMERGE domain and gene expression domain by executing the following:
```
bash jobs/combine.sh adni combined processed-data/datasets/adni/mri.csv
```
#### Run Time: 3 hours 32 minutes 53 seconds
**The merging of the three domains required approximately over a terabyte of memory.** The final data set is now complete and ready for correlation analysis. Enter the data directory of the `CorrelationAnalysis` sub module by executing the following:
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
