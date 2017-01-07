
## DAND Project 2 Investigate the Dataset of Titanic

### Liang Sun        
### January 5, 2017

*****

### Outline
- Introduction to the dataset
- Statement of questions
- Description of investigation steps
- Data wrangling
- Results
- Codes for data analysis

******

### 1. Introduction to the data set

This project investigates a data set on the information of 891 passengers on a British liner named "Titanic" which sank in Atlantic Ocean on her maiden voyage from England to New York in 1912. There were 2,224 passengers and crew aboard, so the data set used here is an incomplete record of Titanic passengers.The data set  contains a variety of variables concerning passenger's age,gender, family, ticket, and survival infromation. The variables are listed as below:



| Variable Name |  Variable Description |
|---------------|-----------------------|
|PassengerID| Passenger ID|
|   Survived  |  Survival (0 = No;1 = Yes)|
|Pclass|Passenger Class (1 = 1st; 2 = 2nd; 3 = 3rd)|
|Name|Name|
|Sex|Sex|
|Age|Age|
|SibSp| Number of Siblings/Spouses Aboard|
|Parch|Number of Parents/Children Aboard|
|Ticket|Ticket Number|
|Fare|Passenger Fare|
|Cabin|Cabin|
|Embarked|Port of Embarkation (C = Cherbourg; Q = Queenstown; S = Southampton)|


****



### 2. Statement of questions

According to historical records, passengers were evacuated in lifeboats after Titanic started to sink. The only twenty lifeboats on the liner were supposed to accomodate 1,178 people but saved only about 600 lives. Anyway, not all passengers could have the chance to get in the lifeboats. It was reported that women and children were given priority to evacuate first. In addition, people in higher-class cabins had better access to the deck where lifeboats were equipped than people who stayed in third-class cabins which were on the lowest level in the ship. To sum up, the chance of survival may have been affected by a variety of factors, including gender, age, class and so on. Therefore, one of the questions to be answered by exploring this data set is what the relationship between these factors and one's chance of survival is. How did factors interact and co-determine the result.

What made it more tragical is that many passengers went on the trip with their family. As passengers were not evacuated in family unit, many of them would have had to choose between surviving or dying with family, which was a toughest and heartbreaking situation that no one could imagine. Survivors who went on the trip with their family may have been separated forever. For those who finally survived, how many of them have lost their family members?  

Given these thoughts, I will investigate the data set and answer the following questions:

- How do the summary statistics of passengers, including their age, gender, and class of ticket,vary by survival?
- What is the relationship between passenger's age and their chance of survival?
- What is the relationship between passenger's gender and their chance of survival?
- What is the relationship between passenger's ticket fare and their chance of survival?
- What is the relationship between passenger's class of cabin and their chance of survival?
- What is the relationship between port of embarkation and chance of survival?
- How did different factors interact in determining the chance of survival?
- How many survivors lost family members?


*****

### 3. Description of investigation steps

To investigate the questions stated above, I started with browsing the data structure and knowing what variables are included in the data, particularly the meaning and type of each variable and convert variables into format that is fit for data analysis.

Second, I explored the descriptive statistics of variables, mainly mean and standard deviation of each variable. I also checked how the descriptive statistics vary between survivors and non-survivors. The variables that showed evident difference between two groups are likely to have had an impact on chance of survival.

Third, I examined the relationship between possibly related variables by checking descriptive statistics and performing t-test or chi-squared test. I also visualized the relationship between variables by drawing box plot, bar chart and histogram plot.  

In the whole process of investigation, I referred to online resources about Titanic historical records for a better understanding of how people survived or died in the disaster, so as to explain or support my findings from the data analysis. I also acquired solutions to tenical problems I came across in the coding process from websites of data and programming.

*******

### 4. Data wrangling

First, import packages that will be used in the data analysis:

 ```
 import matplotlib.pyplot as plt
 import numpy as np
 import pandas as pd

 %pylab inline  

 from scipy.stats import ttest_ind

 ```

Second, read the data file from csv file and save as a Pandas dataframe:
```
filename = 'C:/Users/Liang Sun/Documents/My NanoDegree/dandp2_project/titanic-data.csv'
titanic_df = pd.read_csv(filename)
```

Third, take a brief look at the data structure and variables,

  ```
  titanic_df.head()
  ```

   and notice that variable "Sex","Pclass", and "Embarked" are categorical.

Fourth, convert categorical into dummies:

   ```
   # "sex" is coded as 1=female and 0=male

    def convert_sex(sex):
        if sex == 'female':
            sex = 1
        else:
            sex = 0
        return sex
    titan_df = titanic_df.copy()
    titan_df['Sex'] = titan_df['Sex'].apply(convert_sex)
    titan_df.head()
   ```

   ```
   # "Pclass","Embarked" are categorical, so we get their dummies for analysis

    class_dummies = pd.get_dummies(titan_df['Pclass'],prefix='class')
    embark_dummies = pd.get_dummies(titan_df['Embarked'],prefix='port')

   # and concate the data with columns of dummies

    titan_df = pd.concat([titan_df,class_dummies,embark_dummies],axis=1)
   ```

Finally, check the descriptive statistics of all variables, and notice any variables that have missing values,

   ```
    titan_df.describe()

    titan_df['Embarked'].count()
   ```

Variables "Age" and "Embarked" have missing values. The were 177 missing values for "Age" and 2 missing values for "Embarked".  

If we delete the missing values of these two variables listwise, we will lose 179 observations with non-missing values for other variables that will be examined in the analysis, although we can get a more consistent sample size throughout the whole analysis.

However, I do not want to lose almost 20% of the observations. Since we mainly look at relationship between every two variables instead of performing a regression analysis that involve all variables at the same time, I will handle missing values case by case.

When checking the relationship between "Age" and "Survived", I dropped observations with missing "Age":

  ```
  age_survival = titan_df[['Survived','Age']] # 891 observations
  age_surv = age_survival.dropna()
  #Drop missing values of age, and 714 observations are kept
  ```

When checking the relationship between "Embarked" and "Survived", I dropped observations with missing "Embarked":

  ```
  port_survival = titan_df[['Survived','Embarked']]  
  port_surv=port_survival.dropna()  
  ```
Therefore, the missing values of "Age" or "Embarked" do not affect the analysis of other variables, such as "Sex", "Class", "Fare" and "Survived" which have no missing values.  


The dataframe ```titan_df``` is used as the main data set in the following data analysis.

********

### 5. Results
The results are presented in response to each of the questions stated above.

#### Question 1: Summary statistics


Table 1. Mean and standard deviation of main variables of full sample and subsamples by survival


|Variable | All Passengers | Not Survived | Survived|
|:--------:|:-----:|:--------:|:------------:|
|Female| 0.35 | 0.18 | 0.68 |
| |(0.48)|(0.35)|(0.47)|
|Age|29.70|30.63|28.34|
| |(14.53)|(14.17)|(14.95)|
|Fare|32.20|22.12|48.40|
| |(49.69)|(31.39)|(66.60)|
|1st Class|0.24|0.15|0.40|
| |(0.43)|(0.35)|(0.49)|
|2nd Class|0.21|0.18|0.25|
| |(0.41)|(0.38)|(0.44)|
|3rd Class|0.55|0.68|0.35|
| |(0.50)|(0.47)|(0.48)|
|Port Cherbourg|0.19|0.14|0.27|
| |(0.39)|(0.34)|(0.45)|
|Port Queenstown|0.09|0.09|0.09|
| |(0.28)|(0.28)|(0.28)|
|Port Southampton|0.72|0.78|0.63|
| |(0.45)|(0.42)|(0.48)|
|Number of Siblings/Spouse Aboard| 0.52|0.55|0.47|
| |(1.10)|(1.29)|(0.71)|
|Numer of Parents/Children Aboard|0.38|0.33|0.46|
| |(0.81)|(0.82)|(0.77)|
|*N_Age*|714|424|290|
|*N_Port*|889|549|340|
|*N*|891|549|342|

`Note: "Age" and "Port of Embarked" have missing values.`

From the summary statistics above, we can see that the total number of passengers in this data set is 891, among which 549 (61.62%) died in the disaster and 342(38.38%) survived.


A variety of factors may have determined who could survive from the disaster. It seems that survivors and non-survivors had difference in most of their characteristics, given the mean values in their gender, age, class and so on. Thus, I am going to perform further detailed analysis on the relationship between each factor and survival.

#### Question 2:  Relationship between gender and survival

First, a contingency table is created between gender and survival:



|Sex|Survived|Not Survived|*Total*|
|:---:|:---:|:---:|:---:|
|Male|109|468|577|
|Female|233|81|314|
|*Total*|342|549|891|


Second, a chi-squared test is conducted to test the relationship between these two variables. chi^2-stat=260.717 and p-value = 0.000 suggest that "Sex" and "Survived" are dependent.

Third, a bar chart is drawn to visualize the evident difference by gender in survival:

<center>
![Survival by gender](/image/survival_by_gender.png)
</center>

If we change our perspective to view the relationship between gender and survival, we can get a bar chart like this:

<center>
![Gender by survival](/image/gender.png)
</center>

Both graphs reveal the same thing, that is, female passengers were given priority in evacuation and had higher survival rate.

#### Question 3: Relationship between age and survival

To begin with, a t-test is performed to test the difference of age mean between survivor (28.344) and non-survivors (30.626).

The result (t-stat=2.067 and pvalue=0.039) suggests that the difference is significant, which means that survivors' average age is significantly smaller than non-survivors.

A violin plot is created to show the difference in the distribution of age of survivors and non-survivors:

<center>

![violin](/image/violin_age.png)

</center>

We can see that in the group of survivors, there are more passengers with lower age. It seems that younger passengers were likely to survive. Therefore, I am interested in comparing the survival rate between children (<=16 years old) and adults.

The contingency table between age group and "Survived" is:


|Age Group|Survived|Not Survived|*Total*|
|:---:|:---:|:---:|:---:|
|Children|55|45|100|
|Adult|235|379|614|
|*Total*|290|424|714|

The chi-squared test here suggests an association between the survival and age group ( chi^2-stat=9.293, p-value=0.003).



Moreover, a bar chart was depicted to display such difference:

<center>

![Survival by age group](/image/survival_by_kid.png)

</center>

The higher proportion of survival among children than that among adults proves that children were also given priority in evacuation.

#### Questions 4: Relationship between fare and survival

A t-test is conducted to compare the difference in the mean of fare between non-survivors (mean=22.12) and survivors (mean=48.40). A significant result (t-stat=-7.939, p-value=0.000) suggests that survivors generally paid more than non-survivors for their tickets.

A histogram is created to show the difference of fare between two groups:

<center>
![fare by survival](/image/hist_fare.png)
</center>

To understand how ticket fare is correlated with chance of survival, we need to further explore the factor--class of cabin, which was determined by the ticket price and probably had a more direct and crucial impact on one's access to the deck and lifeboat, given the fact that lifeboats were installed on the deck of the ship while cabins were ranked by order from upper to lower levels of the ship.

#### Question 5: Relationship between class of cabin and survival

First, I checked the contingency table between survival and class of cabin:

|Class of cabin|Survived|Not Survived|*Total*|
|:---:|:---:|:---:|:---:|
|1|136|80|216|
|2|87|97|184|
|3|119|372|491|
|*Total*|342|549|891|


A chi-squared test (chi^2-stat=102.889,p-value=0.000) suggests that class of cabin and survival are strongly associated.

A bar chart was drawn to show the difference between classes:
<center>
![survival by class](/image/survival_by_class.png)
</center>
The death rate for the third class is so much higher than the first and second!

#### Question 6: Relationship between port of embarkation and survival
Port of embarkation does not seem to have a direct or causal relationship with survival, but I am curious about the role of port of embarkation and how it might be related to survival or determinants of survival.

Similarly, I got the contingency table between port of embarkation and survival:


|Port|Survived|Not Survived|*Total*|
|:---:|:---:|:---:|:---:|
|Cherbourg|93|75|168|
|Queenstown|30|47|77|
|Southampton|217|427|644|
|*Total*|340|549|889|


Given a chi-squared test (chi^2-stat=26.489,p-value=0.000), these two variables seem to be associated.

In addition, a bar chart shows the difference:
<center>
![survival by port](/image/survival_by_port.png)
</center>
It seems that passengers from Cherbourg had higher chance of survival. How to explain this? Based on previous analysis, it is probably because port is correlated to other factors, such as gender, age, and class, which in fact determine their chance of survivial. So let's check how gender, age, and class vary by port.



| Port|Survival|Sex|Age|Class|Fare|
|:-----:|:---------:|:-------:|:---------:|:-------:|:-------:|
|Cherbourg|0.554|0.435|30.815|**1.887**|**59.954**|
|Queenstown|0.390|0.468|28.089|2.909|13.276
|Southampton|0.337|0.315|29.445|2.351|27.080


The evident difference between mean values of class and ticket fare here suggest passengers from Cherbourg were more likely to have bought first or second-class tickets than passengers from Queenstown and Southampton. I further plotted the number of passengers in each class by port:
<center>
![class by port](/image/class_by_port.png)
</center>
One may wonder why passengers from Cherbourg preferred first-class cabins. Although we do not have detailed background information of passengers who embarked from Cherbourg in the data, we know that Cherbourg is a port city in northwestern France and we can assume that a large proportion of the passengers who embarked from France might have a different reason for the trip than working-class immigrants. By checking some of the passengers' names online, I found there were tourists, opera singer, publisher, jewller, banker and people of similar socioeconomic status, among those who embarked from Cherbourg.

Therefore, port of embarkation itself has no direct correlation with chance of survival, but passengers' class determines their chance of survival to a great extent.

#### Question 7: How did different factors interact in determining survival?

Since passengers in first-class cabins had better access to lifeboats and higher survival rate while women and children in general were given priority to evacuate, we may think, did women and children in lower-class cabins have same chance to get in lifeboats as those in first-class cabins?

- Interaction between gender and class

First, I created a subset of data with female observations only. The contingency table between class and survival is:


|Class of cabin|Survived|Not Survived|*Total*|
|:---:|:---:|:---:|:---:|
|1|91|3|94|
|2|70|6|76|
|3|72|72|144|
|*Total*|233|81|314|


A chi-squared test (chi^2-stat=81.887,p-value=0.000) reveals a strong assication between these two variables.


A bar chart is drawn to show the difference:
<center>
![Female survival by class](/image/female_survival_by_class.png)
</center>
The statistics and graph above suggest that even though women in general were given priority to evacuate in lifeboat, they did not have equal chance to do so depending on which class of cabins they were in. Women in first and second-class cabins had much greater chance of survival than those in third-class cabins.

- Interaction between age and class

Children were also given priority to leave the ship, but did class also make the chance of survival unequal for children in different classes?

First, I created a subset of data with children only (<=16 years old), and the contingency table between class and survival is:


|Class of cabin|Survived|Not Survived|*Total*|
|:---:|:---:|:---:|:---:|
|1|8|1|9|
|2|19|2|21|
|3|28|42|70|
|*Total*|55|45|100|

A chi-squared test (chi^2-stat=21.219,p-value=0.000) reveals a strong assication between these two variables.

A bar chart is drawn to show the difference:
<center>
![Kid survival by class](/image/kid_survival_by_class.png)
</center>
The statistics and graph above suggest that children in different cabins did not have equal chance to survive, similar to what we found about women.

In conclusion, although women and children had priority to evacuate first in general, their class largely intervened how much is their chance to survive.

#### Question 8: Family loss of passengers who survived

Every passenger had a ticket number and family were likely to share an identical ticket number, so people with same ticket number and with family members on board can be generally identified as family.

Among 342 survivors, 53 (15.50% of the total) lost family members in the disaster. More than half of them lost at least two family members.
<center>
![Family loss](/image/hist_fam_loss.png)
</center>
The data only recorded family members including parents and children, spouses and siblings as two combined counts, which may have omitted other family members. I also notice that many peole shared the same ticket number but had no family relationship recorded. Such people might be relatives, friends, colleagues, companions and so on. Therefore, many survivors since then have lived in severe pain of losing people they loved or they knew, or suffered from the experience of having seen other passengers die in the disaster, as historical records depict.

#### Conclusion

The chance of survival from the Titanic disaster was determined by a variety of factors, mainly including gender, age, and class of cabin. These factors reveal two sides of the story, humanity and disparity. Because of humanity, women and children had better chance of survival than men, regardless of their background. Meanwhile, due to different socioeconomic status, more than half of the passengers in our data could only afford third-class cabins, which lethally reduced their chance of survival. However, humanity shines more because it shows how brave people made selfless decisions in the face of disaster. It is really heart-touching that many gentlemen gave away their chance of get in lifeboats and insisted on "ladies and children first."

It is hard to say which factor plays a more critical role from the data analysis here, but we can be certain that chance of survival is the result of a combination of various factors, and we can have better understanding of the pattern if more detailed information is available.

With regard to the information available here, there are a few limitations of the data. If we are able to overcome these limitations, we will probably obtain more meaningful findings out of the data.

First, the sample collected here may be biased. The data used here included 891 passengers, which is about 67.65% of all passengers (1317) and 40.06% of all passengers and crew aboard (2,224). We are not sure if the sample has a larger proportion of survivors among passengers (342/891 = 38.38%) than that of the population (724/2224 = 32.25%), unless we can figure out how many passengers or crew members survived among the population. It might have been easier to get information about survivors or people who died in the disaster but are known by survivors than information of the unknowns, so the information we have here may be not representative of the population.  If we have a complete or a larger sample including both passengers and crew members, we can have a more accurate sense of the survival rate of passengers and crew members on Titanic. With crew members information, we can also examine questions such as whether female crew members were also given priority in the evacuation.

Second,the data provide unclear information about family members. We cannot differentiate between spouse and siblings, or between parents and children,as they were reported in combined counts. If we know more specific information about family members, we can perform more detailed analysis of the relationship between survivors and non-survivors. For example, since women and children had better chance to survive, we may wonder how many of them lost their spouse or parents. Did women with children have better chance to survive than women without children, or how about men with children? Did men with wife have better chance to survive? How did such chance vary by class of cabin?

Third, the variable of cabin in the data was not usable for the analysis,as it was recorded in text. However, cabin can be an important indicator of survival if we know which level the cabins were located in. There were nine levels in Titanic, among which Deck A to F were for lounge rooms, eating areas and passenger quarters, and the others were for maintenance and storage. It seems that cabin can more accurately measure the distance from passenger to the deck than class of cabin does, because class of cabin has only three general categories, but within each class, cabins might be on different levels.

In addition to the considerations above, here is one more idea about future research on the data.

In theory, more people could have been saved if the lifeboats on Titanic were well and fully utilized. If we know which lifeboat the passenger was in (which is actually avaiable online) and the sequence of lifeboats leaving Titanic, we can perform a time analysis of the evacuation. For example, we may see the number of passengers in each lifeboat changed over time, or change in the gender,age,class composition of passengers in lifeboats over time. We may get a good idea of human's pyschology in crisis and draw lessons from the failure of crisis management in this disaster.    

### 6. Codes for data analysis

Please check the "DAND_Project2_Titanic_code_for_analysis.ipynb" file uploaded to the repository.

### References

* Add a new column to dataframe http://stackoverflow.com/questions/16327055/how-to-add-an-empty-column-to-a-dataframe
* Bar chart https://pythonspot.com/matplotlib-bar-chart/
* Change values of a column http://stackoverflow.com/questions/12604909/pandas-how-to-change-all-the-values-of-a-column
* Cherbourg https://en.wikipedia.org/wiki/Cherbourg-Octeville
* Chi-squared test in Python https://docs.scipy.org/doc/scipy-0.15.1/reference/generated/scipy.stats.chi2_contingency.html
* Concate dataframes http://pandas.pydata.org/pandas-docs/stable/merging.html
* Drop missing values http://pandas.pydata.org/pandas-docs/version/0.17.0/generated/pandas.DataFrame.dropna.html
* Get dummies http://pandas.pydata.org/pandas-docs/stable/generated/pandas.get_dummies.html
* How many floors did the Titanic have https://www.reference.com/education/many-floors-did-titanic-1ee686f0a1ffe96d
* Merge dataframes http://pandas.pydata.org/pandas-docs/stable/merging.html
* Move and resize legender http://stackoverflow.com/questions/23238041/move-and-resize-legends-box-in-matplotlib
* Name ticks in boxplot http://stats.stackexchange.com/questions/3476/how-to-name-the-ticks-in-a-python-matplotlib-boxplot
* Plot multiple boxplots in one graph http://stackoverflow.com/questions/27061137/plot-multiple-boxplot-in-one-graph-in-pandas-or-matplotlib
* Seaborn pair plot http://seaborn.pydata.org/generated/seaborn.pairplot.html
* Seaborn violin plot http://seaborn.pydata.org/generated/seaborn.violinplot.html
* Sort dataframe by column http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.sort.html
* Titanic https://en.wikipedia.org/wiki/RMS_Titanic#Survivors_and_victims
* Titanic Documentary Video https://www.youtube.com/watch?v=SVLt9UfO3Q0
* Titanic facts https://www.encyclopedia-titanica.org/
* Titanic survivors https://www.encyclopedia-titanica.org/titanic-survivors/
* T-test in Pandas http://stackoverflow.com/questions/13404468/t-test-in-pandas-python
* Working with missing values http://pandas.pydata.org/pandas-docs/stable/missing_data.html
