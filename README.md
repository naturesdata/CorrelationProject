# Evaluating Feature Correlation in the Alzheimer's Disease Neuroimaging Initiative
## Table of Contents
1. [Command Line Tools Required For This Project](#table1)
2. [Downloadable ADNI Files Needed](#table2)
3. [Cloning The Project Repository](#header1)
4. [Creating The Raw ADNIMERGE Data Domain](#header2)
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
