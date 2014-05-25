Course project step by step solution

##read the activity data with id and labels
activity_df=read.table("./UCI HAR Dataset/activity_labels.txt",sep=" ")
activity_df
names(activity_df)<-c("id","activity_label")

##reading the subject test data and naming the column as subject_id
subject_test=read.table("./UCI HAR Dataset/test/subject_test.txt")
names(subject_test)<-c("subject_id")


##reading the subject train data and naming the column as subject_id
subject_train=read.table("./UCI HAR Dataset/train/subject_train.txt")
names(subject_train)<-c("subject_id")
subject_train

##merge the subjects from both test and train
subject_final<-rbind(subject_test,subject_train)
dim(subject_final)

##read the measurement labels and format the names properly by removing special chars and caps
measurements_labels<-read.table("./UCI HAR Dataset/features.txt",sep=" ")
measurements_labels[,2]<-gsub("-",".",measurements_labels[,2])
measurements_labels[,2]<-gsub("\\()","",measurements_labels[,2])
measurements_labels[,2]<-tolower(measurements_labels[,2])
measurements_labels[,2]

##reading the X train and test data merge them and name the columns as formatted in the measurement labels 
x_test<-read.table("./UCI HAR Dataset/test/X_test.txt",sep="")
x_train<-read.table("./UCI HAR Dataset/train/X_train.txt",sep="")
x_final<-rbind(x_test,x_train)
names(x_final)<-measurements_labels[,2]
dim(x_final)

##read the Y train and test data and merge them
y_test<-read.table("./UCI HAR Dataset/test/Y_test.txt",sep="")
y_train<-read.table("./UCI HAR Dataset/train/Y_train.txt",sep="")
y_final<-rbind(y_test,y_train)
dim(y_final)
names(y_final)<-c("id")

##merge the Y data with the respective activity names
y_labels_final<-merge(y_final,activity_df,by="id")

##combine the X,Y and subject data into a single dataset
final_dataset<-cbind(subject_final,y_labels_final,x_final)
names(final_dataset)


##read only mean and std measurements
final_dataset_means_stds<-final_dataset[,grep("mean\\.|std\\.",colnames(final_dataset))]
names(final_dataset_means_stds)
final_dataset_means_stds<-cbind(subject_final,y_labels_final,final_dataset_means_stds)
final_dataset_means_stds$id<-NULL
names(final_dataset_means_stds)

##melting the dataset by only keeping subject_id and activity label as the variables and melting other measurements
library(reshape2)
melted_dataset<-melt(final_dataset_means_stds,id.vars=c("subject_id","activity_label"),variable.name = "variable",value.name = "value")
head(melted_dataset)

##find the mean of the filtered out measurements melted in the previous step
install.packages("reshape2")
install.packages("plyr")
library(plyr)
final_tidy_dataset<-ddply(melted_dataset,.(subject_id,activity_label,variable),summarise,`mean`=mean(value))
head(final_tidy_dataset)

##writing the tidy dataset to a output text file.
write.table(final_tidy_dataset,file="tidy_data_set.txt",sep=" ",quote=FALSE,row.names=FALSE)
