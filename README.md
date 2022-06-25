# BigDataProject

## Project description

The project is about predicting the number of recommendations of a comment performed on New York Times Comments dataset that can be found under the link https://www.kaggle.com/datasets/aashita/nyt-comments. Main challanges of this project are handling large data volume and feature engineering.

## Database

### Description

The dataset consists of two .csv files containing data about articles and comments under said articles. Our main interest are the comments, but we also extract features from the "parent" article of every comment such as:
- abstract
- keywords
- date of publication

From the comments themselves the extracted features are:
- comment body
- date of creation
- number of replies

The fact that we use the dates as features might seem unreasonable (since if we'd like to use our model in the future the past dates will never appear again), but our goal here is to determine the time elapsed from publication to posting a comment (which we get by substracting date of publication from date of comment cretion).

### Preprocessing

The initial data didn't need much preprocessing except removing the outliers when it comes to the number of recommendations (we used quantiles to do that).
As stated before, we determined the time elapsed from publication to posting a comment. The values we got were quite high (difference between two Unix timestamps) so we normalized them between 0 and 1.

### Database schema

![schema](https://user-images.githubusercontent.com/62818677/175786969-d272793c-cba8-4393-ae6b-7529c1c6eba3.png)

### PySpark

The database is set up on 8 nodes.

## Feature extraction

The features are extracted by creating vectors from them. If the type of a feature is 'string', it first undergoes tokenization and stopwords removal before being vectorized. Then, we concatenate the vectors and numerical features together to create 202-dimensional vectors for each comment. The vectors then undergo principal component analysis (PCA) to reduce the dimensionality. Finally, we end up with 30 features for each comment.

## Classifier

To predict the number of recommendadtions we utilize a random forest classifier from PySpark. Since the number of recommendations can vary between 0 and ~30 (after removing outliers), we decided to bucketize it to four classes.

## Problems encountered

We had some major issues for which we used up quite a lot of time. First, we discovered that the dataset contained combinations of commas and quotes that messed up the PySpark's .csv default reading method. Luckily we were able to easily resolve it by changing some options in said method (but finding out which options should be changed took some time).
Another problem was with Null values in the article abstracts and keywords. We didn't want to simply drop the rows that caused issues but replace the Nulls with mean values from the column - this turned out to be quite troublesome to do with DenseVectors inside a PySpark dataframe. Regular pandas-esque methods didn't work here, but luckily we managed to do it using pyspark.sql.functions.udf (user defined function).
