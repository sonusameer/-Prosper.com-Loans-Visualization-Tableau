Exploratory Analysis of Prosper.com Loans
## Exploring the Dataset


The dimensions of the dataset are 113937, 82

### Problems with the structure of some variables.

1. ListingCategory and the 2 ProsperRatings had some problematic characters that were converted to dots.
2. Converted LoanOriginationDate and ListingCreationDate to Date class
2. Credit Score is represented by two values providing a range.  I'd like to have a single value, so I'll create a new value that is the average of the upper and lower ranges.

## Univariate Plots

![](figure/unnamed-chunk-2-1.png) 

The majority of loans do not use one of the suggested categories, since the first bar of this chart is "Not Applicable" and the last bar is NA and together they account for more than half the rows of data.

![](figure/unnamed-chunk-3-1.png) 

Credit scores range from 300 to 850.  Prosper borrower have a median score of NA, which is considered good credit.  Prosper now requires a minimum credit score of 640 for new borrowers or 600 for returning borrowers, but initially, subprime borrowers could also apply for loans.

![](figure/unnamed-chunk-4-1.png) 

The plot of income range looks very symmetric until I notice that the categories are not in the correct order.

![](figure/unnamed-chunk-5-1.png) 

Few loans are made to borrowers with incomes below $25,000.  There should be a
similar distribution seen in the borrowers' monthly income. But the max value is 1.75 million. The maximum loan amount is only $35,000 with a 3-year term, which would not be worth the time spent applying for someone making millions a month. Actually, this loan was for $4000.

In fact, there are 530 loans with stated monthly income greater than $25,000. Plotting the amount of their loans shows that they tend to request fairly small amounts.  It seems likely to me that some people entered their annual income in place of their monthly income (though that doesn't explain the $1750000 entry). 

There is also a variable for IncomeRange that uses annual income.  If these columns are independent (ie one from the credit report and one from the borrower's application), then IncomeRange should roughly equal StatedMonthlyIncome for these rows.  If IncomeRange is calculated from the borrower's StatedMonthlyIncome, then all of these rows would be in the $100,000+ category, which is the case in the following plot.

![](figure/unnamed-chunk-6-1.png) 

Since I don't believe wealthy people would be borrowing such comparatively small amounts, this means we can't cross-check the borrower's income this way and we have to be certain to only include one of these variables in any model later on.

![](figure/unnamed-chunk-7-1.png) 

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    1000    4000    6500    8337   12000   35000
```

Half of all Prosper loans are for $1000 - $6500. The most common amount requested appears to be $4,000.

### Timeseries
Flipping through the [Prosper 2013 annual report](https://www.prosper.com/Downloads/Legal/prosper10k12312013.pdf), I found an Excel chart of loan originations (in dollars) by quarter on page 74 and wondered if I could recreate it in ggplot and then modify it to be more interesting.

![](figure/Bivariate_Plots-1.png) 

The chart in the annual report began in the third quarter of 2009.  The period  of October 15, 2008 to July 13, 2009 is known as Prosper's [Quiet Period](http://www.lendacademy.com/a-look-back-at-the-lending-club-and-prosper-quiet-periods/) when they were required to suspend lending pending SEC approval.  When they relaunched in July 2009, there were several changes to their lending process, so I'll have to keep that in mind.

![](figure/unnamed-chunk-8-1.png) 


```
##              Cancelled             Chargedoff              Completed 
##                      5                  11992                  38074 
##                Current              Defaulted FinalPaymentInProgress 
##                  56576                   5018                    205 
##   Past Due (>120 days)   Past Due (1-15 days)  Past Due (16-30 days) 
##                     16                    806                    265 
##  Past Due (31-60 days)  Past Due (61-90 days) Past Due (91-120 days) 
##                    363                    313                    304
```

![](figure/unnamed-chunk-9-1.png) 

Except for the colors (should do a nice red, yellow, money-green) this is a nice chart.  Default rates were high initially, but improved with the new standards implemented after the 'quiet period'.  Default rates drop in recent quarters because those loans have had less time to enter default.  Proper models defaults with curves and notes that those recent loans have default rates below expectations.

Prosper divides loan requests into ProsperRating groups according to expected risk.  The higher risk the borrower is projected to be, the higher the interest rate set on the loan. OK, so to start, I want to look at the Results of the loans by ProsperRatingCategory, but since I'm comparing two factor variables, I think it makes sense to make another stacked barplot and leave it in the univariate section (though I would have considered the timeseries to be bivariate, but Wikipedia disagrees).

![](figure/unnamed-chunk-10-1.png) 

D has more defaults than E, but also more loans. E and HR (for High Risk) have about the same number of defaults, but E's rate is lower.  Variable definitions note that the ProsperRating was introduced in 2009, so that blank column must represent all of the pre-2009 loans, when they used something called the CreditGrade to rank borrowers.  I'm surprised the company survived the quiet period when they couldn't make new loans and over a third of their prior loans went into default.  I'd like to see how well the default rate matches with Prosper's expectations, but then I'd need Time, Results, and ProsperRating, which is definitely not univariate.

## Univariate Analysis

#### What is the structure of your dataset?
The ProsperLoan dataset contains 81 variables about 113937 loans made through the prosper.com marketplace.  The loans cover the period 2005-11-15, 2014-03-12.  Variables are of classes int, numeric, date, and factor.

#### What is/are the main feature(s) of interest in your dataset?
The main feature of the borrowers is their ProsperRating (a proprietary rating system), which is based on their credit score and history with Prosper loans.

For investors, the main features of interest are the LenderYield (interest rate minus the service fee) and the LP_NetPrincipalLoss, which is the principal that remains uncollected after any recoveries.

As a business, Prosper would be most concerned with LP_ServiceFees and LP_CollectionFees, which form their primary revenue source.

#### What other features in the dataset do you think will help support your investigation into your feature(s) of interest?

Prosper sorts borrowers into categories using the ProsperRating and uses the categories to assign interest rates.  I would like to investigate how other variables differ between ProsperRating groups especially default rates and lender yields.

#### Did you create any new variables from existing variables in the dataset?

I created a single CreditScore by averaging the CreditScoreRangeUpper and CreditScoreRangeLower variables.  I also created a factor variable Results for each loan to simplify each loans status as "Current or Paid", "Past Due", or "Defaulted".  I anticipate creating other variables to determine the final profit/loss from each loan for both investors and Prosper. 

#### Of the features you investigated, were there any unusual distributions? Did you perform any operations on the data to tidy, adjust, or change the form of the data? If so, why did you do this?

StatedMonthlyIncome had a very long tail that included values that don't make sense given the size and term of Prosper loans. It seems like the large values are user-entered errors because the other values make logical sense.  The rows with StatedIncomeRange > 25000 should be excluded from any analysis that involves StatedMonthlyIncome or IncomeRange (which I learned was calculated from the StatedMonthlyIncome).

The default rate does not show increasing risk that I would expect.  I plan to investigate the ProsperRating groups further.

## Bivariate Plots Section
### Boxplots
I just want to try making some boxplots.  Boxplots compare a numeric variable to a factor variable.  Well, we have a numeric version of the ProsperRating.

![](figure/unnamed-chunk-11-1.png) 

Well, that's the most boring thing ever.  The numeric rating is just the levels of the Categories.  Well, I can get the ordering over with by combining the two columns.  I'll just call the result Rating, since I'm tired of typing ProsperRating.  Now, what will make a nice boxplot . . . CreditScore!  We can then see how the two measures compare.

![](figure/unnamed-chunk-12-1.png) 

The trend is generally that higher ratings have higher credit scores, but Prosper clearly uses more than credit score, since there is a lot of overlap between the ratings.  

![](figure/unnamed-chunk-13-1.png) 

There is much less overlap in the APR (interest rate) the borrowers are assigned.

### Line graphs
How do default rates vary across the Ratings?

![](figure/unnamed-chunk-14-1.png) 

It's hard to judge since the pre 2009 loans are all lumped together.  I could either exclude them, or try to split them along the CreditGrade variable that was in use prior to 2009.

![](figure/unnamed-chunk-15-1.png) 

This really doesn't stand out as well as I thought it would.  There are high numbers of defaults from categories D through HR.  Perhaps if it were expressed as a rate rather than absolute numbers of loans that end in default.

![](figure/unnamed-chunk-16-1.png) 

That is a lot clearer and the default rates track roughly with the risk categories.  Default rates are much higher than I would have expected.  The E and HR groups, even post-recession have 25-30% of loans default.

![](figure/unnamed-chunk-17-1.png) ![](figure/unnamed-chunk-17-2.png) 

This last plot is just based on the total original loan amount and does not reflect any payments the borrower made before going into default.  Prosper also uses a collections agency to try to recover more money from delinquent borrowers, payments received are used to pay (in order) fees, interest, and principal.  There are two variables relating to the loss from a loan.  LP_GrossPrincipalLoss is the gross charged off amount of the loan.  LP_NetPrincipalLoss the the principal that remains uncollected after any recoveries.  My interpretation of these variables is the the gross loss is the uncollected principal plus unpaid interest and fees.  So, I think that gross loss is of more concern to investors.

![](figure/unnamed-chunk-18-1.png) 

The average loss rate per dollar invested

![](figure/unnamed-chunk-19-1.png) 

I want to see what are the main factors correlate with default.  Based on this plot, I'm going to exclude everything from before July 2009 (the end of the "quiet period") and only include loans that have a ProsperRating.  I'm only going to use loans that are completed, so I will exclude LoanStatus of Current or Past Due.

```
## [1] "Dimensions of new dataset"
```

```
## [1] 26210    14
```

```
## [1] "Loan results by rating"
```

```
##                  
##                     AA    A    B    C    D    E   HR
##   Defaulted       1324 1424 1677  840  588  405   83
##   Current or Paid 2538 2322 4220 3015 2824 3237 1713
```

I also made a column for the InvestorProfit and the InvestorProfitRate for each loan.  With this smaller dataset, I'm ready to look at some scatterplots.

### Scatterplots
![](figure/unnamed-chunk-21-1.png) 

This is not what I expected.  Why would invest in these lending sites if this was the real picture?  So what went wrong?  From [the Prosper website](https://www.prosper.com/invest/marketplace-performance/):  
> Why do we show Seasoned Returns?
As a Prosper investor, your return is based on the lifecycle of the underlying Notes within your portfolio. Because a Note cannot default until it's missed five payments, the return for a portfolio composed solely of young notes will be based entirely on those loans that remain current. This can result in a temporarily higher return for young portfolios than should be expected.
As your Notes age, you may see initial defaults occur between their fifth and ninth months of age. Our research shows that Prosper Note returns historically have shown increased stability after they've reached ten months of age. For that reason, we provide "Actual Seasoned Returns", defined as the Actual Return for Notes aged 10 months or more.

This dataset was saved March 2014, so Prosper would exclude notes after May 2013, which would eliminate the recent losses. Maybe I'm introducing bias by excluding current loans.  While Prosper does not have a prepayment penalty, I would expect most loans that end before their term is up to have defaulted.  Prosper loans have 1, 3, or 5 year terms, so that's March 2013, March 2011, and March 2009 (which is prior to the start of new lending terms). I had excluded current loans, because the investor has not earned back his investment and has a paper loss, that may become a profit with time.  If I subset out only the loans past their term, how many does that leave?  


```
## [1] 10078    14
```

That seems to be enough to play with to see what factors affect default rate. Let's try the Profit/Origination Plot again.

![](figure/unnamed-chunk-23-1.png) 

Now there is an average profit for investors, though not by much for the one year loans.  There is also not much overlap between the two terms, only about six months.  They look like two different populations.  I think I'll just look at the 36 month loans.  How many are there?


```
## [1] 8571   13
```

![](figure/unnamed-chunk-24-1.png) 

Now a regression line shows profit rate to be fairly constant over this time period.

### Correlations

```
##                         Results InvestorProfit InvestorProfitRate Rating
## Results                    1.00           0.63               0.75   0.20
## InvestorProfit             0.63           1.00               0.77  -0.04
## InvestorProfitRate         0.75           0.77               1.00  -0.15
## Rating                     0.20          -0.04              -0.15   1.00
## LoanOriginationDate       -0.04           0.03               0.01  -0.20
## LoanOriginationQuarter       NA             NA                 NA     NA
## LoanOriginalAmount         0.05           0.15              -0.06   0.32
## BorrowerAPR               -0.22           0.05               0.16  -0.94
## ListingCategory              NA             NA                 NA     NA
## CreditScore                0.16           0.01              -0.12   0.75
## RevolvingCreditBalance     0.03           0.05               0.01   0.09
## AvailableBankcardCredit    0.09          -0.01              -0.09   0.47
## StatedMonthlyIncome        0.07           0.09               0.02   0.16
##                         LoanOriginationDate LoanOriginationQuarter
## Results                               -0.04                     NA
## InvestorProfit                         0.03                     NA
## InvestorProfitRate                     0.01                     NA
## Rating                                -0.20                     NA
## LoanOriginationDate                    1.00                     NA
## LoanOriginationQuarter                   NA                     NA
## LoanOriginalAmount                     0.14                     NA
## BorrowerAPR                            0.21                     NA
## ListingCategory                          NA                     NA
## CreditScore                            0.00                     NA
## RevolvingCreditBalance                 0.00                     NA
## AvailableBankcardCredit               -0.04                     NA
## StatedMonthlyIncome                    0.05                     NA
##                         LoanOriginalAmount BorrowerAPR ListingCategory
## Results                               0.05       -0.22              NA
## InvestorProfit                        0.15        0.05              NA
## InvestorProfitRate                   -0.06        0.16              NA
## Rating                                0.32       -0.94              NA
## LoanOriginationDate                   0.14        0.21              NA
## LoanOriginationQuarter                  NA          NA              NA
## LoanOriginalAmount                    1.00       -0.27              NA
## BorrowerAPR                          -0.27        1.00              NA
## ListingCategory                         NA          NA              NA
## CreditScore                           0.46       -0.69              NA
## RevolvingCreditBalance                0.11       -0.07              NA
## AvailableBankcardCredit               0.32       -0.44              NA
## StatedMonthlyIncome                   0.29       -0.13              NA
##                         CreditScore RevolvingCreditBalance
## Results                        0.16                   0.03
## InvestorProfit                 0.01                   0.05
## InvestorProfitRate            -0.12                   0.01
## Rating                         0.75                   0.09
## LoanOriginationDate            0.00                   0.00
## LoanOriginationQuarter           NA                     NA
## LoanOriginalAmount             0.46                   0.11
## BorrowerAPR                   -0.69                  -0.07
## ListingCategory                  NA                     NA
## CreditScore                    1.00                   0.04
## RevolvingCreditBalance         0.04                   1.00
## AvailableBankcardCredit        0.52                   0.19
## StatedMonthlyIncome            0.18                   0.33
##                         AvailableBankcardCredit StatedMonthlyIncome
## Results                                    0.09                0.07
## InvestorProfit                            -0.01                0.09
## InvestorProfitRate                        -0.09                0.02
## Rating                                     0.47                0.16
## LoanOriginationDate                       -0.04                0.05
## LoanOriginationQuarter                       NA                  NA
## LoanOriginalAmount                         0.32                0.29
## BorrowerAPR                               -0.44               -0.13
## ListingCategory                              NA                  NA
## CreditScore                                0.52                0.18
## RevolvingCreditBalance                     0.19                0.33
## AvailableBankcardCredit                    1.00                0.15
## StatedMonthlyIncome                        0.15                1.00
```

The Result of the loan (paid or defaulted) is not really correlated with the other variables.  The highest (relevant) correlations are with BorrowerAPR (-0.22) and Rating (-0.19).  I'm surprised at the correlation between CreditScore and the loan amount (0.46).   
![](figure/unnamed-chunk-26-1.png) 

There is also a correlation between StatedMonthlyIncome and both  LoanOriginalAmount and RevolvingCreditBalance.

![](figure/unnamed-chunk-27-1.png) 

## Bivariate Analysis

#### Talk about some of the relationships you observed in this part of the investigation. How did the feature(s) of interest vary with other features in the dataset?
Loan defaults did not have a strong correlation with any of the expected variables.  The largest correlations are with BorrowerAPR (-0.22) and Rating (-0.19)

Investor profits depend on defaults (of course) and BorrowerAPR (which determines interest income).

#### Did you observe any interesting relationships between the other features (not the main feature(s) of interest)?

StatedMonthlyIncome and CreditScore were both positively correlated with the loan amount, which I found somewhat surprising.  BorrowerAPR was correlated with LoanOriginationDate, reflecting broader changes in interest rates.

#### What was the strongest relationship you found?

The strongest relationship among my variables was between ProsperRating and BorrowerAPR (0.9).  The ProsperRating is based mostly on CreditScore (correlation = 0.7).   

## Multivariate Plots Section


```
## 
## Call:
## lm(formula = CreditScore ~ AvailableBankcardCredit * StatedMonthlyIncome, 
##     data = loans2[loans2$StatedMonthlyIncome < 25000 & loans2$AvailableBankcardCredit < 
##         2e+05, ])
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -290.731  -36.750   -1.805   32.759  148.602 
## 
## Coefficients:
##                                               Estimate Std. Error t value
## (Intercept)                                  6.807e+02  1.166e+00 583.836
## AvailableBankcardCredit                      2.088e-03  5.781e-05  36.113
## StatedMonthlyIncome                          2.750e-03  2.019e-04  13.623
## AvailableBankcardCredit:StatedMonthlyIncome -6.192e-08  7.951e-09  -7.788
##                                             Pr(>|t|)    
## (Intercept)                                  < 2e-16 ***
## AvailableBankcardCredit                      < 2e-16 ***
## StatedMonthlyIncome                          < 2e-16 ***
## AvailableBankcardCredit:StatedMonthlyIncome 7.63e-15 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 47.2 on 8518 degrees of freedom
## Multiple R-squared:  0.3285,	Adjusted R-squared:  0.3283 
## F-statistic:  1389 on 3 and 8518 DF,  p-value: < 2.2e-16
```

![](figure/Multivariate_Plots-1.png) 

This would be the start of a model for credit score.  The residuals show a clear trend indicating that the model is underfit.  None of the other variables I had selected are good candidates to be included.  If I were more interested in credit score, I could add more variables to the dataset to improve the model.  I'd like try modeling investor profit first.


```
## 
## Call:
## lm(formula = InvestorProfitRate ~ BorrowerAPR + LoanOriginalAmount + 
##     StatedMonthlyIncome, data = loans2[loans2$StatedMonthlyIncome < 
##     25000, ])
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -1.22556 -0.02982  0.06545  0.18036  1.05520 
## 
## Coefficients:
##                       Estimate Std. Error t value Pr(>|t|)    
## (Intercept)         -6.119e-03  1.268e-02  -0.483  0.62939    
## BorrowerAPR          5.081e-01  3.618e-02  14.045  < 2e-16 ***
## LoanOriginalAmount  -2.968e-06  1.031e-06  -2.877  0.00402 ** 
## StatedMonthlyIncome  6.309e-06  1.253e-06   5.037 4.83e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.3294 on 8524 degrees of freedom
## Multiple R-squared:  0.02734,	Adjusted R-squared:  0.027 
## F-statistic: 79.88 on 3 and 8524 DF,  p-value: < 2.2e-16
```

![](figure/unnamed-chunk-28-1.png) 

While this model of investor profit rate includes the most promising results of the correlation analysis, it does not do a very good job of fitting the data.  I don't think a linear model is a good choice here.  InvestorProfitRate is the investors' profit on each loan divided by the original amount of the loan.  However, this process does not create a normally distributed variable.

![](figure/unnamed-chunk-29-1.png) 

I'm really looking forward to the next course on Machine Learning.  I'm hoping to really improve my understanding of modeling and predictive functions.  I found a [site](https://www.lendingharbor.com/contact/faq) where someone has modeled just this sort of data and is making money off of it.

## Multivariate Analysis
#### Talk about some of the relationships you observed in this part of the investigation. Were there features that strengthened each other in terms of looking at your feature(s) of interest?  
I first looked at the relationship of income and available credit to credit score.  These two values are enough to explain a third of the credit score value once I included an interaction term.  This interesting because credit scores are not based on income.  The plot of residuals indicates that more variables need to be included in the model.

#### Were there any interesting or surprising interactions between features?  
I found it interesting that larger loan requests were associated with larger incomes and with higher credit scores.  Overall, correlations were lower than I expected. 

#### OPTIONAL: Did you create any models with your dataset? Discuss the strengths and limitations of your model.  

I think it might be better to estimate a rate of defaults, but I'm not sure how to define the population that I would divide the number of defaults by.  I can calculate the percentage of defaults for each rating, but since the rating is attempting to separate loans by default risk, the reasoning seems circular.  I did a little googling and discovered that this type of modeling is far beyond my experience.  [This paper](http://papers.ssrn.com/sol3/papers.cfm?abstract_id=2529240) uses something called a discrete-time hazard model to analyze defaults using Lending Club data.  Lending club is another peer-to-peer lending platform.  Developing models of default risk is very much an active area of research.

# Final Plots and Summary

### Plot One
![](figure/Plot_One-1.png) 

### Description One
Prosper Loan's business history is encoded the the dollar value of the loans originated through their online marketplace.  Prosper was the first peer-to-peer lending marketplace, opening to the public February 5, 2006[[1](https://en.wikipedia.org/wiki/Prosper_Marketplace)].  Initially, lenders bid on loans by offering competing interest rates.  Prosper's business model came under scrutiny by the US Securities and Exchange Commission, who issued a "cease and desist" letter November 24, 2008.[[2](https://www.sec.gov/litigation/admin/2008/33-8984.pdf)]   In anticipation, Prosper filed for SEC registration, which required a "quiet period" from October 15, 2008 until July 13, 2009, during which time, no new loans were originated.[[3](http://www.lendacademy.com/a-look-back-at-the-lending-club-and-prosper-quiet-periods/)]  Prosper attributes the decrease in originations at the end of 2012 to a decrease in liquidity and in January of 2013 undertook an equity financing [[4](https://www.prosper.com/Downloads/Legal/prosper10k12312013.pdf), p 74].  The increase in capital was used in part for a marketing campaign to attract more borrowers and to launch IRA accounts to attract institutional lenders.

### Plot Two
![](figure/Plot_Two-1.png) 

### Description Two
In this plot, we switch from dollar amounts to number of new loans originated each quarter and the final disposition of those loans.  The early days of Prosper were marked by very loose lending standards.  Coupled with the global financial crisis, these early loans had very high default rates and many investors realized losses.  After Prosper's relaunch in 2009, minimum credit scores were increased and Prosper made more of an effort to verify borrower's information[[5](http://www.wsj.com/articles/SB120525138644627455)].  Prosper's prospectus makes it clear that investors should expect some loans to default[[6](https://www.prosper.com/invest/marketplace-performance/)], and charges interest rates high enough to account for risk, but lower than a borrower would get from a credit card.

### Plot Three
![](figure/Plot_Three-1.png) 

### Description Three
While every rating category experiences defaults, investors make money by collecting more (on average) in interest and fees than the principal lost to defaulting borrowers.  Here, principal lost, service fees, and collection fees are subtracted from the interest and (borrower) fees paid to investors.  All rating categories have generated impressive profits since 2009, with generally higher volatility in riskier categories.  This chart explains why peer-to-peer lending is such a hot topic in investing circles.  

# Reflection
I came into this project expecting it to be easy since I had already learned R and covered EDA as part of the Coursera Data Science Specialization that I'm working on concurrently.  Always aiming to be Udacious, I selected the most difficult looking dataset and confidently predicted that I'd be done in a week.  Instead, this project has taught me some deep lessons about data science.

I tried to begin by selecting 10-14 important variables from the list and planned to create a series of histograms, scatterplots, and multivariate figures.  However, the analysis was adrift.  I was creating a series of simple plots, but they were not telling a coherent story.  I found I could display the same variables multiple ways, but had no way to chose between them.  Should I show a histogram of counts?  of summed values? of percentages? changes over time?  and that was just the first column. What about outliers? factor levels? transformations.  The first week had disappeared and I had gotten nowhere.

I realized that unlike the other analyses I had done in R, this project did not come with a question to answer.  But there are a lot of possible questions that could be addressed by this sort of data.  I tried to pretend that I was the different types of stake holders and ask questions of the data from their point of view.  This approach got farther, but was extremely frustrating.  The truth was that I didn't know what I was doing.

I forgot that the third leg of data science is substantive experience.  I put aside the Rmd file and started doing research.  I read all of Prosper's website and their annual report.  I learned about peer-to-peer lending and read what investors, borrowers, and financial gurus had to say about it.  I read about the history of the company, its accomplishments and struggles.  And it started to tell a coherent story.  The variable list made a lot more sense, and I could see what information wasn't included for public release. At last, I can explore. I chose these three plots because together they tell the history of Prosper Loan and how it works.  

To continue from here, I would really like to make a predictive model to compare against the FY 2014 data.  I could treat defaults as a binary factor and use logistic regression except that I don't know enough to extend the technique to multiple regressors that interact with each other.  I am looking forward to learning more about modeling and predictions in the Machine Learning class.

After this analysis, I would consider investing in p2p loans if I was looking to add an income stream to my portfolio.  I will certainly compare their interest rates to banks and credit unions if I am ever looking for a loan.
