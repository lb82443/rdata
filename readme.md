Documentation for creation of tidy_data.csv
========================================================

This data is based on analysis as documented in the paper below:

[1] Davide Anguita, Alessandro Ghio, Luca Oneto, Xavier Parra and Jorge L. Reyes-Ortiz. Human Activity Recognition on Smartphones using a Multiclass Hardware-Friendly Support Vector Machine. International Workshop of Ambient Assisted Living (IWAAL 2012). Vitoria-Gasteiz, Spain. Dec 2012

The data represents information from various subjects relative to their activities. These activities are documented in the tidy data set below and are:

1 WALKING
2 WALKING_UPSTAIRS
3 WALKING_DOWNSTAIRS
4 SITTING
5 STANDING
6 LAYING

A wide variety of measures were gathered from the accelerometer of a Samsung smartphone and captured in the data gathered below.

The final variables and definitions in tidy_data.csv are defined in the CodeBook.md in the repositor on Git Hub.

Below is the detailed documentation of the process of creating the tidy_data.csv file.


## set the base directory used to performa the analysis
setwd("C:\\Users\\Lou\\rdata")

## in this step we will download the zip file
## data will be unzipped manually to appropriate directory....in this case the 
## \data directory under the woking directory set above.

fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"

download.file(fileUrl,destfile="data\\dataset.zip",mode="wb")
 

## Lets read in file of variable names and then load into vector to use to add 
## variables to the large test file

varDF = read.table("data\\features.txt", header=FALSE,col.names=c("id","Vname"))

vctrname = as.vector(varDF[,2])

## Lets read the file of activities to match back to the datasets
## I'm going to hold this data frame to the very end
## becuase if I match too soon the sort sequence is changed
## and cbind() will not work.

actlblDF = read.table("data\\activity_labels.txt", header=FALSE,col.names=c("activity","act_label"))

########################################################
## Begin reading in raw files for test datasets        #
########################################################

## read the data table for test data
tstDF = read.table("data\\test\\X_test.txt", header=FALSE, col.names=vctrname)

## read the subject ID's for the test data
tstidDF = read.table("data\\test\\subject_test.txt", header=FALSE, col.names=c("subject"))

## read the activities for the test data
tstactDF = read.table("data\\test\\y_test.txt", header=FALSE, col.names=c("activity"))

## had to use this command to see more than 100 columns in Rstudio
utils::View(tstDF)

## documentation stated that the data was gathered in order so I was able to column bind
## to get a combined data frame

test_DF = cbind(tstidDF,tstactDF,tstDF)

utils::View(test_DF)

########################################################
## Begin reading in raw files for training datasets    #
########################################################
## read the data table for training data
trnDF = read.table("data\\train\\X_train.txt", header=FALSE, col.names=vctrname)

## read the subject ID's for the training data
trnidDF = read.table("data\\train\\subject_train.txt", header=FALSE, col.names=c("subject"))

## read the activities for the training data
trnactDF = read.table("data\\train\\y_train.txt", header=FALSE, col.names=c("activity"))

## had to use this command to see more than 100 columns in Rstudio
utils::View(trnDF)

## documentation stated that the data was gathered in order so I was able to column bind
## to get a combined data frame

train_DF = cbind(trnidDF,trnactDF,trnDF)

utils::View(train_DF)

## Now we rbind() the training and test data frames to create one large file

final_DF = rbind(test_DF,train_DF)

utils::View(final_DF)


## now we need to add activity labels to the final_DF
## I waited until now because the order of the data frame will be  changed
## after the merge runs. 

lbld_DF = merge(final_DF,actlblDF,by.x="activity",by.y="activity",all=TRUE  )

utils::View(lbld_DF)


## Now we need to select only those variables related to mean and standard deviation
## based on visual inspection of column names I'll search for Mean mean and std
## I selected this broad definition to err on the side of including more data than
## may have been necessary. Some of the definitions in the paper were vague.



cln_DF <- lbld_DF[, grep("subject|act_label|Mean|mean|std", names(lbld_DF), value=TRUE)]

utils::View(cln_DF)

## Now we do our final summary by activity label and subject calculate the mean for the values above

sum_DF <- aggregate(cln_DF, by=list(subj_id=cln_DF$subject, act_lbl=cln_DF$act_label), FUN=mean)

## Drop variables we don't need since aggregate creates two new ones

drops <- c("subject","act_label")
out_DF = sum_DF[,!(names(sum_DF) %in% drops)]

## Write out final file as a csv
## drink a beer

write.csv(out_DF, "tidy_data.csv", row.names=FALSE)



