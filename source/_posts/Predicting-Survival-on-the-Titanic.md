---
title: 'Predicting Survival on the Titanic'
author: "Warren Elder"
date: '2017-12-06'
categories:
- Data science
tags:
- Kaggle
- R
- Binary classification
output:
  html_document:
    number_sections: true
    fig_caption: true
    toc: true
    fig_width: 7
    fig_height: 4.5
    theme: cosmo
    highlight: tango
    code_folding: show
clearReading: true
metaAlignment: center
coverImage: iceberg.jpg
coverMeta: in
coverSize: partial
comments: false
meta: false
actions: false
---

A common introduction to Kaggle competitions is to analyse the Titanic dataset to understand why certain people aboard perished, and others did not. I recently shared my attempt on {% link Kaggle https://www.kaggle.com/warrenelder/titanic-survival-prediction [https://www.kaggle.com/warrenelder/titanic-survival-prediction] [Titanic: Survival Prediction] %}, but you can read my Rmarkdown report here too.
<!-- more -->


```r
knitr::opts_chunk$set(echo=TRUE, error=FALSE)
```

# Introduction

This is my first attempt at analysing a Kaggle dataset. My principle aim here is to pick up as many methods and techniques while investigating this type of classification problem. I am also relatively new to using *R* language. To that end, any input/suggestions that would improve the analysis would be most welcome.

The Titanic is a well known ship that floundered on it's maiden voyage after striking an ice-berg in the North Atlantic in 1912. There was a large loss of life which was worsened by insufficient lifeboats. The aim of this challenge is to predict the probability of whether a given person survived this tragedy or not.

The [data](https://www.kaggle.com/c/titanic/data) comes in the form of one training and test file each: `../input/train.csv` & `../input/test.csv`. Each row corresponds to a specific passenger on the ship and the columns describe their features.

Additional data has been created on passengers nationality, and is detailed in section 4.5.

# Preparations

## Load libraries

We load a range of libraries for general data wrangling and general visualisation together with more specialised tools.



## Load data

We use *data.table's* fread function to speed up reading in the data, even though in this challenge our files are small, under 100KB for both *train* and *test*.


```r
train <- as.tibble(fread('train.csv'))
test <- as.tibble(fread('test.csv'))
```

# File structure and data overview

As a first step let's have an overview of the data sets using the *summary* and *glimpse* tools.

## Training data


```r
summary(train)
```

```
##   PassengerId     Survived         Pclass         Name          
##  Min.   :  1   Min.   :0.000   Min.   :1.00   Length:891        
##  1st Qu.:224   1st Qu.:0.000   1st Qu.:2.00   Class :character  
##  Median :446   Median :0.000   Median :3.00   Mode  :character  
##  Mean   :446   Mean   :0.384   Mean   :2.31                     
##  3rd Qu.:668   3rd Qu.:1.000   3rd Qu.:3.00                     
##  Max.   :891   Max.   :1.000   Max.   :3.00                     
##                                                                 
##      Sex                 Age            SibSp           Parch      
##  Length:891         Min.   : 0.42   Min.   :0.000   Min.   :0.000  
##  Class :character   1st Qu.:20.12   1st Qu.:0.000   1st Qu.:0.000  
##  Mode  :character   Median :28.00   Median :0.000   Median :0.000  
##                     Mean   :29.70   Mean   :0.523   Mean   :0.382  
##                     3rd Qu.:38.00   3rd Qu.:1.000   3rd Qu.:0.000  
##                     Max.   :80.00   Max.   :8.000   Max.   :6.000  
##                     NA's   :177                                    
##     Ticket               Fare          Cabin          
##  Length:891         Min.   :  0.0   Length:891        
##  Class :character   1st Qu.:  7.9   Class :character  
##  Mode  :character   Median : 14.5   Mode  :character  
##                     Mean   : 32.2                     
##                     3rd Qu.: 31.0                     
##                     Max.   :512.3                     
##                                                       
##    Embarked        
##  Length:891        
##  Class :character  
##  Mode  :character  
##                    
##                    
##                    
##
```


```r
glimpse(train)
```

```
## Observations: 891
## Variables: 12
## $ PassengerId <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14...
## $ Survived    <int> 0, 1, 1, 1, 0, 0, 0, 0, 1, 1, 1, 1, 0, 0, 0, ...
## $ Pclass      <int> 3, 1, 3, 1, 3, 3, 1, 3, 3, 2, 3, 1, 3, 3, 3, ...
## $ Name        <chr> "Braund, Mr. Owen Harris", "Cumings, Mrs. Joh...
## $ Sex         <chr> "male", "female", "female", "female", "male",...
## $ Age         <dbl> 22, 38, 26, 35, 35, NA, 54, 2, 27, 14, 4, 58,...
## $ SibSp       <int> 1, 1, 0, 1, 0, 0, 0, 3, 0, 1, 1, 0, 0, 1, 0, ...
## $ Parch       <int> 0, 0, 0, 0, 0, 0, 0, 1, 2, 0, 1, 0, 0, 5, 0, ...
## $ Ticket      <chr> "A/5 21171", "PC 17599", "STON/O2. 3101282", ...
## $ Fare        <dbl> 7.250, 71.283, 7.925, 53.100, 8.050, 8.458, 5...
## $ Cabin       <chr> "", "C85", "", "C123", "", "", "E46", "", "",...
## $ Embarked    <chr> "S", "C", "S", "S", "S", "Q", "S", "S", "S", ...
```

We find:

- There are a number of features here. In total, our *training* data has 12 variables, including *PassengerId* and *Survived*. In some of them we already see a number of missing values.
- Note the [data description](https://www.kaggle.com/c/titanic/data) provides ancillary information on keys and labeling.

## Test data


```r
summary(test)
```

```
##   PassengerId       Pclass         Name               Sex           
##  Min.   : 892   Min.   :1.00   Length:418         Length:418        
##  1st Qu.: 996   1st Qu.:1.00   Class :character   Class :character  
##  Median :1100   Median :3.00   Mode  :character   Mode  :character  
##  Mean   :1100   Mean   :2.27                                        
##  3rd Qu.:1205   3rd Qu.:3.00                                        
##  Max.   :1309   Max.   :3.00                                        
##                                                                     
##       Age            SibSp           Parch          Ticket         
##  Min.   : 0.17   Min.   :0.000   Min.   :0.000   Length:418        
##  1st Qu.:21.00   1st Qu.:0.000   1st Qu.:0.000   Class :character  
##  Median :27.00   Median :0.000   Median :0.000   Mode  :character  
##  Mean   :30.27   Mean   :0.447   Mean   :0.392                     
##  3rd Qu.:39.00   3rd Qu.:1.000   3rd Qu.:0.000                     
##  Max.   :76.00   Max.   :8.000   Max.   :9.000                     
##  NA's   :86                                                        
##       Fare          Cabin             Embarked        
##  Min.   :  0.0   Length:418         Length:418        
##  1st Qu.:  7.9   Class :character   Class :character  
##  Median : 14.5   Mode  :character   Mode  :character  
##  Mean   : 35.6                                        
##  3rd Qu.: 31.5                                        
##  Max.   :512.3                                        
##  NA's   :1
```


```r
glimpse(test)
```

```
## Observations: 418
## Variables: 11
## $ PassengerId <int> 892, 893, 894, 895, 896, 897, 898, 899, 900, ...
## $ Pclass      <int> 3, 3, 2, 3, 3, 3, 3, 2, 3, 3, 3, 1, 1, 2, 1, ...
## $ Name        <chr> "Kelly, Mr. James", "Wilkes, Mrs. James (Elle...
## $ Sex         <chr> "male", "female", "male", "male", "female", "...
## $ Age         <dbl> 34.5, 47.0, 62.0, 27.0, 22.0, 14.0, 30.0, 26....
## $ SibSp       <int> 0, 1, 0, 0, 1, 0, 0, 1, 0, 2, 0, 0, 1, 1, 1, ...
## $ Parch       <int> 0, 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, ...
## $ Ticket      <chr> "330911", "363272", "240276", "315154", "3101...
## $ Fare        <dbl> 7.829, 7.000, 9.688, 8.662, 12.287, 9.225, 7....
## $ Cabin       <chr> "", "", "", "", "", "", "", "", "", "", "", "...
## $ Embarked    <chr> "Q", "S", "Q", "S", "S", "S", "Q", "S", "C", ...
```

We find:

- Our *test* data has 11 variables, as it excludes the *Survived* feature which is the object of interest in our analysis.

## Missing values


```r
colSums(is.na(train))
```

```
## PassengerId    Survived      Pclass        Name         Sex
##           0           0           0           0           0
##         Age       SibSp       Parch      Ticket        Fare
##         177           0           0           0           0
##       Cabin    Embarked
##           0           0
```

```r
colSums(is.na(test))
```

```
## PassengerId      Pclass        Name         Sex         Age
##           0           0           0           0          86
##       SibSp       Parch      Ticket        Fare       Cabin
##           0           0           0           1           0
##    Embarked
##           0
```

There are 177 and 87 missing values for the *training* data and *test* data, respectively. The majority are missing ages but there is also a missing fare.

# Investigate how individual features impact survival

We first investigate our feature space to gain a better understanding on how they relate to the predictor and one another. Lets first understand the number of passengers that survived versus those that did not.


```r
prop.table(table(train$Survived))
```

```
##
##      0      1
## 0.6162 0.3838
```

In the training data set 342 passengers out of 891 survived. Assuming the training data is a fair representation of the full data set, then approximately 38 out of every hundred passengers on the Titanic survived.

## Impact of gender on survival
Lets investigate the proportion of men and women who survived.


```r
prop.table(table(train$Sex, train$Survived), margin = 1)
```

```
##         
##               0      1
##   female 0.2580 0.7420
##   male   0.8111 0.1889
```

We find:

- 74% of females survived while only 19% of males survived. So gender appears to have an impact on a passengers survival rate.

## Impact of ticket class and fare on survival.

Impact of Ticket Class:


```r
prop.table(table(train$Pclass, train$Survived), margin = 1)
```

```
##    
##          0      1
##   1 0.3704 0.6296
##   2 0.5272 0.4728
##   3 0.7576 0.2424
```

We find:

- Passengers in first class have the highest rate of survival.
- The rate of survival reduces for passengers with the lower class of ticket.

Impact of Fare:

It can be expected that there is some degree of correlation between the *Fare* and *Pclass* variables with the first class passengers paying the highest fares. If this assumption holds true then we might expect a higher chance of survival with increasing fare. So lets see if that's the case. We shall examine the training and test data together as we first wish to understand the correlation between *Fare* and *Pclass* variables at the moment.


```r
all_dataInvestigate <- bind_rows(train, test)
# We have a missing fare for passenger 1044 which we set to the median fare
all_dataInvestigate$Fare[1044] <- median(all_dataInvestigate$Fare, na.rm = TRUE)
# Plot data
fig.4.2.a <- ggplot(all_dataInvestigate, aes(x = factor(Pclass), y = Fare)) +
  geom_boxplot() +
  theme(axis.line.y = element_line(colour = "black"),
        panel.grid.major = element_blank(),
        panel.grid.minor = element_blank(),
        panel.border = element_blank(),
        panel.background = element_blank(),
        panel.grid.major.y = element_line(colour = "grey"),
        legend.position="none") +
  labs(x = "Pclass", y = "Fare (units)", title = "Fig 4.2.a: Fare box plots for each passenger class.")
plot(fig.4.2.a)
```

![plot of chunk unnamed-chunk-11](/figure/posts/Predicting-Survival-on-the-Titanic/unnamed-chunk-11-1.png)

We do see an increase in the mean fare price as we consider passengers in higher classes(*Pclass* = 1 being first or highest class). Interestingly, we also observe outlier fares in each of the classes which indicate some passengers in lower classes pay more for their ticket than in the upper classes. Let see an example of one of these outlier fares to see why this may be the case.

```r
filtered <- all_dataInvestigate %>% filter((Pclass==2 & Fare > 30.0)) %>% arrange(desc(Fare)) %>% head(10)
kable(filtered, caption="Second class (Pclass=2) passengers with ticket Fares greater than 30.00, ordered by fare")
```



| PassengerId| Survived| Pclass|Name                             |Sex    | Age| SibSp| Parch|Ticket       | Fare|Cabin |Embarked |
|-----------:|--------:|------:|:--------------------------------|:------|---:|-----:|-----:|:------------|----:|:-----|:--------|
|          73|        0|      2|Hood, Mr. Ambrose Jr             |male   |  21|     0|     0|S.O.C. 14879 | 73.5|      |S        |
|         121|        0|      2|Hickman, Mr. Stanley George      |male   |  21|     2|     0|S.O.C. 14879 | 73.5|      |S        |
|         386|        0|      2|Davies, Mr. Charles Henry        |male   |  18|     0|     0|S.O.C. 14879 | 73.5|      |S        |
|         656|        0|      2|Hickman, Mr. Leonard Mark        |male   |  24|     2|     0|S.O.C. 14879 | 73.5|      |S        |
|         666|        0|      2|Hickman, Mr. Lewis               |male   |  32|     2|     0|S.O.C. 14879 | 73.5|      |S        |
|        1104|       NA|      2|Deacon, Mr. Percy William        |male   |  17|     0|     0|S.O.C. 14879 | 73.5|      |S        |
|        1244|       NA|      2|Dibden, Mr. William              |male   |  18|     0|     0|S.O.C. 14879 | 73.5|      |S        |
|         616|        1|      2|Herman, Miss. Alice              |female |  24|     1|     2|220845       | 65.0|      |S        |
|         755|        1|      2|Herman, Mrs. Samuel (Jane Laver) |female |  48|     1|     2|220845       | 65.0|      |S        |
|        1122|       NA|      2|Sweet, Mr. George Frederick      |male   |  14|     0|     0|220845       | 65.0|      |S        |

It is apparent that some passengers have purchased tickets as a group (they have the same ticket number). These passengers are usually part of a family or maybe friends. Observe the same surnames in the *Name* column, or non zero values in the number of spouse or parent and siblings variables (*Parch* or *SibSp*). A more accurate way of representing the fare would be to calculate the fare-per-person. We perform this calculation on only those passengers which have the same Fare and Ticket number. We then re-plot the results as a factored *Pclass* boxplot.

Note- we must consider both the train and test data sets together to determine all of the passengers that have a group fare.


```r
all_dataInvestigate <- all_dataInvestigate %>%
  group_by(Ticket, Fare) %>%
  dplyr::mutate(FarePerPerson = Fare / n())

fig.4.2.b <- ggplot(all_dataInvestigate, aes(x = factor(Pclass), y = FarePerPerson)) +
  geom_boxplot() +
    theme(axis.line.y = element_line(colour = "black"),
        panel.grid.major = element_blank(),
        panel.grid.minor = element_blank(),
        panel.border = element_blank(),
        panel.background = element_blank(),
        panel.grid.major.y = element_line(colour = "grey"),
        legend.position="none") +
  labs(x = "Pclass", y = "Fare per person (units)", title = "Fig 4.2.b: Fare-per-person box plots for each passenger class.")
plot(fig.4.2.b)
```

![plot of chunk unnamed-chunk-13](/figure/posts/Predicting-Survival-on-the-Titanic/unnamed-chunk-13-1.png)

Examining *Fig 4.2.b*, we see the number of outliers and the length of the box whiskers have been reduced. There is now a much clearer fare distinction between the passenger classes with little or no overlap between the whiskers of each box plot. Also apparent by the reduction of whisker size is the zero fare outliers. These passengers may have got free tickets, but is more likely they are null values encoded as 0.

To determine the correlation between the variable of *Pclass* and *FarePerPerson* we use Pearsons Correlation Coefficient, as we have ordinal and continuous variables, respectively. We do this for both the *Fare* variable and our engineered feature *FarePerPerson* and compare coefficients.


```r
trainInvestigate <- all_dataInvestigate[1:891,]
# Pearson Correlation for Fare and Pclass
cor.test(trainInvestigate$Pclass, trainInvestigate$Fare, method = "pearson", use = "complete.obs")
```

```
##
## 	Pearson's product-moment correlation
##
## data:  trainInvestigate$Pclass and trainInvestigate$Fare
## t = -20, df = 890, p-value <2e-16
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  -0.5937 -0.5019
## sample estimates:
##     cor
## -0.5495
```

```r
# Pearson Correlation for FarePerPerson and Pclass
cor.test(trainInvestigate$Pclass, trainInvestigate$FarePerPerson, method = "pearson", use = "complete.obs")
```

```
##
## 	Pearson's product-moment correlation
##
## data:  trainInvestigate$Pclass and trainInvestigate$FarePerPerson
## t = -35, df = 890, p-value <2e-16
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  -0.7888 -0.7337
## sample estimates:
##     cor
## -0.7627
```

We find:

- Pearson coefficient between *Pclass* and *Fare* is -0.55.
- Pearson coefficient between *Pclass* and *FarePerPerson* are -0.76.
- The *FarePerPerson* variable is more highly correlated with *Pclass* after removing the impact of Group tickets.
- As the *Pclass* and *FarePerPerson* variables have a relatively strong correlation, we should consider the impact of multicollininarity on our chosen model in section 5.

Let's now consider the impact of *FarePerPerson* on survival. Note- we only consider the training data and those *FarePerPerson* greater than 0.


```r
# Select fares above 0
trainInvestigate_farePerPerson <- trainInvestigate[(trainInvestigate$FarePerPerson>0.0), ]
# Group fares in 5 unit intervals
trainInvestigate_farePerPerson$fareGroup <- cut(trainInvestigate_farePerPerson$FarePerPerson, seq(0, 130, 5), include.lowest=T)
# Plot
fig.4.2.c <- ggplot(trainInvestigate_farePerPerson, aes(x = fareGroup)) +
  geom_bar(stat='count', aes(fill = factor(Survived)), position = 'fill') +
      theme(axis.text.x = element_text(angle = 50, hjust = 1),
            axis.line.y = element_line(colour = "black"),
        panel.grid.major = element_blank(),
        panel.grid.minor = element_blank(),
        panel.border = element_blank(),
        panel.background = element_blank(),
        panel.grid.major.y = element_line(colour = "grey")) +
  labs(x = "Fare-per-person", y = "Survival rate", title = "Fig 4.2.c: Passenger survival rate for grouped fares per person.") +
  scale_fill_discrete(name  ="Survived")
plot(fig.4.2.c)
```

![plot of chunk unnamed-chunk-15](/figure/posts/Predicting-Survival-on-the-Titanic/unnamed-chunk-15-1.png)

Examining *Fig 4.2.c*, there is a trend indicating the proportion of passengers surviving at each Fare grouping increases as the values in Fares increase. For Fare's above 55 the trend is not as easy to discern but such values are considered outliers for first class ticket holders. Comparing with the variable *Pclass*'s impact on surviving, we observe much more granularity which could be expected to improve our prediction later.

It would also be interesting to know how the price of a Fare relates to a cabin's location on the ship. We can hypothesize that the cheapest tickets would likely provide passengers with a cabin lower in the ship. Subsequently, such passengers may find themselves at the back of the queue for lifeboats once they had made it to deck, lowering their overall chance of survival. Unfortunately, many of the cabins numbers are not present in the *Cabin* variable so we can not test this hypothesis.

## Impact of age on survival

We now consider the survival rate of different age groups. We have removed all passengers that have no age recorded, and grouped passengers into 5 year age intervals, with the first as *0-5 years* and the last *75-80* years.


```r
train_age <- train[(!is.na(train$Age)), ]
train_age$Age_group <- cut(train_age$Age, seq(0, 85, 5), include.lowest=T)
# Plot
fig.4.3.a <- ggplot(train_age, aes(x = Age_group)) +
  geom_bar(stat='count', aes(fill = factor(Survived)), position = 'fill') +
        theme(axis.text.x = element_text(angle = 50, hjust = 1),
            axis.line.y = element_line(colour = "black"),
        panel.grid.major = element_blank(),
        panel.grid.minor = element_blank(),
        panel.border = element_blank(),
        panel.background = element_blank(),
        panel.grid.major.y = element_line(colour = "grey")) +
  labs(x = "Age", y = "Survival rate", title = "Fig 4.3.a: Passenger survival rate for grouped ages.") +
  scale_fill_discrete(name ="Survived")
plot(fig.4.3.a)
```

![plot of chunk unnamed-chunk-16](/figure/posts/Predicting-Survival-on-the-Titanic/unnamed-chunk-16-1.png)

The rate of survival is highest\* among the youngest age grouping of *0-5 years*.
In general, most age groups have similar rates of survival, so it appears that age may not be a great predictor of whether a passenger survived or not

\* Note- there was only a single passenger in the 75-80 year age group. While this passenger survived, overall there are not enough passengers in the higher age ranges to provide data for confident predictions.

## Impact of family relations on survival

There are two variables that consider family relations, *Sibsp* - whether a passenger is a sibling or spouse, and *Parch* - whether parent, child. It would be interesting to consider whether travelling alone or with family had an impact on survival. Lets then use these variables to create a *Family* variable, with labels of *Single* (no family members present) and *Family* (> 0 family members present)


```r
trainInvestigate <- train %>% dplyr::mutate(Family= ifelse(SibSp > 0 | Parch > 0, "Multiple", "Single"))
prop.table(table(trainInvestigate$Family, trainInvestigate$Survived), margin = 1)
```

```
##           
##                 0      1
##   Multiple 0.4944 0.5056
##   Single   0.6965 0.3035
```

We find:

- Single passengers have a lower rate of survival compared with those traveling with family.
- Passengers travelling with family members have 0.51 rate of survival, which is considerably better than the overall rate of survival.

## Impact of departure port on survival

Passengers embarked from three different ports designated as *S* (Southampton), *C* (Cherbourg) and *Q* (Queenstown), and the survival rates for these passengers are;


```r
prop.table(table(train$Embarked, train$Survived), margin = 1)
```

```
##    
##          0      1
##     0.0000 1.0000
##   C 0.4464 0.5536
##   Q 0.6104 0.3896
##   S 0.6630 0.3370
```

We find:

- Passengers embarking from Cherbourg in France had the highest survival rate of 55%
- Passengers embarking from Southampton in England had the lowest survival rate of 34%

## Impact of nationality on survival

A potentially interesting investigation is to consider whether a passengers
nationality has an impact on survival rate. To extract a passengers nationality we shall consider their name and utilise an open source name <-> nationality predictor at *NamePrism*. We use the open API from *http://www.name-prism.com/api* to determine
ethnicity. Details from site;

*NamePrism is a non-commercial nationality/ethnicity classification tool that aims to support academic research, e.g. sociology and demographic studies. In this project, we learn name embeddings for name parts (first/last names) and classify names to 39 leaf nationalities and 6 U.S. ethnicities. Please cite following paper if you used this tool in your work.*

Nationality Classification using Name Embeddings. Junting Ye, Shuchu Han, Yifan Hu, Baris Coskun, Meizhu Liu, Hong Qin and Steven Skiena. CIKM, Singapore, Nov. 2017.

I have submitted the results of this data manipulation to Kaggle as a new data set with the new variable *Nationality*. Steps performed in this data manipulation are however discussed below;

To perform a request at *Name Prism* we must first extract the First name and
Surname from the *Name* variable, and URL encode it. We shall also consider married women's (name title designation identified as *Mrs*) first names even though their husbands names are given precedence.

First, we create a separate data frame *Nationality* with the train$Name
column to allow us to perform our data manipulations in easy to follow steps.


```r
trainNationalitySubset <- train[,c("PassengerId","Name")]
```

Next, employ the *formatName* function to extract first names and surnames
into new column *FullName*. In this step we also modify the output so the
names are in a form that can make a GET request at *NamePrism* API
eg. {base_url}/{firstName}%20{Surname}.


```r
trainNationalitySubset$FullName <- lapply(as.character(trainNationalitySubset$Name), formatName)
```

We then extract the most probable nationality for each passenger. Note- the code snippet below has a long execution time as it is making sequential requests to *NamePrism* API. It is not executed here, but the results are provided in the *trainNationalitySubset* dataframe. Additionally, please note that *NamePrism* API has a daily usage limit of 1000 requests.


```r
# Name preperation
formatName <- function(name) {
  title <- gsub('(.*, )|(\\..*)', '', name)
  nameSplit <- strsplit(name, split = '[,./]')
  surname <- gsub(" ", "%20", nameSplit[[1]][1], fixed = TRUE)
  firstname <- strsplit(nameSplit[[1]][3], split = '[ ]')[[1]][2]
  if(title=='Mrs') {
    maidenFullName <- regmatches(name, gregexpr("(?<=\\().*?(?=\\))", name, perl=T))[[1]]
    if (length(maidenFullName)!=0) {
      firstname <- strsplit(maidenFullName, split = "[ ]")[[1]][1]
    }
  }

  paste(firstname, surname, sep= "%20")
}
# API request and process response
requestLocation <- function(name) {
  url <- paste("http://www.name-prism.com/api/json/", name, sep='')
  response <- fromJSON(url, simplifyDataFrame = FALSE)
  nationalityProbs <- as_data_frame(response)
  nationality <- colnames(nationalityProbs)[apply(nationalityProbs,1,which.max)]
  nationality
}
# Make requests to Name Prism api
trainNationalitySubset$Nationality <- sapply(trainInvestigateNationalitySubset$fullName, requestLocation)
# Exclude redundant columns
trainNationalitySubset = trainNationalitySubset[ , c("PassengerId", "Nationality")]
```

Finally, we import the *trainNationalitySubset* dataframe and merge the Nationality variable with the *train* data set.


```r
# Import
trainNationalitySubset <- as.tibble(fread('trainNationalitySubset.csv'))
# Merge
trainInvestigateNationality <- merge(x = train, y = trainNationalitySubset, by = "PassengerId", all.x=TRUE)
```

Now that we have completed the data preparation lets first consider the passenger nationalities and their survival numbers.


```r
table(trainInvestigateNationality$Nationality, trainInvestigateNationality$Survived)
```

```
##                               
##                                  0   1
##   African,EastAfrican            2   1
##   African,WestAfrican            1   0
##   CelticEnglish                287 206
##   EastAsian,Chinese              1   2
##   EastAsian,Indochina,Vietnam    1   1
##   EastAsian,Malay,Indonesia     12   3
##   EastAsian,Malay,Malaysia       1   0
##   EastAsian,South Korea          1   0
##   European,French               39  40
##   European,German               43  27
##   European,Italian,Italy         4   4
##   European,Italian,Romania       2   0
##   European,Russian               1   0
##   European,SouthSlavs           15   1
##   Greek                          1   1
##   Hispanic,Philippines           2   0
##   Hispanic,Portuguese           10   6
##   Hispanic,Spanish              23  17
##   Muslim,Nubian                 10   2
##   Muslim,Pakistanis,Bangladesh   0   1
##   Muslim,Persian                 2   0
##   Muslim,Turkic,Turkey           1   0
##   Nordic,Finland                14   9
##   Nordic,Scandinavian,Denmark   11   0
##   Nordic,Scandinavian,Norway    13   4
##   Nordic,Scandinavian,Sweden    50  17
##   SouthAsian                     2   0
```

We find:

- The largest Nationality by passenger name is categorised as *CelticEnglish*, which is to be expected as most passengers embarked from ports in England and Ireland
- The next largest Nationalities are *European, French*, *European, German* and *Nordic,Scandinavian,Sweden*.
- In Figures 4.6.a and 4.6.b we immediately see there are some very different survival rates between different nationalities. This engineered variable could prove a useful addition to our model later.


```r
fig.4.6.a = ggplot(trainInvestigateNationality, aes(x = reorder(Nationality,Nationality, function(x)-length(x)))) +
  geom_bar(stat='count', aes(fill = factor(Survived)), position = 'stack') +
  theme(axis.text.x = element_text(angle = 50, hjust = 1),
        axis.line.y = element_line(colour = "black"),
        panel.grid.major = element_blank(),
        panel.grid.minor = element_blank(),
        panel.border = element_blank(),
        panel.background = element_blank(),
        panel.grid.major.y = element_line(colour = "grey")) +
  labs(x = "Nationality", y = "Number of Passengers", title = "Fig 4.6.a: Passenger survival by nationality ordered by largest nationality") +
  scale_fill_discrete(name  ="Survived")

fig.4.6.b = ggplot(trainInvestigateNationality, aes(x = reorder(Nationality,Nationality, function(x)-length(x)))) +
  geom_bar(stat='count', aes(fill = factor(Survived)), position = 'fill') +
  theme(axis.text.x = element_text(angle = 50, hjust = 1),
        axis.line.y = element_line(colour = "black"),
        panel.grid.major = element_blank(),
        panel.grid.minor = element_blank(),
        panel.border = element_blank(),
        panel.background = element_blank()) +
  labs(x = "Nationality", y = "Survival Rate", title = "Fig 4.6.b: Passenger survival rate by Nationality") +
  scale_fill_discrete(name  ="Survived")
plot(fig.4.6.a)
```

![plot of chunk unnamed-chunk-24](/figure/posts/Predicting-Survival-on-the-Titanic/unnamed-chunk-24-1.png)

```r
plot(fig.4.6.b)
```

![plot of chunk unnamed-chunk-24](/figure/posts/Predicting-Survival-on-the-Titanic/unnamed-chunk-24-2.png)

Examining Figure 4.6.a and 4.6.b there are some very different survival rates between nationalities. The largest nationality *CelticEnglish* has a survival rate of 42% which is close to, but slightly higher than the overall survival rate of 38%. The second largest nationality, *European, French* had a survival rate of 51% which is coincidentally similar to the survival rate of those passengers whom embarked at Cherbourg at 55%.  Other nationalities have differing survival rates, eg *European, German* - 39% and *Nordic,Scandinavian,Sweden* - 25%. Clearly, some nationalities found it easier to get on a lifeboat than others.

# Predicting passenger survival

We shall now consider and implement a number of statistical learning techniques to generate a model that predicts whether a passenger survives or not.

As we seek to categorise each passenger into either class 0 (not survived) or class 1 (survived) we might consider methods that seek to classify responses. Methods we might consider in this problem include: logistic regression, k-nearest neighbour, trees, random forests, boosting, and support vector machines.

I shall not consider optimising these techniques with improved feature selection or by tuning fitting parameters, as I simply want to compare each and demonstrate their implementation.

## Clean data and incorporate engineered features

The subsequent steps detail the pre-requisite steps undertaken to prepare the data for implementation of our trial models.

First, combine the *train* and *test* data frames for homogeneous treatment.


```r
all_data <- bind_rows(train, test)
```

Attempt to fill in missing values for passengers *Embarkment* point, *Fare* and *Age*. Set missing embarkment values as *S* as this port has the largest number of passengers joining the Titanic.


```r
sort(table(all_data$Embarked),decreasing=TRUE)[1]
```

```
##   S
## 914
```

```r
all_data$Embarked[c(62, 830)] <- "S"
```

Set the *NA* *Fare* as the median value.


```r
median(all_data$Fare, na.rm = TRUE)
```

```
## [1] 14.45
```

```r
all_data$Fare[1044] <- median(all_data$Fare, na.rm = TRUE)
```

Due to the large number of missing *Age* values we predict them using the other variables and a decision tree model with the *anova* method.


```r
predicted_age <- rpart(Age ~ Pclass + Sex + SibSp + Parch + Fare + Embarked,
                       data = all_data[!is.na(all_data$Age),], method = "anova")
all_data$Age[is.na(all_data$Age)] <- predict(predicted_age, all_data[is.na(all_data$Age),])
```

Ensure, all categorical variables are encoded as factors.


```r
all_data$Name = as.factor(all_data$Name)
all_data$Sex = as.factor(all_data$Sex)
all_data$Ticket = as.factor(all_data$Ticket)
all_data$Cabin = as.factor(all_data$Cabin)
all_data$Embarked = as.factor(all_data$Embarked)
```

We now add in the variables engineered in the previous section on *Nationality*, *FarePerPerson*, and *Family*.


```r
# FarePerPerson
all_data <- all_data %>%
  group_by(Ticket, Fare) %>%
  dplyr::mutate(FarePerPerson = Fare / n())
# Family
all_data <- all_data %>%
  dplyr::mutate(Family= ifelse(SibSp > 0 | Parch > 0, "Multiple", "Single"))
# Nationality
trainNationalitySubset <- as.tibble(fread('trainNationalitySubset.csv'))
testNationalitySubset <- as.tibble(fread('testNationalitySubset.csv'))
all_dataNationalitySubset <- bind_rows(trainNationalitySubset, testNationalitySubset)
all_data <- merge(x = all_data, y = all_dataNationalitySubset, by = "PassengerId", all.x=TRUE)
```

In the next step the *Nationality* variable was *one-hot encoded*. While it usually unnecessary to *one-hot encode* categorical variables in R, as most models do this automatically under the hood, we can run into issues if we employ cross validation methods. If we separate the training data into folds we may end up in a situation where a categorical variable is not present in the training set but is present in the holdout set, which will cause an error when making predictions. In the case of the *Nationality* variable there are several categories that only occur making this a real possibility.


```r
dummy = acm.disjonctif(all_data['Nationality'])
all_data['Nationality'] = NULL
all_data = cbind(all_data, dummy)
all_data <- all_data %>%
  mutate_at(vars(starts_with("Nationality")), funs(as.integer))
names(all_data) <- make.names(names(all_data))
```

Ensure categorical engineered features that are of type *char* are encoded as factors.


```r
all_data$Family = as.factor(all_data$Family)
```

We also exclude features that will not provide benefit to the model.


```r
all_PassengerId <- all_data %>% select(PassengerId)
all_data <- all_data %>% select(-PassengerId, -Name, -Ticket, -Fare, -Cabin)
```

Split the data back into a train set and a test set.


```r
train_PassengerId <- all_PassengerId[1:891,]
test_PassengerId <- all_PassengerId[892:1309,]

train <- all_data[1:891,]
test <- all_data[892:1309,]
glimpse(train)
```

```
## Observations: 891
## Variables: 36
## $ Survived                                 <int> 0, 1, 1, 1, 0, 0...
## $ Pclass                                   <int> 3, 1, 3, 1, 3, 3...
## $ Sex                                      <fctr> male, female, f...
## $ Age                                      <dbl> 22.00, 38.00, 26...
## $ SibSp                                    <int> 1, 1, 0, 1, 0, 0...
## $ Parch                                    <int> 0, 0, 0, 0, 0, 0...
## $ Embarked                                 <fctr> S, C, S, S, S, ...
## $ FarePerPerson                            <dbl> 7.250, 35.642, 7...
## $ Family                                   <fctr> Multiple, Multi...
## $ Nationality.African.EastAfrican          <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.African.WestAfrican          <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.CelticEnglish                <int> 1, 1, 0, 1, 1, 1...
## $ Nationality.EastAsian.Chinese            <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.EastAsian.Indochina.Vietnam  <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.EastAsian.Malay.Indonesia    <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.EastAsian.Malay.Malaysia     <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.EastAsian.South.Korea        <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.European.French              <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.European.German              <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.European.Italian.Italy       <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.European.Italian.Romania     <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.European.Russian             <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.European.SouthSlavs          <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.Greek                        <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.Hispanic.Philippines         <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.Hispanic.Portuguese          <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.Hispanic.Spanish             <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.Muslim.Nubian                <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.Muslim.Pakistanis.Bangladesh <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.Muslim.Persian               <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.Muslim.Turkic.Turkey         <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.Nordic.Finland               <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.Nordic.Scandinavian.Denmark  <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.Nordic.Scandinavian.Norway   <int> 0, 0, 0, 0, 0, 0...
## $ Nationality.Nordic.Scandinavian.Sweden   <int> 0, 0, 1, 0, 0, 0...
## $ Nationality.SouthAsian                   <int> 0, 0, 0, 0, 0, 0...
```

## Model comparison and accuracy

As a number of different methods will be examined it is important that we are able to asses accuracy and make comparisons between each of them. To validate our models and allow comparison we shall employ the cross-validation technique. This involves splitting our *train* data set into a new training set and a hold-out set. The new training set is used to train our model while the holdout set is used for testing. As we have the known predictor variable in our hold-out set we can assess the accuracy of each of our models against common data.

I have chosen to perform a k-fold cross validation with 5 folds. The train data will be randomly separated into 5 subsets with 4 folds designated as *train* subset and the remaining fold as the *holdout* that acts as our *test* subset. The fold designated as the holdout will change in 5 rounds of fitting so we shall end with a *train* data set that also has a set of model predictions.



```r
# Create Folds
folds <- createFolds(train$Survived, k=5, list=TRUE, returnTrain=FALSE)
# Set up results table
results <- data.frame(PassengerId = train_PassengerId, Fold = NA, Survived = train$Survived)
for(i in 1:5){
  foldIndex <- folds[[i]]
  results$Fold[foldIndex] <- i
}
```
## Tree Analysis


```r
results$Tmodel <- NA
for(i in 1:5){
  indexes <- folds[[i]]
  holdoutData <- train[indexes, ]
  trainData <- train[-indexes, ]

  t_model <- rpart(Survived ~ .,
                  method = "class",
                  data = trainData)
  predict <- predict(t_model, holdoutData, type="class")
  results$Tmodel[indexes] <- as.vector(predict)
}
```

We can now asses the accuracy of this model by employing a confusion matrix.


```r
confMat <- table(results$Tmodel, results$Survived)
sum(diag(confMat))/sum(confMat)
```

```
## [1] 0.8204
```

Now lets also get an idea of which variables were useful predictors in the model. We can do this by training the model on the full *train* data set.


```r
# Create model
t_model <- rpart(Survived ~ .,
                  method = "class",
                  data = train)
# Calculate variable importance
importance <- varImp(t_model)
varImportance <- data.frame(Variables = row.names(importance), Score = importance$Overall)
# Create a rank variable based on importance and filter out 0 score variables
rankImportance <- varImportance %>% filter(Score > 0) %>%
  mutate(Rank = paste0('#',dense_rank(desc(Score))))
# Use ggplot2 to visualise the relative importance of variables
fig.5.3.a <- ggplot(rankImportance, aes(x = reorder(Variables, Score),
    y = Score, fill = Score)) +
  geom_bar(stat='identity') +
  theme(axis.line.y = element_line(colour = "black"),
        panel.grid.major = element_blank(),
        panel.grid.minor = element_blank(),
        panel.border = element_blank(),
        panel.background = element_blank(),
        legend.position="none",
        panel.grid.major.x = element_line(colour = "grey")) +
  labs(x = "Variables", y = "Scores", title = "Fig 5.3.a: Tree model variable importance scores") +
  coord_flip()
print(fig.5.3.a)
```

![plot of chunk unnamed-chunk-38](/figure/posts/Predicting-Survival-on-the-Titanic/unnamed-chunk-38-1.png)

```r
# Use rpart plot function to visualise the decision tree
rpart.plot(t_model)
```

![plot of chunk unnamed-chunk-38](/figure/posts/Predicting-Survival-on-the-Titanic/unnamed-chunk-38-2.png)

Examining the importance score of the variables shown in *Fig 5.3.a* we see that *Sex*, *FarePerPerson*, *Pclass* are the highest scored variables in the model indicating that they are strong predictors of a passenger surviving. The *Age*, *SibSp*, *Family* and to a lesser extent *Parch* and *Embarked* have some impact on a passengers survival. We also see in the decision tree plot that the branches occur on the *Sex*, *FarePerPerson*, *Pclass*, *Age*, *Embarkment*, *SibSp* variables. All of our engineered variables feature with non zero scores, though only the *FarePerPerson* variable appears to have a high level of importance in our decision tree.

With this information I could consider removing some of these variables to see if this will improve the model or performing further feature engineering, but my focus here is a comparison between different techniques.

## Logistic Regression


```r
results$LRmodel <- NA
for(i in 1:5){
  indexes <- folds[[i]]
  holdoutData <- train[indexes, ]
  trainData <- train[-indexes, ]

  lr_model <- glm(Survived ~ .,
                  family = binomial(link='logit'),
                  data = trainData)
  predict <- predict(lr_model, holdoutData, type="response")
  results$LRmodel[indexes] <- as.vector(ifelse(predict > 0.5,1,0))
}
```

The accuracy of the model is calculated as;


```r
confMat <- table(results$LRmodel, results$Survived)
sum(diag(confMat))/sum(confMat)
```

```
## [1] 0.7845
```

## Random Forest Analysis

Apply the Random Forest Algorithm


```r
set.seed(1)
results$RFmodel <- NA
for(i in 1:5){
  indexes <- folds[[i]]
  holdoutData <- train[indexes, ]
  trainData <- train[-indexes, ]

  rf_model <- randomForest(Survived ~ .,
                  method = "class",
                  data = trainData)
  predict <- predict(rf_model, holdoutData, type="response")
  results$RFmodel[indexes] <- as.vector(ifelse(predict > 0.5,1,0))
}
```

The accuracy of the model is calculated as;


```r
confMat <- table(results$RFmodel, results$Survived)
sum(diag(confMat))/sum(confMat)
```

```
## [1] 0.8159
```

## Suport vector machine


```r
names(train) <- make.names(names(train))
results$SVMmodel <- NA
for(i in 1:5){
  indexes <- folds[[i]]
  holdoutData <- train[indexes, ]
  trainData <- train[-indexes, ]
  svm_model <- svm(Survived ~ .,
                  data = trainData,
                  cost = 100,
                  gamma = 1)
  predict <- predict(svm_model, holdoutData)
  results$SVMmodel[indexes] <- as.vector(ifelse(predict > 0.5,1,0))
}
```

The accuracy of the model is calculated as;


```r
confMat <- table(results$SVMmodel, results$Survived)
sum(diag(confMat))/sum(confMat)
```

```
## [1] 0.6566
```

## Model Prediction

In our comparison of various techniques in the previous section, we determined that the *Tree* and *RandomForest* models provided the highest prediction accuracy. Lets select the *RandomForest* model and re-train on the entire *train* set and make the predictions on the *test* set. We shall then submit to Kaggle.


```r
rf_model <- randomForest(Survived ~ .,
                  method = "class",
                  data = train)
prediction <- predict(rf_model, test, type="response")
prediction <- as.vector(ifelse(prediction > 0.5,1,0))
```

Create a data frame with two columns: *PassengerId* & *Survived.* so that *Survived* contains your predictions.


```r
solution <- data.frame(PassengerId = test_PassengerId, Survived = prediction)
```

Write your solution away to a csv file with the name solution.csv


```r
write.csv(solution, file = "solution.csv", row.names = FALSE)
```

# Conclusion

After submitting the results for the *Random Forest model* to Kaggle we obtained a score of 0.77. There are several approaches we could consider to improve our model;

- Employ ensemble learning and utilise the predictions from our different models. I shall discuss this in a subsequent blog.
- Remove features with little predictive value or remove those variable pairings that have a high degree of multicolliniarity. Such approaches must however be considered on a per technique basis.
- Try different methods such K-nearest neighbour, XGBoost, LGBoost to name a few.
- Tune the fitting parameters. While I did do some minor tuning to ensure models were not hopelessly poor, much more could be done.
