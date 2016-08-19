##Merges the training and the test sets to create one data set.

###Read the files

Read the subject files.

      dtSubjectTrain <- fread(file.path(pathIn, "train", "subject_train.txt"))
      dtSubjectTest <- fread(file.path(pathIn, "test", "subject_test.txt"))\
Read the activity 

      dtActivityTrain <- fread(file.path(pathIn, "train", "Y_train.txt"))
      dtActivityTest <- fread(file.path(pathIn, "test", "Y_test.txt"))
      
Read the data files. fread seems to be giving me some trouble reading files. Using a helper function, read the file with 
  read.table instead, then convert the resulting data frame to a data table. Return the data table.
  
  
     fileToDataTable <- function(f) {
      df <- read.table(f)
      dt <- data.table(df)
     }
      dtTrain <- fileToDataTable(file.path(pathIn, "train", "X_train.txt"))
      dtTest <- fileToDataTable(file.path(pathIn, "test", "X_test.txt"))
  
### Merge Files
Concatenate the data tables.

      dtSubject <- rbind(dtSubjectTrain, dtSubjectTest)
      setnames(dtSubject, "V1", "subject")
      dtActivity <- rbind(dtActivityTrain, dtActivityTest)
      setnames(dtActivity, "V1", "activityNum")
      dt <- rbind(dtTrain, dtTest)
Merge columns.

      dtSubject <- cbind(dtSubject, dtActivity)
      dt <- cbind(dtSubject, dt)
Set key.  

      setkey(dt, subject, activityNum)
      
##Extracts only the measurements on the mean and standard deviation for each measurement.
Read the features.txt file. This tells which variables in dt are measurements for the mean and standard deviation.

      dtFeatures <- fread(file.path(pathIn, "features.txt"))
      setnames(dtFeatures, names(dtFeatures), c("featureNum", "featureName"))

Subset only measurements for the mean and standard deviation.

      dtFeatures <- dtFeatures[grepl("mean\\(\\)|std\\(\\)", featureName)]

Convert the column numbers to a vector of variable names matching columns in dt.

    dtFeatures$featureCode <- dtFeatures[, paste0("V", featureNum)]
    head(dtFeatures)      

Output looks like 

            featureNum       featureName featureCode
            1:          1 tBodyAcc-mean()-X          V1
            2:          2 tBodyAcc-mean()-Y          V2
            3:          3 tBodyAcc-mean()-Z          V3
            4:          4  tBodyAcc-std()-X          V4
            5:          5  tBodyAcc-std()-Y          V5
            6:          6  tBodyAcc-std()-Z          V6

      dtFeatures$featureCode
      
      
        [1] "V1"   "V2"   "V3"   "V4"   "V5"   "V6"   "V41"  "V42"  "V43"  "V44"  "V45"  "V46"  "V81"  "V82"  "V83"  "V84"  "V85"  "V86" 
      [19] "V121" "V122" "V123" "V124" "V125" "V126" "V161" "V162" "V163" "V164" "V165" "V166" "V201" "V202" "V214" "V215" "V227" "V228"
      [37] "V240" "V241" "V253" "V254" "V266" "V267" "V268" "V269" "V270" "V271" "V345" "V346" "V347" "V348" "V349" "V350" "V424" "V425"
      [55] "V426" "V427" "V428" "V429" "V503" "V504" "V516" "V517" "V529" "V530" "V542" "V543"


Subset these variables using variable names.

      select <- c(key(dt), dtFeatures$featureCode)
      dt <- dt[, select, with = FALSE]

## Uses descriptive activity names to name the activities in the data set.
## Appropriately labels the data set with descriptive activity names.
## Creates a second, independent tidy data set with the average of each variable for each activity and each subject.
