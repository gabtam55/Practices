# Practices
A place for my R learnings using open data and participating in the <a href="https://github.com/Hong-Kong-Districts-Info">Hong Kong Districts Info Project</a>.

## 30th November 2020 - T-test and ANOVA (Using {infer})
In 2017, Beryl et al. published the data that they collected for their 'Mood sampling on smartphones' project. I wanted to ask a silly question by looking at their participants' responses on the Smartphone Use Questionnaire and the Big Five Inventory.

### Independent sample t-test
So here was my question:
>Does people's levels of extroversion depend on the hand (e.g. one hand, two hands) they use their smartphones with?

My hypotheses were as follow:
* Null hypothesis: Individuals' extroversion is independent on the hand they use their smartphones with
* Alternative hypothesis: Individuals' extroversion is dependent on the hand they use their smartphones with

In order to answer this question, an independent sample t-test was conducted, looking at the difference in mean extraversion scores between people who use their smartphones with one hand and those with two hands. A computational approach was adopted using {infer}.

First, I calculated the observed t-statistic, which is -0.388.
```
obs_t <- dsuq %>%
  specify(Extraversion ~ X18_2groups) %>%
  calculate(stat = "t", order = c("one hand", "two hands")) %>%
  pull()
```

I then generated the null t-distribution for when the null hypothesis is true.
```
null_distribution_t <- dsuq %>%
  specify(Extraversion ~ X18_2groups) %>%
  hypothesise(null = "independence") %>%
  generate(reps = 5000, type = "permute") %>%
  calculate(stat = "t", order = c("one hand", "two hands"))
```

Finally, I calculated the p-value and visualised it under the null t-distribution. The area in red represents the p-value, whereas the red line is the observed t-statistic. 
```
null_distribution_t %>%
  get_p_value(obs_stat = obs_t, direction = "both")
```
<img src="https://github.com/gabtam55/Practices/blob/master/301120%20-%20Hypothesis%20testing%20(T-test%20and%20ANOVA)/null_distribution_t2.png" alt="Null t-distribution" height="350" />

As the p-value was smaller than 0.05, I failed to reject the null hypothesis. Afterall, the hand people use their smartphones with didn't seem to have an impact on their levels of extraversion (or the other way round).

### One-way ANOVA
I didn't want to give up just yet. Therefore, I tweaked my question to the following while keeping my hypotheses the same.
>Does people's levels of extroversion depend on the hand (e.g. left hand, right hand, two hands) they use their smartphones with?

This time, an one-way ANOVA was conducted to answer my question, looking at the difference in mean extraversion scores between people who use their smartphones with left hand, right hand and two hands.
```
tidy(aov(Extraversion ~ X18..I.usually.use.my.smartphone.with, data = dsuq)) # p-value is 0.857
```

A p-value of 0.857 suggested that there wasn't much point to conduct a post-hoc analysis but I did it anyway. First, I used the Bonferroni correction to find out the appropriate significance level. This was to avoid inflating the type 1 error rate when running multiple two sample t-tests. The modified significance level was 0.017.
```
K <- length(unique(dsuq$X18..I.usually.use.my.smartphone.with)) *
  (length(unique(dsuq$X18..I.usually.use.my.smartphone.with)) - 1) /
  2

bonferroni_corrected_sig_lv <- 0.05 / K # 0.017
```

I then ran a pairwise comparison to look at the results of two sample t-tests for differences in each possible pair of groups. None of the p-values was smaller than 0.017.
```
pairwise.t.test(dsuq$Extraversion, dsuq$X18..I.usually.use.my.smartphone.with, p.adjust.method = "none")
```

As I failed to reject the null hypothesis. The one-way ANOVA also suggested that people's preferred hand to use their smartphones with has nothing to do with their levels of extraversion.

### References
Noë, B., Turner, L., Linden, D., Allen, S., Maio, G., Whitaker, R. (2017). Mood sampling on smartphones. [data collection]. UK Data Service. SN: 852732, http://doi.org/10.5255/UKDA-SN-852732


## 18th November 2020 - Chi-squared Test (Using {infer})
>The General Social Survey (GSS) is a sociological survey created and regularly collected since 1972 by the National Opinion Research Center at the University of Chicago. It is funded by the National Science Foundation. The GSS collects information and keeps a historical record of the concerns, experiences, attitudes, and practices of residents of the United States. (<a href="https://en.wikipedia.org/wiki/General_Social_Survey">Wikipedia, 2020</a>)

Using the results from the GSS conducted since 2000, I wanted to look at the relationship between socioeconomic class and political party affiliation.

First of all, I explored the data with a bar plot. The biggest supporter for the democratic party was the working class, whereas the biggest supporter for the republican party was the middle class. It seemed to suggest that a relationship existed between socioeconomic class and political party affiliation.
<img src="https://github.com/gabtam55/Practices/blob/master/181120%20-%20Hypothesis%20Testing%20(Chi-squared%20Test)/Socioeconomic%20class%20by%20political%20party%20affilliation.png?raw=true" alt="Socioeconomic class by political party affilitation" height="350" />

To find out whether this relationship happened by chance, I performed a chi-squared test using the computational approach (see extract of code below), with the following hypotheses:
* Null hypothesis: The socioeconomic class of US residents is independent of their political party affiliations.
* Alternative hypothesis: The socioeconomic class of US residents is dependent on their political party affiliations.

<img src="https://github.com/gabtam55/Practices/blob/master/181120%20-%20Hypothesis%20Testing%20(Chi-squared%20Test)/code.png?raw=true" alt="Code" height="350" />

The results indicated a p-value of 0.016. The density plot below shows the proportion of chi-squares from the null distribution that are larger than the observed chi-square (where the red line is).
<img src="https://github.com/gabtam55/Practices/blob/master/181120%20-%20Hypothesis%20Testing%20(Chi-squared%20Test)/Null%20distribution%20of%20chi-squared%20distances.png?raw=true" alt="null distribution" height="350" />

As p-value was smaller than 0.05, I rejected the null hypothesis and was in favor of the alternative hypothesis that socioeconomic class of US residents is dependent on their political party affiliations.

## 28th October 2020 - Hypothesis Testing On A Single Proportion (Using {infer})
<img src="https://github.com/gabtam55/Practices/blob/master/281020%20-%20Hypothesis%20Testing%20(Single%20Proportion)/code.png?raw=true" alt="HypoTestOneProp" height="500" />

## 2nd October 2020 - Job Description Generator (shiny interactive form)
The idea of this app is to speed up the job description creation process by providing the user with standard job information pulled from the O*NET to start with. The job information displayed on the right hand side of the app are editable, which allows users to customise them accordingly. An export feature is yet to be built to allow the edited job information to be downloaded.
<br/>
<br/>
<a href="https://hfigabriel.shinyapps.io/job_profile/?_ga=2.71492179.1301702451.1601633849-849884923.1596283104">Link</a> to shiny app
<br/>
<br/>
<img src="https://github.com/gabtam55/Practices/blob/master/021020%20-%20Job%20Description%20Generator/Job%20Description%20Generator.png?raw=true" alt="JDGeneratorpng" width="500" />
<p style="text-align: center"><a href="https://services.onetcenter.org/" title="This shiny app incorporates information from O*NET Web Services. Click to learn more."></a></p>
<p>This shiny app incorporates information from <a href="https://services.onetcenter.org/">O*NET Web Services</a> by the U.S. Department of Labor, Employment and Training Administration (USDOL/ETA). O*NET&reg; is a trademark of USDOL/ETA.</p>


## 30th August 2020 - Hong Kong Traffic Collision Data - Wireframe For Later Use (shinydashboard wireframe)
<img src="https://raw.githubusercontent.com/gabtam55/Practices/master/300820%20-%20Hong%20Kong%20Collision%20Data%20Wireframe/Shinydashboard%20Wireframe%20300820.png" alt="HKColShinydashboardpng" width="500" />


## 21st August 2020 - Mental Issue and Socioeconomic Status (shiny interactive plot & table)
<a href="https://gabtam55.shinyapps.io/unemployment_and_mental_illness_survey_190820/?_ga=2.116531782.1301702451.1601633849-849884923.1596283104">Link</a> to shiny app
<br/>
<br/>
<img src="https://raw.githubusercontent.com/gabtam55/Practices/master/210820%20-%20Mental%20Health%20%26%20Socioeconomic%20Status/Shiny%20App%20210820.png" alt="MHShinypng" width="500" />


## 17th August 2020 - Hong Kong Traffic Collision Incident Map (shiny interactive map)
<img src="https://raw.githubusercontent.com/gabtam55/Practices/master/170820%20-%20Hong%20Kong%20Collision%20Data%20Map/Shiny%20Map%20170820.png" alt="HKColShinypng" width="500" />
