# Getting and Cleaning Data Project
Author: Roman Sujatinov

Original data was obtained from: http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

## Reading data

Data for this project if broken down into two different groups: training data and test data. Furthermore, withing each group, data is stored in three different files:
* X_test.txt (X_train.txt)
* y_test.txt (y_train.txt)
* subject_test.txt (subject_train.txt)

After inspecting data, we can see that files above does not have appropriate headers. To fix this issue we'll do the following:
* Headers for X.txt can be found in another file - features.txt
* y.txt contains only one column, which corresponds to activity type
* subject.txt contains only one column, which corresponds to subject id

First, we'll read column names for X dataset from features.txt:

`features = read.table("features.txt", col.names = c("index", "feature"))`

Next, we'll read X dataset for both test and training groups, specifiying col.names:

`x.test = read.table("X_test.txt", col.names = features$feature)`

`x.train = read.table("X_train.txt", col.names = features$feature)`

Then, wel'll read Y datasets for both test and training groups, specifying column name "activity":
`y.test = read.table("y_test.txt", col.names = "activity")`

`y.train = read.table("y_train.txt", col.names = "activity")`


Then, wel'll read Y datasets for both test and training groups, specifying column name "subject":

`subject.test = read.table("subject_test.txt", col.names = "subject")`

`subject.train = read.table("subject_train.txt", col.names = "subject")`

## Merging Data

By analyzing raw data documentation and data dimensions, we see that data should be merged in the following way:

1. Within test and training groups: subject and activity columns should be prepended to X dataset

2. Across test and training groups: merged datasets from step 1 have the same columns, and they should be combined in a single dataset by combining rows

Let's merge data within datasets using columns binding:

`test_data = cbind(subject.test, y.test, x.test)`

`train_data = cbind(subject.train, y.train, x.train)`

Let's merge data across datasets using rows binding:

`full_data = tbl_df(rbind(test_data, train_data))`


## Reducing data

According to the task, we are interested only in columns that have "mean" or "std" words in their name.

Let's filter needed columns:

`col_names = grep("mean|std", names(full_data), value = TRUE)`

And keep only columns of interest:

`data = full_data[, c("activity", "subject", col_names)]`

## Tidying Data (part 1)

### Descriptive Activity Names
Currently we see that activity are represented by numbers. We want to have descriptive activity names. 

Let's read activity labels from corresponding file:

`activity_labels = read.table("activity_labels.txt", col.names = c("id", "label"))`

And substitute activity labels instead of numbers:

`data$activity = activity_labels[data$activity, "label"]`

Also, we see that many column names are not well formatted, let's make them more user friendly, by specifying "time" instead of "t" prefix, "freq" instead of "f" prefix, filtering out "..." and "..",  and removing "." in the end of column names:

`colnames(data) = names(data) %>%
    str_replace("^t", "time") %>% 
    str_replace("^f", "freq") %>%
    str_replace("\\.\\.\\.", ".") %>%
    str_replace("\\.\\.", ".") %>%
    str_replace("\\.$", "")`

### Tidy Dataset
Finally, we need to create a second independent tidy data set with the average of each variable for each activity and each subject.

`tidy_data = data %>%
    group_by(subject, activity) %>%
    summarize_all(mean)`

Let's save out tidy data for submission

`write.table(tidy_data, file="tidy_data.txt", row.name=FALSE)`



