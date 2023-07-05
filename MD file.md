<<<<<<< HEAD
---
editor_options: 
  markdown: 
    wrap: 72
---

# Ames, Iowa: Which characteristics predict if a home will sell above or below the median home value?

Our aim in this project is to determine which variables in the Ames
dataset predict whether a home will sell above or below the median home
value.

## Set-Up & Selection

To start I load in the desired libraries.

=======
# Ames, Iowa: Which characteristics predict if a home will sell above or below the median home value?
Our aim in this project is to determine which variables in the Ames dataset predict whether a home will sell above or below the median home value.
## Set-Up & Selection
I start by loading my desired libraries.
>>>>>>> 41c0b8c36b1ab562db0b29093d39f81d13a81e39
``` r
library(tidyverse) 
```

```         
── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
✔ dplyr     1.1.2     ✔ readr     2.1.4
✔ forcats   1.0.0     ✔ stringr   1.5.0
✔ ggplot2   3.4.2     ✔ tibble    3.2.1
✔ lubridate 1.9.2     ✔ tidyr     1.3.0
✔ purrr     1.0.1     
── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
✖ dplyr::filter() masks stats::filter()
✖ dplyr::lag()    masks stats::lag()
ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```

``` r
library(tidymodels)
```

```         
── Attaching packages ────────────────────────────────────── tidymodels 1.1.0 ──
✔ broom        1.0.4     ✔ rsample      1.1.1
✔ dials        1.2.0     ✔ tune         1.1.1
✔ infer        1.0.4     ✔ workflows    1.1.3
✔ modeldata    1.1.0     ✔ workflowsets 1.0.1
✔ parsnip      1.1.0     ✔ yardstick    1.2.0
✔ recipes      1.0.6     
── Conflicts ───────────────────────────────────────── tidymodels_conflicts() ──
✖ scales::discard() masks purrr::discard()
✖ dplyr::filter()   masks stats::filter()
✖ recipes::fixed()  masks stringr::fixed()
✖ dplyr::lag()      masks stats::lag()
✖ yardstick::spec() masks readr::spec()
✖ recipes::step()   masks stats::step()
• Search for functions across packages at https://www.tidymodels.org/find/
```

``` r
library(glmnet) #for Lasso, ridge, and elastic net models 
```

```         
Loading required package: Matrix

Attaching package: 'Matrix'

The following objects are masked from 'package:tidyr':

    expand, pack, unpack

Loaded glmnet 4.1-7
```

``` r
library(GGally) #create ggcorr and ggpairs plots
```

```         
Registered S3 method overwritten by 'GGally':
  method from   
  +.gg   ggplot2
```

``` r
library(ggcorrplot) #create an alternative to ggcorr plots
library(MASS) #access to forward and backward selection algorithms
```

```         
Attaching package: 'MASS'

The following object is masked from 'package:dplyr':

    select
```

``` r
library(leaps) #best subset selection
library(lmtest) #for the dw test
```

```         
Loading required package: zoo

Attaching package: 'zoo'

The following objects are masked from 'package:base':

    as.Date, as.Date.numeric
```

``` r
library(splines) #for nonlinear fitting
library(e1071) 
```

```         
Attaching package: 'e1071'

The following object is masked from 'package:tune':

    tune

The following object is masked from 'package:rsample':

    permutations

The following object is masked from 'package:parsnip':

    tune
```

``` r
library(ROCR)
library(caret)
```

```         
Loading required package: lattice

Attaching package: 'caret'

The following objects are masked from 'package:yardstick':

    precision, recall, sensitivity, specificity

The following object is masked from 'package:purrr':

    lift
```

``` r
library(rpart)
```

```         
Attaching package: 'rpart'

The following object is masked from 'package:dials':

    prune
```

``` r
library(rpart.plot)
library(RColorBrewer)
library(rattle)
```

```         
Loading required package: bitops

Attaching package: 'bitops'

The following object is masked from 'package:Matrix':

    %&%

Rattle: A free graphical interface for data science with R.
Version 5.5.1 Copyright (c) 2006-2021 Togaware Pty Ltd.
Type 'rattle()' to shake, rattle, and roll your data.
```

``` r
library(gridExtra)
```

```         
Attaching package: 'gridExtra'

The following object is masked from 'package:dplyr':

    combine
```

``` r
library(vip)
```

```         
Attaching package: 'vip'

The following object is masked from 'package:utils':

    vi
```

``` r
library(ranger)
```

```         
Attaching package: 'ranger'

The following object is masked from 'package:rattle':

    importance
```

``` r
library(usemodels)
```

Above I have loaded in the required packages. Then I load in the ames
dataset.

``` r
ames = read_csv("ames_student-1.csv")
```

```         
Rows: 2053 Columns: 81
── Column specification ────────────────────────────────────────────────────────
Delimiter: ","
chr (47): MS_SubClass, MS_Zoning, Street, Alley, Lot_Shape, Land_Contour, Ut...
dbl (34): Lot_Frontage, Lot_Area, Year_Built, Year_Remod_Add, Mas_Vnr_Area, ...

ℹ Use `spec()` to retrieve the full column specification for this data.
ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

First I mutate ames to convert character variables to factors.

``` r
ames=
  ames%>%
  mutate_if(is.character,as_factor)
```

Next I check that our response variable, Above_Median, is ordered
correctly so that our negative response is first.

``` r
levels(ames$Above_Median)
```

```         
[1] "Yes" "No" 
```

``` r
ames = ames %>% mutate(Above_Median = fct_relevel(Above_Median, c("No","Yes")))
levels(ames$Above_Median)
```

```         
[1] "No"  "Yes"
```

Then I look at the summary for the ames set. My aim to eliminate
variables that have low variance, or may be heavily skewed one way or
another.

``` r
summary(ames)
```

```         
                               MS_SubClass                         MS_Zoning   
 One_Story_1946_and_Newer_All_Styles :772   Residential_Low_Density     :1600  
 Two_Story_1946_and_Newer            :383   Residential_High_Density    :  20  
 One_and_Half_Story_Finished_All_Ages:204   Floating_Village_Residential:  87  
 One_Story_PUD_1946_and_Newer        :129   Residential_Medium_Density  : 326  
 One_Story_1945_and_Older            : 98   C_all                       :  17  
 Two_Story_1945_and_Older            : 95   A_agr                       :   2  
 (Other)                             :372   I_all                       :   1  
  Lot_Frontage       Lot_Area       Street                 Alley     
 Min.   :  0.00   Min.   :  1300   Pave:2046   No_Alley_Access:1914  
 1st Qu.: 43.00   1st Qu.:  7500   Grvl:   7   Paved          :  45  
 Median : 62.00   Median :  9548               Gravel         :  94  
 Mean   : 57.38   Mean   : 10258                                     
 3rd Qu.: 78.00   3rd Qu.: 11600                                     
 Max.   :313.00   Max.   :215245                                     
                                                                     
                Lot_Shape    Land_Contour  Utilities      Lot_Config  
 Slightly_Irregular  : 714   Lvl:1833     AllPub:2052   Corner : 359  
 Regular             :1275   HLS:  94     NoSewr:   1   Inside :1495  
 Moderately_Irregular:  53   Bnk:  81                   CulDSac: 135  
 Irregular           :  11   Low:  45                   FR2    :  56  
                                                        FR3    :   8  
                                                                      
                                                                      
 Land_Slope        Neighborhood   Condition_1    Condition_2      Bldg_Type   
 Gtl:1951   North_Ames   : 327   Norm   :1771   Norm   :2027   OneFam  :1706  
 Mod:  89   College_Creek: 183   Feedr  : 113   Feedr  :  12   TwnhsE  : 157  
 Sev:  13   Old_Town     : 181   Artery :  67   PosA   :   4   Twnhs   :  67  
            Edwards      : 129   RRAn   :  35   Artery :   4   Duplex  :  76  
            Somerset     : 119   PosN   :  24   PosN   :   3   TwoFmCon:  47  
            Gilbert      : 109   RRAe   :  19   RRNn   :   1                  
            (Other)      :1005   (Other):  24   (Other):   2                  
           House_Style          Overall_Qual        Overall_Cond 
 One_Story       :1052   Average      :587   Average      :1143  
 Two_Story       : 590   Above_Average:518   Above_Average: 376  
 One_and_Half_Fin: 225   Good         :411   Good         : 286  
 SLvl            :  90   Very_Good    :237   Very_Good    :  98  
 SFoyer          :  56   Below_Average:169   Below_Average:  73  
 Two_and_Half_Unf:  19   Excellent    : 70   Fair         :  35  
 (Other)         :  21   (Other)      : 61   (Other)      :  42  
   Year_Built   Year_Remod_Add   Roof_Style     Roof_Matl     Exterior_1st
 Min.   :1875   Min.   :1950   Hip    : 404   CompShg:2023   VinylSd:705  
 1st Qu.:1953   1st Qu.:1965   Gable  :1607   WdShake:   8   MetalSd:319  
 Median :1972   Median :1993   Mansard:   9   Tar&Grv:  17   Wd Sdng:313  
 Mean   :1971   Mean   :1984   Gambrel:  14   WdShngl:   3   HdBoard:303  
 3rd Qu.:2000   3rd Qu.:2004   Shed   :   5   Roll   :   1   Plywood:151  
 Max.   :2010   Max.   :2010   Flat   :  14   Metal  :   1   CemntBd: 90  
                                                             (Other):172  
  Exterior_2nd  Mas_Vnr_Type   Mas_Vnr_Area        Exter_Qual  
 VinylSd:699   Stone  : 166   Min.   :   0.0   Typical  :1272  
 MetalSd:317   None   :1231   1st Qu.:   0.0   Good     : 682  
 Wd Sdng:302   BrkFace: 638   Median :   0.0   Excellent:  78  
 HdBoard:277   BrkCmn :  17   Mean   : 103.8   Fair     :  21  
 Plywood:190   CBlock :   1   3rd Qu.: 164.0                   
 CmentBd: 90                  Max.   :1600.0                   
 (Other):178                                                   
     Exter_Cond    Foundation        Bsmt_Qual         Bsmt_Cond   
 Typical  :1787   CBlock:880   Typical    :911   Good       :  80  
 Good     : 213   PConc :911   Good       :849   Typical    :1833  
 Fair     :  43   Wood  :  4   Excellent  :178   Poor       :   4  
 Excellent:   9   BrkTil:216   No_Basement: 57   No_Basement:  57  
 Poor     :   1   Slab  : 36   Fair       : 57   Fair       :  76  
                  Stone :  6   Poor       :  1   Excellent  :   3  
                                                                   
     Bsmt_Exposure      BsmtFin_Type_1  BsmtFin_SF_1      BsmtFin_Type_2
 Gd         : 199   BLQ        :196    Min.   :1.00   Unf        :1740  
 No         :1331   Rec        :216    1st Qu.:3.00   LwQ        :  64  
 Av         : 284   ALQ        :298    Median :3.00   BLQ        :  47  
 Mn         : 179   GLQ        :578    Mean   :4.21   Rec        :  79  
 No_Basement:  60   Unf        :602    3rd Qu.:7.00   GLQ        :  23  
                    LwQ        :106    Max.   :7.00   No_Basement:  58  
                    No_Basement: 57                   ALQ        :  42  
  BsmtFin_SF_2      Bsmt_Unf_SF     Total_Bsmt_SF   Heating    
 Min.   :   0.00   Min.   :   0.0   Min.   :   0   GasA :2019  
 1st Qu.:   0.00   1st Qu.: 226.0   1st Qu.: 793   GasW :  21  
 Median :   0.00   Median : 460.0   Median : 988   Grav :   6  
 Mean   :  52.57   Mean   : 561.2   Mean   :1055   Wall :   5  
 3rd Qu.:   0.00   3rd Qu.: 801.0   3rd Qu.:1304   Floor:   1  
 Max.   :1526.00   Max.   :2336.0   Max.   :5095   OthW :   1  
                                                               
     Heating_QC   Central_Air   Electrical    First_Flr_SF  Second_Flr_SF   
 Fair     :  61   Y:1916      SBrkr  :1887   Min.   : 432   Min.   :   0.0  
 Typical  : 618   N: 137      FuseA  : 126   1st Qu.: 882   1st Qu.:   0.0  
 Excellent:1040               FuseF  :  33   Median :1088   Median :   0.0  
 Good     : 333               FuseP  :   6   Mean   :1168   Mean   : 326.1  
 Poor     :   1               Unknown:   1   3rd Qu.:1402   3rd Qu.: 701.0  
                                             Max.   :5095   Max.   :1862.0  
                                                                            
 Low_Qual_Fin_SF     Gr_Liv_Area   Bsmt_Full_Bath   Bsmt_Half_Bath   
 Min.   :   0.000   Min.   : 480   Min.   :0.0000   Min.   :0.00000  
 1st Qu.:   0.000   1st Qu.:1137   1st Qu.:0.0000   1st Qu.:0.00000  
 Median :   0.000   Median :1447   Median :0.0000   Median :0.00000  
 Mean   :   4.973   Mean   :1499   Mean   :0.4301   Mean   :0.05796  
 3rd Qu.:   0.000   3rd Qu.:1737   3rd Qu.:1.0000   3rd Qu.:0.00000  
 Max.   :1064.000   Max.   :5095   Max.   :3.0000   Max.   :2.00000  
                                                                     
   Full_Bath       Half_Bath      Bedroom_AbvGr   Kitchen_AbvGr  
 Min.   :0.000   Min.   :0.0000   Min.   :0.000   Min.   :1.000  
 1st Qu.:1.000   1st Qu.:0.0000   1st Qu.:2.000   1st Qu.:1.000  
 Median :2.000   Median :0.0000   Median :3.000   Median :1.000  
 Mean   :1.564   Mean   :0.3751   Mean   :2.855   Mean   :1.047  
 3rd Qu.:2.000   3rd Qu.:1.0000   3rd Qu.:3.000   3rd Qu.:1.000  
 Max.   :4.000   Max.   :2.0000   Max.   :6.000   Max.   :3.000  
                                                                 
    Kitchen_Qual  TotRms_AbvGrd      Functional     Fireplaces   
 Typical  :1070   Min.   : 3.000   Typ    :1896   Min.   :0.000  
 Good     : 790   1st Qu.: 5.000   Min2   :  54   1st Qu.:0.000  
 Excellent: 142   Median : 6.000   Min1   :  51   Median :1.000  
 Fair     :  50   Mean   : 6.442   Mod    :  27   Mean   :0.603  
 Poor     :   1   3rd Qu.: 7.000   Maj1   :  15   3rd Qu.:1.000  
                  Max.   :15.000   Maj2   :   6   Max.   :4.000  
                                   (Other):   4                  
       Fireplace_Qu              Garage_Type     Garage_Finish  Garage_Cars   
 Good        :538   Attchd             :1204   Fin      :509   Min.   :0.000  
 No_Fireplace:993   BuiltIn            : 127   Unf      :872   1st Qu.:1.000  
 Typical     :409   Basment            :  29   RFn      :563   Median :2.000  
 Poor        : 36   Detchd             : 549   No_Garage:109   Mean   :1.774  
 Excellent   : 21   No_Garage          : 108                   3rd Qu.:2.000  
 Fair        : 56   CarPort            :  15                   Max.   :5.000  
                    More_Than_Two_Types:  21                                  
  Garage_Area      Garage_Qual      Garage_Cond             Paved_Drive  
 Min.   :   0   Typical  :1839   Typical  :1872   Partial_Pavement:  42  
 1st Qu.: 320   No_Garage: 109   No_Garage: 109   Paved           :1848  
 Median : 478   Fair     :  85   Fair     :  53   Dirt_Gravel     : 163  
 Mean   : 472   Good     :  16   Excellent:   1                          
 3rd Qu.: 576   Excellent:   2   Poor     :   8                          
 Max.   :1488   Poor     :   2   Good     :  10                          
                                                                         
  Wood_Deck_SF     Open_Porch_SF    Enclosed_Porch   Three_season_porch
 Min.   :   0.00   Min.   :  0.00   Min.   :  0.00   Min.   :  0.000   
 1st Qu.:   0.00   1st Qu.:  0.00   1st Qu.:  0.00   1st Qu.:  0.000   
 Median :   0.00   Median : 27.00   Median :  0.00   Median :  0.000   
 Mean   :  93.52   Mean   : 48.17   Mean   : 23.02   Mean   :  2.799   
 3rd Qu.: 168.00   3rd Qu.: 72.00   3rd Qu.:  0.00   3rd Qu.:  0.000   
 Max.   :1424.00   Max.   :742.00   Max.   :584.00   Max.   :407.000   
                                                                       
  Screen_Porch      Pool_Area            Pool_QC                   Fence     
 Min.   :  0.00   Min.   :  0.000   No_Pool  :2047   No_Fence         :1661  
 1st Qu.:  0.00   1st Qu.:  0.000   Excellent:   2   Minimum_Privacy  : 225  
 Median :  0.00   Median :  0.000   Typical  :   2   Good_Privacy     :  81  
 Mean   : 16.68   Mean   :  1.339   Fair     :   1   Good_Wood        :  77  
 3rd Qu.:  0.00   3rd Qu.:  0.000   Good     :   1   Minimum_Wood_Wire:   9  
 Max.   :576.00   Max.   :800.000                                            
                                                                             
 Misc_Feature    Misc_Val           Mo_Sold         Year_Sold      Sale_Type   
 None:1978    Min.   :    0.00   Min.   : 1.000   Min.   :2006   WD     :1789  
 Gar2:   5    1st Qu.:    0.00   1st Qu.: 4.000   1st Qu.:2007   New    : 163  
 Shed:  66    Median :    0.00   Median : 6.000   Median :2008   COD    :  54  
 Othr:   3    Mean   :   60.12   Mean   : 6.189   Mean   :2008   ConLD  :  16  
 Elev:   1    3rd Qu.:    0.00   3rd Qu.: 8.000   3rd Qu.:2009   ConLI  :   8  
              Max.   :17000.00   Max.   :12.000   Max.   :2010   CWD    :   8  
                                                                 (Other):  15  
 Sale_Condition   Longitude         Latitude     Above_Median
 Normal :1712   Min.   :-93.69   Min.   :41.99   No :1010    
 Partial: 169   1st Qu.:-93.66   1st Qu.:42.02   Yes:1043    
 Family :  30   Median :-93.64   Median :42.03               
 Abnorml: 121   Mean   :-93.64   Mean   :42.03               
 Alloca :  16   3rd Qu.:-93.62   3rd Qu.:42.05               
 AdjLand:   5   Max.   :-93.58   Max.   :42.06               
                                                             
```

``` r
#looking at these results, to decide which variables to remove or slim down
```

I also look at correlation between our predictor variables. To avoid
multicollinearity, I want to observe and remove any predictor variables
with correlation with one another.

``` r
amescorr1 = ames %>%
  dplyr::select(
    Lot_Frontage,
    Lot_Area,
    Year_Built,
    Year_Remod_Add,
    Mas_Vnr_Area,
    BsmtFin_SF_1,
    BsmtFin_SF_2,
    Bsmt_Unf_SF,
    Total_Bsmt_SF
  )

amescorr2 = ames %>%
  dplyr::select(
    First_Flr_SF,
    Second_Flr_SF,
    Low_Qual_Fin_SF,
    Gr_Liv_Area,
    Bsmt_Full_Bath,
    Bsmt_Half_Bath,
    Full_Bath,
    Half_Bath,
    Bedroom_AbvGr,
    Kitchen_AbvGr,
    TotRms_AbvGrd,
    Fireplaces,
    Garage_Cars,
    Garage_Area,
  )

amescorr3 = ames %>%
  dplyr::select(
    Wood_Deck_SF,
    Open_Porch_SF,
    Enclosed_Porch,
    Three_season_porch,
    Screen_Porch,
    Pool_Area,
    Misc_Val,
    Year_Sold,
    Mo_Sold,
    Year_Built,
    Longitude,
    Latitude
  )

ggcorr(ames)
```

```         
Warning in ggcorr(ames): data in column(s) 'MS_SubClass', 'MS_Zoning',
'Street', 'Alley', 'Lot_Shape', 'Land_Contour', 'Utilities', 'Lot_Config',
'Land_Slope', 'Neighborhood', 'Condition_1', 'Condition_2', 'Bldg_Type',
'House_Style', 'Overall_Qual', 'Overall_Cond', 'Roof_Style', 'Roof_Matl',
'Exterior_1st', 'Exterior_2nd', 'Mas_Vnr_Type', 'Exter_Qual', 'Exter_Cond',
'Foundation', 'Bsmt_Qual', 'Bsmt_Cond', 'Bsmt_Exposure', 'BsmtFin_Type_1',
'BsmtFin_Type_2', 'Heating', 'Heating_QC', 'Central_Air', 'Electrical',
'Kitchen_Qual', 'Functional', 'Fireplace_Qu', 'Garage_Type', 'Garage_Finish',
'Garage_Qual', 'Garage_Cond', 'Paved_Drive', 'Pool_QC', 'Fence',
'Misc_Feature', 'Sale_Type', 'Sale_Condition', 'Above_Median' are not numeric
and were ignored
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/looking%20at%20corr%20plots-1.png)

``` r
ggcorr(amescorr1)
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/looking%20at%20corr%20plots-2.png)

``` r
ggcorr(amescorr2)
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/looking%20at%20corr%20plots-3.png)

``` r
ggcorr(amescorr3)
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/looking%20at%20corr%20plots-4.png)

With this information in mind, I remove variables from the Ames dataset.
I selected these based on graphical representation of the variable
vs. the response variable, variance, and strong co-linearity with other
predictor variables. This set had a lot of significant multicollinearity
for example, Garage Area and Garage Cars were strongly correlated, so I
removed Garage Area. Ground Living Area was correlated with many
variables, such as Total Rooms Above Ground, Second Floor Square
Footage, and First Floor Square Footage. Other variables, such as
Heating QC had poor variance, with a massively larger count of one level
than the others. This would interfere with the model and needed removal.
Some variables graphically demonstrated no relationship or a poor
relationship with Above_Median, such as Overall Condition, and were
removed due to this.

``` r
ames_cleaned = ames %>%
  dplyr::select(
    -Street, 
    -Alley, 
    -Utilities, 
    -Lot_Config,
    -Land_Slope, 
    -Land_Contour,
    -Condition_1,
    -Condition_2, 
    -Roof_Style,
    -Roof_Matl,
    -BsmtFin_Type_1, 
    -BsmtFin_SF_1,
    -BsmtFin_Type_2,
    -BsmtFin_SF_2,
    -Bsmt_Cond,
    -Bsmt_Exposure,
    -Bsmt_Unf_SF,
    -Bsmt_Qual,
    -Heating, 
    -Electrical, 
    -Functional, 
    -Heating_QC, 
    -Fence, 
    -Garage_Cond,
    -Garage_Qual,
    -Garage_Finish,
    -Functional,
    -Misc_Feature,
    -Sale_Type,
    -Pool_QC,
    -Lot_Frontage,
    -Bldg_Type,
    -Overall_Cond,
    -First_Flr_SF,
    -Second_Flr_SF,
    -Low_Qual_Fin_SF,
    -Wood_Deck_SF,
    -Open_Porch_SF,
    -Enclosed_Porch,
    -Three_season_porch,
    -Screen_Porch,
    -Wood_Deck_SF,
    -Mas_Vnr_Area,
    -Bsmt_Full_Bath,
    -Bsmt_Half_Bath,
    -Year_Remod_Add,
    -Paved_Drive,
    -Fireplaces,
    -Fireplace_Qu,
    -Mas_Vnr_Type,
    -Exterior_2nd,
    -Garage_Area, 
    -TotRms_AbvGrd, 
    -Year_Remod_Add, 
    -Bedroom_AbvGr, 
    -Longitude,
    -Latitude,
    -Exterior_1st,
    -Exter_Qual)
```

Now it's time to build our models. I start by splitting the data into a
training set and a testing set with strata set as Above_Median.

``` r
#Splitting into a training and testing set. We will build the models off of the training set and then test their accuracy on the testing set. Setting Above_Median as strata guarantees a distribution in both sets centered around the response variable.

set.seed(123) 
ames_cleaned_split = initial_split(ames_cleaned, prop = 0.70, strata = Above_Median)
train = training(ames_cleaned_split)
test = testing(ames_cleaned_split)
```

## Logistic Regression Model

Next I construct the logistic regression model. I chose logistic
regression due to the binary nature of the response variable,
Above_Median. I use step_other for variables that have levels of smaller
count. This will improve the model by avoiding excessive dummy variables
for less-frequent levels. It will condense these less-frequent levels
into an "other" category. I use step_dummy to prepare factor variables
into a numeric representation to enable machine modeling.

``` r
#Setting up the logistic regression model
ames_log_model =
  logistic_reg(mode="classification")%>%
  set_engine("glm")

#recipe for log reg using step_other to condense and step_dummy to set our factor variables as such
ames_log_recipe = recipe(Above_Median~.,train)%>%
  step_other(MS_SubClass, threshold = 0.01) %>%
  step_other(Neighborhood, threshold = 0.01) %>%
  step_other(Overall_Qual, threshold = 0.01) %>%
  step_other(Kitchen_Qual, threshold = 0.01) %>%
  step_other(House_Style, threshold = 0.01) %>%
  step_dummy(all_nominal(),-all_outcomes())

#combining together recipe and model
logreg_wf=workflow()%>%
  add_recipe(ames_log_recipe)%>%
  add_model(ames_log_model)

#fitting the model
ames_log_fit = fit(logreg_wf, train)
```

```         
Warning: glm.fit: fitted probabilities numerically 0 or 1 occurred
```

``` r
#summarizing the results of our model
summary(ames_log_fit$fit$fit$fit)
```

```         
Call:
stats::glm(formula = ..y ~ ., family = stats::binomial, data = data)

Coefficients: (1 not defined because of singularities)
                                                        Estimate Std. Error
(Intercept)                                            2.512e+02  2.281e+02
Lot_Area                                               7.320e-05  3.724e-05
Year_Built                                             2.921e-02  1.658e-02
Total_Bsmt_SF                                          1.431e-03  6.651e-04
Gr_Liv_Area                                            4.464e-03  7.756e-04
Full_Bath                                              9.816e-01  3.963e-01
Half_Bath                                              3.009e-01  4.088e-01
Kitchen_AbvGr                                         -1.186e+01  7.575e+00
Garage_Cars                                            1.667e+00  4.135e-01
Pool_Area                                              2.635e-02  5.495e+00
Misc_Val                                               2.446e-04  1.107e-04
Mo_Sold                                                3.133e-02  5.621e-02
Year_Sold                                             -1.547e-01  1.134e-01
MS_SubClass_Two_Story_1946_and_Newer                   5.434e+00  2.743e+00
MS_SubClass_One_Story_PUD_1946_and_Newer              -3.852e-01  9.765e-01
MS_SubClass_One_and_Half_Story_Finished_All_Ages       4.828e+00  2.299e+00
MS_SubClass_Two_Story_PUD_1946_and_Newer              -1.814e+01  1.340e+03
MS_SubClass_Split_or_Multilevel                        2.278e+01  8.658e+03
MS_SubClass_One_Story_1945_and_Older                  -8.929e-01  1.614e+00
MS_SubClass_Duplex_All_Styles_and_Ages                 5.669e+00  7.556e+00
MS_SubClass_Split_Foyer                                4.339e+00  7.573e+00
MS_SubClass_Two_Family_conversion_All_Styles_and_Ages  6.169e-01  1.527e+00
MS_SubClass_Two_Story_1945_and_Older                   4.768e+00  2.680e+00
MS_SubClass_Two_and_Half_Story_All_Ages               -1.359e-01  8.871e+00
MS_SubClass_other                                     -5.114e+00  3.057e+00
MS_Zoning_Residential_High_Density                     5.295e+00  1.842e+00
MS_Zoning_Floating_Village_Residential                 8.702e+00  4.828e+03
MS_Zoning_Residential_Medium_Density                  -1.134e+00  9.039e-01
MS_Zoning_C_all                                       -2.070e+01  3.829e+03
MS_Zoning_A_agr                                       -1.152e+01  2.963e+04
MS_Zoning_I_all                                       -3.085e+01  2.923e+04
Lot_Shape_Regular                                     -4.903e-02  3.322e-01
Lot_Shape_Moderately_Irregular                        -1.385e+00  1.252e+00
Lot_Shape_Irregular                                    4.883e+00  2.546e+01
Neighborhood_Gilbert                                   4.789e-01  1.022e+00
Neighborhood_Stone_Brook                               3.452e+01  3.399e+03
Neighborhood_Northwest_Ames                            4.213e-01  5.808e-01
Neighborhood_Somerset                                  1.476e+01  4.638e+03
Neighborhood_Northridge_Heights                        1.698e+01  1.340e+03
Neighborhood_Northridge                                1.248e+01  3.406e+03
Neighborhood_Sawyer_West                              -6.529e-01  9.027e-01
Neighborhood_Sawyer                                    5.033e-01  5.985e-01
Neighborhood_Old_Town                                 -1.999e-01  1.305e+00
Neighborhood_Brookside                                 1.888e+00  1.070e+00
Neighborhood_Iowa_DOT_and_Rail_Road                    3.360e+00  1.534e+00
Neighborhood_Clear_Creek                               3.591e+00  1.233e+00
Neighborhood_South_and_West_of_Iowa_State_University   1.906e-01  1.242e+00
Neighborhood_Edwards                                   2.064e-01  7.185e-01
Neighborhood_College_Creek                            -1.499e-01  8.361e-01
Neighborhood_Crawford                                  3.612e+00  1.098e+00
Neighborhood_Mitchell                                  1.138e+00  6.671e-01
Neighborhood_Timberland                                1.372e+00  1.454e+00
Neighborhood_Meadow_Village                           -1.431e+01  5.048e+03
Neighborhood_other                                     6.917e-01  1.039e+00
House_Style_Two_Story                                 -5.544e+00  2.769e+00
House_Style_One_and_Half_Fin                          -4.902e+00  2.258e+00
House_Style_SLvl                                      -2.230e+01  8.658e+03
House_Style_SFoyer                                    -3.895e+00  7.542e+00
House_Style_Two_and_Half_Unf                           1.473e+00  8.900e+00
House_Style_other                                      3.005e+00  8.849e+00
Overall_Qual_Average                                  -7.451e-01  3.553e-01
Overall_Qual_Good                                      1.712e+00  5.285e-01
Overall_Qual_Very_Good                                 7.952e+00  3.255e+00
Overall_Qual_Excellent                                -6.218e+00  1.645e+00
Overall_Qual_Below_Average                            -3.780e+00  1.141e+00
Overall_Qual_Fair                                     -4.042e+01  4.632e+03
Overall_Qual_Very_Excellent                            1.050e+01  4.654e+03
Overall_Qual_other                                    -6.082e+00  4.824e+03
Exter_Cond_Good                                        5.022e-01  4.915e-01
Exter_Cond_Fair                                       -3.632e+00  1.457e+00
Exter_Cond_Excellent                                   1.926e+00  1.501e+00
Exter_Cond_Poor                                               NA         NA
Foundation_PConc                                       1.193e+00  4.813e-01
Foundation_Wood                                       -4.354e+00  5.144e+00
Foundation_BrkTil                                      3.922e-01  8.039e-01
Foundation_Slab                                       -4.392e+00  3.562e+00
Foundation_Stone                                       1.363e+01  2.112e+03
Central_Air_N                                         -2.516e+00  1.265e+00
Kitchen_Qual_Good                                      4.737e-01  3.772e-01
Kitchen_Qual_Excellent                                 3.579e+00  1.306e+00
Kitchen_Qual_Fair                                     -1.484e+00  1.510e+00
Kitchen_Qual_other                                    -1.566e+01  2.923e+04
Garage_Type_BuiltIn                                    1.890e+00  1.060e+00
Garage_Type_Basment                                    1.490e+00  9.262e-01
Garage_Type_Detchd                                    -1.195e+00  4.654e-01
Garage_Type_No_Garage                                 -4.168e+00  8.064e+00
Garage_Type_CarPort                                   -2.132e+01  5.695e+03
Garage_Type_More_Than_Two_Types                        1.204e+00  1.318e+00
Sale_Condition_Partial                                -1.812e+00  9.273e-01
Sale_Condition_Family                                 -4.476e+00  1.437e+00
Sale_Condition_Abnorml                                -2.953e+00  7.304e-01
Sale_Condition_Alloca                                  6.398e+00  5.065e+00
Sale_Condition_AdjLand                                -1.254e+01  1.894e+04
                                                      z value Pr(>|z|)    
(Intercept)                                             1.101 0.270810    
Lot_Area                                                1.966 0.049354 *  
Year_Built                                              1.761 0.078167 .  
Total_Bsmt_SF                                           2.152 0.031404 *  
Gr_Liv_Area                                             5.756 8.63e-09 ***
Full_Bath                                               2.477 0.013255 *  
Half_Bath                                               0.736 0.461683    
Kitchen_AbvGr                                          -1.566 0.117378    
Garage_Cars                                             4.032 5.52e-05 ***
Pool_Area                                               0.005 0.996174    
Misc_Val                                                2.211 0.027061 *  
Mo_Sold                                                 0.557 0.577290    
Year_Sold                                              -1.363 0.172803    
MS_SubClass_Two_Story_1946_and_Newer                    1.981 0.047599 *  
MS_SubClass_One_Story_PUD_1946_and_Newer               -0.395 0.693206    
MS_SubClass_One_and_Half_Story_Finished_All_Ages        2.100 0.035727 *  
MS_SubClass_Two_Story_PUD_1946_and_Newer               -0.014 0.989193    
MS_SubClass_Split_or_Multilevel                         0.003 0.997901    
MS_SubClass_One_Story_1945_and_Older                   -0.553 0.580030    
MS_SubClass_Duplex_All_Styles_and_Ages                  0.750 0.453127    
MS_SubClass_Split_Foyer                                 0.573 0.566708    
MS_SubClass_Two_Family_conversion_All_Styles_and_Ages   0.404 0.686301    
MS_SubClass_Two_Story_1945_and_Older                    1.779 0.075246 .  
MS_SubClass_Two_and_Half_Story_All_Ages                -0.015 0.987776    
MS_SubClass_other                                      -1.673 0.094390 .  
MS_Zoning_Residential_High_Density                      2.874 0.004052 ** 
MS_Zoning_Floating_Village_Residential                  0.002 0.998562    
MS_Zoning_Residential_Medium_Density                   -1.254 0.209686    
MS_Zoning_C_all                                        -0.005 0.995687    
MS_Zoning_A_agr                                         0.000 0.999690    
MS_Zoning_I_all                                        -0.001 0.999158    
Lot_Shape_Regular                                      -0.148 0.882675    
Lot_Shape_Moderately_Irregular                         -1.106 0.268812    
Lot_Shape_Irregular                                     0.192 0.847891    
Neighborhood_Gilbert                                    0.469 0.639426    
Neighborhood_Stone_Brook                                0.010 0.991897    
Neighborhood_Northwest_Ames                             0.725 0.468241    
Neighborhood_Somerset                                   0.003 0.997460    
Neighborhood_Northridge_Heights                         0.013 0.989884    
Neighborhood_Northridge                                 0.004 0.997075    
Neighborhood_Sawyer_West                               -0.723 0.469487    
Neighborhood_Sawyer                                     0.841 0.400324    
Neighborhood_Old_Town                                  -0.153 0.878288    
Neighborhood_Brookside                                  1.765 0.077589 .  
Neighborhood_Iowa_DOT_and_Rail_Road                     2.190 0.028534 *  
Neighborhood_Clear_Creek                                2.914 0.003574 ** 
Neighborhood_South_and_West_of_Iowa_State_University    0.153 0.878044    
Neighborhood_Edwards                                    0.287 0.773969    
Neighborhood_College_Creek                             -0.179 0.857730    
Neighborhood_Crawford                                   3.290 0.001003 ** 
Neighborhood_Mitchell                                   1.705 0.088169 .  
Neighborhood_Timberland                                 0.944 0.345325    
Neighborhood_Meadow_Village                            -0.003 0.997738    
Neighborhood_other                                      0.665 0.505780    
House_Style_Two_Story                                  -2.002 0.045306 *  
House_Style_One_and_Half_Fin                           -2.171 0.029928 *  
House_Style_SLvl                                       -0.003 0.997944    
House_Style_SFoyer                                     -0.516 0.605508    
House_Style_Two_and_Half_Unf                            0.165 0.868568    
House_Style_other                                       0.340 0.734198    
Overall_Qual_Average                                   -2.097 0.035968 *  
Overall_Qual_Good                                       3.240 0.001197 ** 
Overall_Qual_Very_Good                                  2.443 0.014573 *  
Overall_Qual_Excellent                                 -3.780 0.000157 ***
Overall_Qual_Below_Average                             -3.313 0.000922 ***
Overall_Qual_Fair                                      -0.009 0.993037    
Overall_Qual_Very_Excellent                             0.002 0.998199    
Overall_Qual_other                                     -0.001 0.998994    
Exter_Cond_Good                                         1.022 0.306875    
Exter_Cond_Fair                                        -2.493 0.012672 *  
Exter_Cond_Excellent                                    1.283 0.199501    
Exter_Cond_Poor                                            NA       NA    
Foundation_PConc                                        2.478 0.013204 *  
Foundation_Wood                                        -0.846 0.397321    
Foundation_BrkTil                                       0.488 0.625633    
Foundation_Slab                                        -1.233 0.217613    
Foundation_Stone                                        0.006 0.994853    
Central_Air_N                                          -1.989 0.046742 *  
Kitchen_Qual_Good                                       1.256 0.209087    
Kitchen_Qual_Excellent                                  2.741 0.006123 ** 
Kitchen_Qual_Fair                                      -0.983 0.325617    
Kitchen_Qual_other                                     -0.001 0.999573    
Garage_Type_BuiltIn                                     1.783 0.074584 .  
Garage_Type_Basment                                     1.609 0.107631    
Garage_Type_Detchd                                     -2.568 0.010242 *  
Garage_Type_No_Garage                                  -0.517 0.605230    
Garage_Type_CarPort                                    -0.004 0.997013    
Garage_Type_More_Than_Two_Types                         0.913 0.361147    
Sale_Condition_Partial                                 -1.954 0.050686 .  
Sale_Condition_Family                                  -3.115 0.001841 ** 
Sale_Condition_Abnorml                                 -4.042 5.29e-05 ***
Sale_Condition_Alloca                                   1.263 0.206574    
Sale_Condition_AdjLand                                 -0.001 0.999472    
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 1991.74  on 1436  degrees of freedom
Residual deviance:  375.19  on 1345  degrees of freedom
AIC: 559.19

Number of Fisher Scoring iterations: 20
```

My model has an AIC of 559.19. By itself this does not necessarily mean
much. However, if we were to add or subtract variables from the model, I
could use the AIC to compare the quality of the model. A lower AIC is
considered to be an increase in quality.

From our model we can see which variables are significant in the
determining the probability of a home selling above or below the median
home value. Most appear to follow a logical pattern, but I am not
content with House Style and Overall Quality. House Style's negative
coefficient for Two Story appears to be in direct contract with
MS_Subclass's positive coefficient for Two Story 1946 and Newer.
Furthermore, House Style contradicts the visualization I performed in
the preliminary descriptive data analysis.

With this in mind I will continue to assess the performance of this
model. I will use accuracy, sensitivity, specificity, and AUC to
determine model quality on both the training and testing sets.

``` r
#Developing predictions
predictions = predict(ames_log_fit, train, type="prob")
```

```         
Warning in predict.lm(object, newdata, se.fit, scale = 1, type = if (type == :
prediction from rank-deficient fit; attr(*, "non-estim") has doubtful cases
```

``` r
head(predictions)
```

```         
# A tibble: 6 × 2
  .pred_No .pred_Yes
     <dbl>     <dbl>
1    0.937  6.31e- 2
2    0.999  5.03e- 4
3    1      2.22e-16
4    1      2.22e-16
5    0.994  6.06e- 3
6    0.640  3.60e- 1
```

``` r
#ROCR plot
predictions = predict(ames_log_fit, train, type="prob")[2] #extracting the "yes" prediction
```

```         
Warning in predict.lm(object, newdata, se.fit, scale = 1, type = if (type == :
prediction from rank-deficient fit; attr(*, "non-estim") has doubtful cases
```

``` r
ROCRpred = prediction(predictions, train$Above_Median)
ROCRperf = performance(ROCRpred, "tpr", "fpr")
plot(ROCRperf, colorize=TRUE, print.cutoffs.at=seq(0,1,by=0.1), text.adj=c(-0.2,1.7))
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/ROCR,%20AUC,%20accuracy%20-%20train-1.png)

``` r
#AUC
as.numeric(performance(ROCRpred, "auc")@y.values)
```

```         
[1] 0.9895119
```

``` r
#balance sensitivity & specificity - cut off/threshold
opt.cut = function(perf, pred){
    cut.ind = mapply(FUN=function(x, y, p){
        d = (x - 0)^2 + (y-1)^2
        ind = which(d == min(d))
        c(sensitivity = y[[ind]], specificity = 1-x[[ind]], 
            cutoff = p[[ind]])
    }, perf@x.values, perf@y.values, pred@cutoffs)
}
print(opt.cut(ROCRperf, ROCRpred))
```

```         
                 [,1]
sensitivity 0.9383562
specificity 0.9561528
cutoff      0.5606287
```

From this, we have an AUC of 0.9895119. The closer an AUC is to 1, the
better.

``` r
#I then use the cutoff value found from ROCR and used this to create a confusion matrix and pinpoint the accuracy of my model with this cutoff.

t1 = table(train$Above_Median,predictions > 0.5606287)
t1
```

```         
      FALSE TRUE
  No    676   31
  Yes    45  685
```

``` r
(t1[1,1]+t1[2,2])/nrow(train)
```

```         
[1] 0.947112
```

The accuracy of the model on the training set with a cut off of
0.5606287 is 0.946112.

Next we will check the accuracy of the model on the testing set.

``` r
#Developing predictions
predictions2 = predict(ames_log_fit, test, type="prob")
head(predictions2)
```

```         
# A tibble: 6 × 2
  .pred_No .pred_Yes
     <dbl>     <dbl>
1 6.97e- 1     0.303
2 3.94e- 5     1.00 
3 2.22e-16     1    
4 2.22e-16     1    
5 2.22e-16     1    
6 4.84e-13     1.00 
```

``` r
#ROCR plot
predictions2 = predict(ames_log_fit, test, type="prob")[2] #extracting the "yes" prediction
ROCRpred = prediction(predictions2, test$Above_Median)
ROCRperf = performance(ROCRpred, "tpr", "fpr")
plot(ROCRperf, colorize=TRUE, print.cutoffs.at=seq(0,1,by=0.1), text.adj=c(-0.2,1.7))
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/ROCR,%20AUC,%20accuracy%20for%20test-1.png)

``` r
#AUC
as.numeric(performance(ROCRpred, "auc")@y.values)
```

```         
[1] 0.9512121
```

``` r
#balance sensitivity & specificity - cut off/threshold
opt.cut = function(perf, pred){
    cut.ind = mapply(FUN=function(x, y, p){
        d = (x - 0)^2 + (y-1)^2
        ind = which(d == min(d))
        c(sensitivity = y[[ind]], specificity = 1-x[[ind]], 
            cutoff = p[[ind]])
    }, perf@x.values, perf@y.values, pred@cutoffs)
}
print(opt.cut(ROCRperf, ROCRpred))
```

```         
                 [,1]
sensitivity 0.9105431
specificity 0.8844884
cutoff      0.3447664
```

AUC for the testing set is 0.9512121. This is lower than on the training
set but is still very close to 1. The cutoff for the model on the
testing set is 0.3447664. The sensitivity is 0.9105431 and specificity
is 0.8844884.

``` r
t2 = table(test$Above_Median,predictions2 > 0.3447664)
t2
```

```         
      FALSE TRUE
  No    268   35
  Yes    29  284
```

``` r
(t2[1,1]+t2[2,2])/nrow(test)
```

```         
[1] 0.8961039
```

Here we can see the accuracy on the testing set to be 0.8961039.

## Logistic Regression with Lasso

While the previous logistic regression had good accuracy, sensitivity,
and specificity, I was not content with all of the variable
coefficients. I decided to trial a lasso logistic regression to see
if the model can be improved.

``` r
set.seed(123)
folds=vfold_cv(train,v=5) 
```

``` r
#use models generates a code template.
use_glmnet(formula = Above_Median~., data = train)
```

```         
glmnet_recipe <- 
  recipe(formula = Above_Median ~ ., data = train) %>% 
  step_zv(all_predictors()) %>% 
  step_normalize(all_numeric_predictors()) 

glmnet_spec <- 
  logistic_reg(penalty = tune(), mixture = tune()) %>% 
  set_mode("classification") %>% 
  set_engine("glmnet") 

glmnet_workflow <- 
  workflow() %>% 
  add_recipe(glmnet_recipe) %>% 
  add_model(glmnet_spec) 

glmnet_grid <- tidyr::crossing(penalty = 10^seq(-6, -1, length.out = 20), mixture = c(0.05, 
    0.2, 0.4, 0.6, 0.8, 1)) 

glmnet_tune <- 
  tune_grid(glmnet_workflow, resamples = stop("add your rsample object"), grid = glmnet_grid) 
```

``` r
#I modified the usemodels template for glmnet by including step_other and step_dummy for the same purposes in the previous logistic regression. I also included step_normalize to scale and center the variables as required by lasso regressions.

glmnet_recipe <- 
  recipe(formula = Above_Median ~ ., data = train) %>% 
  step_other(MS_SubClass, threshold = 0.01) %>%
  step_other(MS_Zoning, threshold = 0.01) %>%
  step_other(Exter_Cond, threshold = 0.01) %>%
  step_other(Neighborhood, threshold = 0.01) %>%
  step_other(Overall_Qual, threshold = 0.01) %>%
  step_other(Kitchen_Qual, threshold = 0.05) %>% #I had to increase the threshold on Kitchen Quality to avoid variance issues by capturing the "other" categories
  step_other(House_Style, threshold = 0.01) %>%
  step_dummy(all_nominal(), -all_outcomes())%>%
  step_normalize(all_predictors(), -all_nominal())

glmnet_spec <- 
  logistic_reg(penalty = tune(), mixture = 1) %>% #mixture of 1 to trigger lasso (as opposed to 0 would trigger ridge)
  set_mode("classification") %>% 
  set_engine("glmnet") 

glmnet_workflow <- 
  workflow() %>% 
  add_recipe(glmnet_recipe) %>% 
  add_model(glmnet_spec) 

glmnet_grid = grid_regular(penalty(), levels = 100)

#Using mean log loss for our lambda metric to generate probabilities.
glmnet_tune =
  tune_grid(glmnet_workflow, resamples = folds, 
            grid = glmnet_grid, metrics = metric_set(mn_log_loss))
```

Next I will take this model and plot penalty vs mean of log loss. I want
to try to find the optimal penalty value in order to be closest

``` r
glmnet_tune %>%
  collect_metrics() %>%
  ggplot(aes(penalty, mean)) +
  geom_errorbar(aes(
    ymin = mean - std_err,
    ymax = mean + std_err
  ),
  alpha = 0.5
  ) +
  geom_line(size = 1.5) +
  theme(legend.position = "none") 
```

```         
Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
ℹ Please use `linewidth` instead.
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/penalty%20value%20v%20mean%20log%20loss%20plot-1.png)

Zooming in on this penalty value peak.

``` r
glmnet_tune %>%
  collect_metrics() %>%
  ggplot(aes(penalty, mean)) +
  geom_errorbar(aes(
    ymin = mean - std_err,
    ymax = mean + std_err
  ),
  alpha = 0.5
  ) +
  geom_line(size = 1.5) +
  theme(legend.position = "none") +
  xlim(0,0.1)
```

```         
Warning: Removed 10 rows containing missing values (`geom_line()`).
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/penalty%20value%20v%20mean%20log%20loss%20zoomed%20in-1.png)

This appears to peak very close to 0.000-0.005. Next I can extract this
best value for mean of log loss.

``` r
best_mnlog = glmnet_tune %>%
  select_best("mn_log_loss")
best_mnlog
```

```         
# A tibble: 1 × 2
  penalty .config               
    <dbl> <chr>                 
1 0.00376 Preprocessor1_Model076
```

Our ideal penalty value is thus 0.003764936.

``` r
final_lasso = glmnet_workflow %>% finalize_workflow(best_mnlog)
lasso_fit = fit(final_lasso, train)
```

``` r
tidy(lasso_fit)
```

```         
# A tibble: 89 × 3
   term          estimate penalty
   <chr>            <dbl>   <dbl>
 1 (Intercept)     0.328  0.00376
 2 Lot_Area        0.0765 0.00376
 3 Year_Built      0.264  0.00376
 4 Total_Bsmt_SF   0.633  0.00376
 5 Gr_Liv_Area     1.33   0.00376
 6 Full_Bath       0.477  0.00376
 7 Half_Bath       0.122  0.00376
 8 Kitchen_AbvGr  -0.770  0.00376
 9 Garage_Cars     0.660  0.00376
10 Pool_Area       0      0.00376
# ℹ 79 more rows
```

The coefficients for the lasso logistic regression appear more in line
with logical expectations than the logistic regression alone. But how is
the model performing? I will look at thresholds for this model.

``` r
#generating our predictions based on the model
predictions = predict(lasso_fit, train, type="prob")[2]

#generating the ROC
ROCRpred = prediction(predictions, train$Above_Median) 
ROCRperf = performance(ROCRpred, "tpr", "fpr")
plot(ROCRperf, colorize=TRUE, print.cutoffs.at=seq(0,1,by=0.1), text.adj=c(-0.2,1.7))
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/predictions%20and%20ROCR%20-%20train-1.png)

``` r
#AUC
as.numeric(performance(ROCRpred, "auc")@y.values)
```

```         
[1] 0.9853888
```

``` r
#balance sensitivity & specificity - cut off/threshold
opt.cut = function(perf, pred){
    cut.ind = mapply(FUN=function(x, y, p){
        d = (x - 0)^2 + (y-1)^2
        ind = which(d == min(d))
        c(sensitivity = y[[ind]], specificity = 1-x[[ind]], 
            cutoff = p[[ind]])
    }, perf@x.values, perf@y.values, pred@cutoffs)
}
print(opt.cut(ROCRperf, ROCRpred))
```

```         
                 [,1]
sensitivity 0.9315068
specificity 0.9405941
cutoff      0.5121469
```

For this model on the training set, the sensitivity is 0.9315068,
specificity is 0.9405941, and AUC is 0.9853888. Let's use the cutoff
value to find the accuracy.

``` r
t3 = table(train$Above_Median,predictions > 0.5121469)
t3
```

```         
      FALSE TRUE
  No    665   42
  Yes    51  679
```

``` r
(t3[1,1]+t3[2,2])/nrow(train)
```

```         
[1] 0.9352818
```

This model has an accuracy of 0.93528118 on the training set.

``` r
#generating our predictions based on the model
predictions = predict(lasso_fit, test, type="prob")[2]

#generating the ROC
ROCRpred = prediction(predictions, test$Above_Median) 
ROCRperf = performance(ROCRpred, "tpr", "fpr")
plot(ROCRperf, colorize=TRUE, print.cutoffs.at=seq(0,1,by=0.1), text.adj=c(-0.2,1.7))
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/predictions%20and%20ROCR%20-%20test-1.png)

``` r
#AUC
as.numeric(performance(ROCRpred, "auc")@y.values)
```

```         
[1] 0.9695378
```

``` r
#balance sensitivity & specificity - cut off/threshold
opt.cut = function(perf, pred){
    cut.ind = mapply(FUN=function(x, y, p){
        d = (x - 0)^2 + (y-1)^2
        ind = which(d == min(d))
        c(sensitivity = y[[ind]], specificity = 1-x[[ind]], 
            cutoff = p[[ind]])
    }, perf@x.values, perf@y.values, pred@cutoffs)
}
print(opt.cut(ROCRperf, ROCRpred))
```

```         
                 [,1]
sensitivity 0.9329073
specificity 0.8943894
cutoff      0.3949216
```

For this model on the testing set, the AUC is 0.9695378, sensitivity is
0.9329073, and specificity is 0.8943894. All of these out-perform the
previous logistic regression on the testing set (0.9512121 AUC,
sensitivity 0.9105431 and specificity 0.8844884). We will use the
cut-off value to see if it out-performs with accuracy as well.

``` r
t4 = table(test$Above_Median,predictions > 0.3949216)
t4
```

```         
      FALSE TRUE
  No    271   32
  Yes    21  292
```

``` r
(t4[1,1]+t4[2,2])/nrow(test)
```

```         
[1] 0.913961
```

On the testing set the logistic lasso regression has an accuracy of
0.913961. This out-performs the previous logistic regression
(accuracy on testing set of 0.8961039).

Overall, the logistic lasso regression out-performs the previous
logistic regression without lasso when applied to the testing set for
both AUC and accuracy. Furthermore, the coefficients generated make more
logical sense and correspond with the previous descriptive analysis.

## Classification Tree

Classification trees are decision trees that allow predictions to be
made based off of the interaction between variables. I first built a
simple classification tree off of the training set.

``` r
#Building the tree
ames_classtree_recipe = recipe(Above_Median~., train)

tree_model = decision_tree() %>% 
  set_engine("rpart", model = TRUE) %>%
  set_mode("classification")

ames_classtree_wflow = 
  workflow() %>% 
  add_model(tree_model) %>% 
  add_recipe(ames_classtree_recipe)

ames_classtree_fit = fit(ames_classtree_wflow, train)

#tree fit
tree = ames_classtree_fit %>% 
  pull_workflow_fit() %>% 
  pluck("fit")
```

```         
Warning: `pull_workflow_fit()` was deprecated in workflows 0.2.3.
ℹ Please use `extract_fit_parsnip()` instead.
```

``` r
fancyRpartPlot(tree, tweak=0.75)
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/visualizing%20the%20tree-1.png)

The tree looks alright, if a bit difficult to read. Next I need to
assess the accuracy of the tree. I do this using a confusion matrix.

``` r
#Confusion matrix for the simple tree
treepred = predict(ames_classtree_fit, train, type = "class")
head(treepred)
```

```         
# A tibble: 6 × 1
  .pred_class
  <fct>      
1 No         
2 No         
3 No         
4 No         
5 No         
6 No         
```

``` r
confusionMatrix(treepred$.pred_class,train$Above_Median,positive="Yes")
```

```         
Confusion Matrix and Statistics

          Reference
Prediction  No Yes
       No  654  95
       Yes  53 635
                                          
               Accuracy : 0.897           
                 95% CI : (0.8801, 0.9122)
    No Information Rate : 0.508           
    P-Value [Acc > NIR] : < 2.2e-16       
                                          
                  Kappa : 0.7942          
                                          
 Mcnemar's Test P-Value : 0.0007512       
                                          
            Sensitivity : 0.8699          
            Specificity : 0.9250          
         Pos Pred Value : 0.9230          
         Neg Pred Value : 0.8732          
             Prevalence : 0.5080          
         Detection Rate : 0.4419          
   Detection Prevalence : 0.4788          
      Balanced Accuracy : 0.8974          
                                          
       'Positive' Class : Yes             
                                          
```

The initial tree is not too bad. I have a sensitivity of 0.8699,
specificity of 0.9250, and accuracy of 0.8974. However, I'm wondering if
I can improve these.

On the training set this tree has an accuracy of 0.8970, sensitivity of
0.8699, and specificity of 0.9250. I want to see how this performs on
the testing set.

``` r
treepred = predict(ames_classtree_fit, test, type = "class")
head(treepred)
```

```         
# A tibble: 6 × 1
  .pred_class
  <fct>      
1 No         
2 Yes        
3 Yes        
4 Yes        
5 Yes        
6 Yes        
```

``` r
confusionMatrix(treepred$.pred_class,test$Above_Median,positive="Yes")
```

```         
Confusion Matrix and Statistics

          Reference
Prediction  No Yes
       No  267  47
       Yes  36 266
                                          
               Accuracy : 0.8653          
                 95% CI : (0.8357, 0.8912)
    No Information Rate : 0.5081          
    P-Value [Acc > NIR] : <2e-16          
                                          
                  Kappa : 0.7306          
                                          
 Mcnemar's Test P-Value : 0.2724          
                                          
            Sensitivity : 0.8498          
            Specificity : 0.8812          
         Pos Pred Value : 0.8808          
         Neg Pred Value : 0.8503          
             Prevalence : 0.5081          
         Detection Rate : 0.4318          
   Detection Prevalence : 0.4903          
      Balanced Accuracy : 0.8655          
                                          
       'Positive' Class : Yes             
                                          
```

On the testing set, this tree has an accuracy of 0.8653, sensitivity of
0.8498, and specificity of 0.8812.

I want to improve this tree. To maximize the accuracy of my
classification tree, I want to find the most optimal complexity
parameter, or 'cp' value. I am going to have R do this for me.

``` r
set.seed(123) #Specifies randomness to makes sure the randomness in the code will generate the same results.
folds=vfold_cv(train,v=5) #5-fold cross-validation on the training data

#Recipe for the tree
ames_classtree2_recipe = recipe(Above_Median~., train) %>%
  step_dummy(all_nominal(),-all_outcomes())

#Model for the tree
ames_classtree2_model = decision_tree(cost_complexity = tune()) %>% 
  set_engine("rpart", model = TRUE) %>%
  set_mode("classification")

#This specifies that we are looking for 
tree_grid = grid_regular(cost_complexity(),
                          levels = 25)

#Workflow for the tree
ames_tree2_wflow = 
  workflow() %>% 
  add_model(ames_classtree2_model) %>% 
  add_recipe(ames_classtree2_recipe)

#Will gather information on evaluation metrics after tuning
tree_res = 
  ames_tree2_wflow %>% 
  tune_grid(
    resamples = folds,
    grid = tree_grid
    )

tree_res
```

```         
# Tuning results
# 5-fold cross-validation 
# A tibble: 5 × 4
  splits             id    .metrics          .notes          
  <list>             <chr> <list>            <list>          
1 <split [1149/288]> Fold1 <tibble [50 × 5]> <tibble [0 × 3]>
2 <split [1149/288]> Fold2 <tibble [50 × 5]> <tibble [0 × 3]>
3 <split [1150/287]> Fold3 <tibble [50 × 5]> <tibble [0 × 3]>
4 <split [1150/287]> Fold4 <tibble [50 × 5]> <tibble [0 × 3]>
5 <split [1150/287]> Fold5 <tibble [50 × 5]> <tibble [0 × 3]>
```

``` r
#This will extract metrics from our plotted cost complexity
tree_res %>%
  collect_metrics() %>%
  ggplot(aes(cost_complexity, mean)) +
  geom_line(size = 1.5, alpha = 0.6) +
  geom_point(size = 2) +
  facet_wrap(~ .metric, scales = "free", nrow = 2) 
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/deriving%20the%20best%20tree-1.png)

``` r
#Creates the best tree from the most optimal accuracy measurement

best_tree = tree_res %>%
  select_best("accuracy")

best_tree
```

```         
# A tibble: 1 × 2
  cost_complexity .config              
            <dbl> <chr>                
1         0.00750 Preprocessor1_Model22
```

With the optimal accuracy and cp value captured in "best_tree" object I
can now plot the tree.

``` r
#Workflow for the new tree with the best_tree object
final_classtree_wf = 
  ames_tree2_wflow %>% 
  finalize_workflow(best_tree)

#Fitting
final_fit = fit(final_classtree_wf, train)

tree2 = final_fit %>% 
  pull_workflow_fit() %>% 
  pluck("fit")
```

``` r
fancyRpartPlot(tree2, tweak = 1.25) 
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/visualizing%20the%20tree%20part%202-1.png)

Immediately, it's notable that this tree has different variables
included than the original tree.

``` r
treepred = predict(final_fit, train, type = "class")
head(treepred)
```

```         
# A tibble: 6 × 1
  .pred_class
  <fct>      
1 No         
2 No         
3 No         
4 No         
5 No         
6 No         
```

``` r
confusionMatrix(treepred$.pred_class,train$Above_Median,positive="Yes")
```

```         
Confusion Matrix and Statistics

          Reference
Prediction  No Yes
       No  643  70
       Yes  64 660
                                          
               Accuracy : 0.9068          
                 95% CI : (0.8905, 0.9213)
    No Information Rate : 0.508           
    P-Value [Acc > NIR] : <2e-16          
                                          
                  Kappa : 0.8135          
                                          
 Mcnemar's Test P-Value : 0.6658          
                                          
            Sensitivity : 0.9041          
            Specificity : 0.9095          
         Pos Pred Value : 0.9116          
         Neg Pred Value : 0.9018          
             Prevalence : 0.5080          
         Detection Rate : 0.4593          
   Detection Prevalence : 0.5038          
      Balanced Accuracy : 0.9068          
                                          
       'Positive' Class : Yes             
                                          
```

The new confusion matrix shows an improved accuracy of 0.9068.
Sensitivity declined slightly to 0.9041 but Specificity increased to
0.9095. Overall this new tree has improved accuracy and specificity on
the training. set.

Now we will look at how this tree performs on the testing set.

``` r
treepred = predict(final_fit, test, type = "class")
head(treepred)
```

```         
# A tibble: 6 × 1
  .pred_class
  <fct>      
1 No         
2 Yes        
3 Yes        
4 Yes        
5 Yes        
6 Yes        
```

``` r
confusionMatrix(treepred$.pred_class,test$Above_Median,positive="Yes")
```

```         
Confusion Matrix and Statistics

          Reference
Prediction  No Yes
       No  272  40
       Yes  31 273
                                          
               Accuracy : 0.8847          
                 95% CI : (0.8568, 0.9089)
    No Information Rate : 0.5081          
    P-Value [Acc > NIR] : <2e-16          
                                          
                  Kappa : 0.7695          
                                          
 Mcnemar's Test P-Value : 0.3424          
                                          
            Sensitivity : 0.8722          
            Specificity : 0.8977          
         Pos Pred Value : 0.8980          
         Neg Pred Value : 0.8718          
             Prevalence : 0.5081          
         Detection Rate : 0.4432          
   Detection Prevalence : 0.4935          
      Balanced Accuracy : 0.8849          
                                          
       'Positive' Class : Yes             
                                          
```

On the testing set we see an accuracy of 0.8847, sensitivity of 0.8722,
and specificity of 0.8977. This is also improved from the previous tree.

## Random Forests

``` r
#recipe
ames_rf_recipe = recipe(Above_Median~., train) %>%
  step_dummy(all_nominal(), -all_outcomes())

#model
rf_model = rand_forest() %>% 
  set_engine("ranger", importance = "permutation") %>%
  set_mode("classification")

#putting it all together
ames_rf_wflow = 
  workflow() %>% 
  add_model(rf_model) %>% 
  add_recipe(ames_rf_recipe)

set.seed(123)
#fit model to the training set
ames_rf_fit = fit(ames_rf_wflow, train)
```

Now that the random tree is set, I want to see how it performs on the
testing and training sets.

``` r
trainpredrf = predict(ames_rf_fit, train)
head(trainpredrf)
```

```         
# A tibble: 6 × 1
  .pred_class
  <fct>      
1 No         
2 No         
3 No         
4 No         
5 No         
6 No         
```

``` r
confusionMatrix(trainpredrf$.pred_class, train$Above_Median, 
                positive = "Yes")
```

```         
Confusion Matrix and Statistics

          Reference
Prediction  No Yes
       No  695   8
       Yes  12 722
                                          
               Accuracy : 0.9861          
                 95% CI : (0.9786, 0.9915)
    No Information Rate : 0.508           
    P-Value [Acc > NIR] : <2e-16          
                                          
                  Kappa : 0.9722          
                                          
 Mcnemar's Test P-Value : 0.5023          
                                          
            Sensitivity : 0.9890          
            Specificity : 0.9830          
         Pos Pred Value : 0.9837          
         Neg Pred Value : 0.9886          
             Prevalence : 0.5080          
         Detection Rate : 0.5024          
   Detection Prevalence : 0.5108          
      Balanced Accuracy : 0.9860          
                                          
       'Positive' Class : Yes             
                                          
```

On the training set, it has an accuracy of 0.9861, sensitivity of
0.9890, and specificity of 0.9830.

``` r
testpredrf = predict(ames_rf_fit, test)
head(testpredrf)
```

```         
# A tibble: 6 × 1
  .pred_class
  <fct>      
1 No         
2 Yes        
3 Yes        
4 Yes        
5 Yes        
6 Yes        
```

``` r
confusionMatrix(testpredrf$.pred_class, test$Above_Median, 
                positive = "Yes")
```

```         
Confusion Matrix and Statistics

          Reference
Prediction  No Yes
       No  284  37
       Yes  19 276
                                          
               Accuracy : 0.9091          
                 95% CI : (0.8836, 0.9306)
    No Information Rate : 0.5081          
    P-Value [Acc > NIR] : <2e-16          
                                          
                  Kappa : 0.8183          
                                          
 Mcnemar's Test P-Value : 0.0231          
                                          
            Sensitivity : 0.8818          
            Specificity : 0.9373          
         Pos Pred Value : 0.9356          
         Neg Pred Value : 0.8847          
             Prevalence : 0.5081          
         Detection Rate : 0.4481          
   Detection Prevalence : 0.4789          
      Balanced Accuracy : 0.9095          
                                          
       'Positive' Class : Yes             
                                          
```

On the testing set, it has an accuracy of 0.9091, sensitivity of 0.8818,
and specificity of 0.9373.

``` r
saveRDS(ames_rf_fit, "ames_rf_fit.rds")
ames_rf_fit = readRDS("ames_rf_fit.rds")
ames_rf_fit %>% pull_workflow_fit() %>% vip(geom = "point")
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/saving,%20loading,%20visualizing%20the%20model%20for%20variable%20importance-1.png)

Our chart of variable importance was informative. It should be noted
that this chart does not discriminate between above or below median home
value sale.

## Conclusions

Overall, these four different approaches yielded acceptable models. We
can compare their performances in this table. The top performers on the
testing set are in bold.

+------------+------------+------------+------------+------------+
|            | L          | L          | Class      | Random     |
|            | ogarithmic | ogarithmic | ification/ | Forests    |
|            | Regression | Lasso      | Decision   |            |
|            |            | Regression | Tree       |            |
+============+============+============+============+============+
| Accuracy   | Train:     | Train:     | Train:     | Train:     |
|            | 0.946112   | 0.93528118 | 0.9068     | 0.9861     |
|            |            |            |            |            |
|            | Test:      | **Test:    | Test:      | Test:      |
|            | 0.8961039  | 0.913961** | 0.8847     | 0.9091     |
+------------+------------+------------+------------+------------+
| S          | Train:     | Train:     | Train:     | Train:     |
| ensitivity | 0.9383562  | 0.9315068  | 0.9041     | 0.9890     |
|            |            |            |            |            |
|            | Test:      | **Test: 0  | Test:      | Test:      |
|            | 0.9105431  | .9329073** | 0.8722     | 0.8818     |
+------------+------------+------------+------------+------------+
| S          | Train:     | Train:     | Train:     | Train:     |
| pecificity | 0.9561528  | 0.9405941  | 0.9095     | 0.9830     |
|            |            |            |            |            |
|            | Test:      | Test:      | Test:      | **Test:    |
|            | 0.8844884  | 0.8943894  | 0.8977     | 0.9373**   |
+------------+------------+------------+------------+------------+
| AUC        | Train:     | Train:     | n/a        | n/a        |
|            | 0.9895119  | 0.9853888  |            |            |
|            |            |            |            |            |
|            | Test:      | **Test: 0  |            |            |
|            | 0.9512121  | .9695378** |            |            |
+------------+------------+------------+------------+------------+

The logistic lasso regression had the best performance on the testing
set in accuracy, sensitivity, and AUC. The random forest had the best
performance on the testing set in specificity.

For ease of reference, I extracted the coefficient data from the
logistic lasso regression and plugged them into Excel. I removed the
excluded variables and ordered by coefficient strength both positively
and negatively. Recall the strength of a coefficient is determined by
how close its absolute value is to 1.

|                                          |                |
|------------------------------------------|----------------|
| **Variable**                             | **Coefficent** |
| Gr_Liv_Area                              | 1.332043       |
| Overall_Qual_Very_Good                   | 0.935834       |
| Garage_Cars                              | 0.66009        |
| Total_Bsmt_SF                            | 0.633479       |
| Full_Bath                                | 0.476946       |
| Overall_Qual_Good                        | 0.440551       |
| Foundation_PConc                         | 0.437148       |
| MS_Zoning_Floating_Village_Residential   | 0.416718       |
| (Intercept)                              | 0.328013       |
| Neighborhood_Crawford                    | 0.311684       |
| Year_Built                               | 0.263913       |
| Neighborhood_Clear_Creek                 | 0.235276       |
| Neighborhood_Timberland                  | 0.224135       |
| Sale_Condition_Alloca                    | 0.215948       |
| Kitchen_Qual_Excellent                   | 0.182781       |
| Kitchen_Qual_Good                        | 0.170933       |
| MS_SubClass_Split_or_Multilevel          | 0.14732        |
| Half_Bath                                | 0.122338       |
| MS_SubClass_Two_Story_1946_and_Newer     | 0.115926       |
| Neighborhood_Gilbert                     | 0.112638       |
| Neighborhood_Northwest_Ames              | 0.098113       |
| Neighborhood_Mitchell                    | 0.080147       |
| Exter_Cond_other                         | 0.079037       |
| Misc_Val                                 | 0.077807       |
| House_Style_SFoyer                       | 0.07774        |
| Lot_Area                                 | 0.076459       |
| House_Style_Two_and_Half_Unf             | 0.064444       |
| Garage_Type_BuiltIn                      | 0.04681        |
| Neighborhood_Brookside                   | 0.039986       |
| Garage_Type_Basment                      | 0.037168       |
| Neighborhood_Iowa_DOT_and_Rail_Road      | 0.032858       |
| MS_SubClass_One_Story_PUD_1946_and_Newer | 0.013008       |
| Exter_Cond_Good                          | 0.006206       |
| Year_Sold                                | -0.01111       |
| Foundation_Wood                          | -0.01618       |
| Neighborhood_Sawyer_West                 | -0.01686       |
| Foundation_Slab                          | -0.02186       |
| Lot_Shape_Regular                        | -0.08793       |
| Kitchen_Qual_other                       | -0.12375       |
| Exter_Cond_Fair                          | -0.14993       |
| Central_Air_N                            | -0.15413       |
| MS_Zoning_Residential_Medium_Density     | -0.17407       |
| Neighborhood_Old_Town                    | -0.19252       |
| Overall_Qual_Fair                        | -0.21114       |
| Garage_Type_CarPort                      | -0.21228       |
| Sale_Condition_Family                    | -0.22482       |
| Overall_Qual_Average                     | -0.25063       |
| Sale_Condition_Abnorml                   | -0.29358       |
| MS_SubClass_other                        | -0.32032       |
| Garage_Type_Detchd                       | -0.33815       |
| MS_SubClass_Two_Story_PUD_1946_and_Newer | -0.43284       |
| Overall_Qual_Below_Average               | -0.49515       |
| Kitchen_AbvGr                            | -0.77021       |

Let's also revisit our random forests variable importance and decision
tree.

``` r
saveRDS(ames_rf_fit, "ames_rf_fit.rds")
ames_rf_fit = readRDS("ames_rf_fit.rds")
ames_rf_fit %>% pull_workflow_fit() %>% vip(geom = "point")
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/tree%20and%20forest-1.png)

``` r
fancyRpartPlot(tree2, tweak = 1.25) 
```

![](Ames-Iowa-Above-Median-Predictions-Quarto.markdown_strict_files/figure-markdown_strict/tree%20and%20forest-2.png)

Considering these three models, for home investors in Ames, Iowa, I have
the following recommendations:

<u>Prioritize</u> investments in homes with the following features and
characteristics:

-   Larger ground living area

-   Newer builds

-   Large basements

-   Large/more-car garages

-   More full baths

-   Good overall quality

-   Concrete foundation

-   Floating Village Residential-zoned homes

-   Homes in the Crawford Neighborhood

<u>Avoid</u> investing in homes with the following features:

-   More than 1 kitchen

-   Detached or carport garages

-   Fair, average, or below average overall quality

-   Two story planned unit development homes

-   Family or abnormal sale conditions
