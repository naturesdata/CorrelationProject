# Evaluating Feature Correlation in the Alzheimer's Disease Neuroimaging Initiative
## Table of Contents
1. [Command Line Tools Required For This Project](#table1)
2. [Downloadable ADNI Files Needed](#table2)
3. [Cloning The Project Repository](#header1)
4. [Creating The Raw ADNIMERGE Data Domain](#header2)
5. [Creating The Python Virtual Environment](#header3)
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
