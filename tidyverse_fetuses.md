---
title: "Importing and Cleaning Data in the R tidyverse"
output:
  html_document:
    keep_md: TRUE
---



## Importing and Cleaning Data in the R tidyverse

R has existed for over 25 years, and is itself based on software that was invented in 1976.  Over that time, many different styles of R coding have evolved, and consequently there are a large number of different ways of accomplishing the same tasks.  The most modern way of using R is within what is called the tidyverse.  The tidyverse is a collection of extensions to base R that are focussed on making it as easy as possible to import, process and analyse data in R.  Central to this are two key concepts:  1) tidy data are easier to work with than messy data, and 2) R code should be as clear and easy to read as possible even for users that are not expert programmers.  R code written for the tidyverse looks a bit different to the code using base R that you might have used before, but once you get used to the differences it is much easier to create code that does relatively compex things with your data.

This document will explain some of the key features of importing, cleaning and checking data using the calf fetuses dataset as an example.  We will assume that you have installed both R (http://cran.r-project.org) and RStudio (http://rstudio.org) but you do not need to have any prior experience with R to follow the document.  First, open RStudio and create a new R script by clicking 'File' then 'New File' then 'R Script'.  Save it and give it a name, like 'tidy_fetuses.R'.  You should see a blank text file appear within R studio - you can then copy code given here in 'code chunks' like this one:


```r
2+4
```

...and paste them into your tidy_fetuses.R file.  To run the code, place the cursor somewhere on the line you want to run, and click 'Run' (or use the keyboard shortcut control/command + enter).  This document also displays the result of running these code chunks, which should match what you see when you run the code in R.  For example, the code above should give the following output:


```
## [1] 6
```

To run multiple lines at once, highlight them all and click 'Run'.  To run the entire file click 'Source'. 


### Installing tidyverse

You need to have the tidyverse package installed within R.  If you have not already installed the package, click 'Tools' then 'Install Packages' then type 'tidyverse' and click on 'Install'.  You only need to install a package once, but you need to load it every time you restart R.  Do this using the library() function:


```r
library('tidyverse')
```

```
## ── Attaching packages ───────────────────────────────────────────────────────────── tidyverse 1.2.1 ──
```

```
## ✔ ggplot2 3.1.0     ✔ purrr   0.2.5
## ✔ tibble  2.0.1     ✔ dplyr   0.7.8
## ✔ tidyr   0.8.2     ✔ stringr 1.3.1
## ✔ readr   1.3.1     ✔ forcats 0.3.0
```

```
## ── Conflicts ──────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
```

When you run this code, you should see similar output to that shown above.  If you have any problems then make sure that R and RStudio are updated to the latest versions.  If you still have problems, then let us know ASAP so we can try to resolve them (the most common problems are due to old operating system versions or restricted access on institutional computers - if you are using an admin account on a relatively up-to-date computer then you should be OK).


### Preparing your data

Before reading an Excel file into R, you will need to make sure that the relevant sheet is laid out in an appropriate way.  The first row should be column names, and all columns must have a unique name.  You will make life much easier for yourself if the column names are concise, and do not use spaces, special characters (e.g. + - / * & % € , etc), dates, or non-English characters (e.g. å æ ø).  You can use an underscore ( _ ) to create two-word names e.g. birth_date.

Each column can either be a number, date/time, or text - but this must be consistent within a column.  It is a good idea to ensure that date/time columns are stored correctly by selecting the column and going to 'Format' -> 'Cells' and selecting either date or time as the format.  As a general rule it is a much better idea to record categorical data (e.g. gender) as character rather than number codes e.g. male/female rather than 1/2.  Remember that all cell comments and formatting (e.g. bold, colours etc) will be completely ignored by R when reading the data, so DO NOT use any type of comment or format to represent important information: create a new column instead.

Finally, ensure that you do not have any stray non-blank cells within the data, or rows that do not form part of the data (e.g. formulas to calculate means as the last row) placed within the same sheet.  If you want to do this in Excel, you can create a second sheet specifically for any formulas / notes etc.  It is a good idea to have a separate 'data key' sheet where you record the meaning of the varibles, where the data has come from, and any other relevant meta-data such as the collection date or any other important notes that you might need to remember when you come back to the data later on.  This will also make it much easier for you to share the data with others.


### Reading data

The first thing you are going to want to do is to read your data file.  This can be in a number of formats, but the most common is an Excel file.  For this we need to load the readxl package (this is installed as part of tidyverse, but not loaded by default):


```r
library('readxl')
```

You will then need to make sure that your working directory is set correctly so that R can find the data file on your hard drive.  Go to the 'Session' menu, 'Set Working Directory', and 'Choose Directory' and then select the folder that contains the calf foetuses dataset that you downloaded from Absalon as part of a previous task.

You should now be able to load the data from a specified sheet within the Excel file using:


```r
fetuses <- read_excel('calf_fetuses.xlsx', sheet='calf_fetuses')
```

When you run this code, nothing much seems to happen (unless you get an error saying that the file could not be found, in which case check that you have included the .xlsx or .xlx file extension!).  But in fact R has read the data from the Excel file, and assigned the resulting 'data frame' (which is like an Excel worksheet) called 'fetuses' (using the assignment operator <-) in which the data is now being stored within your R envirnoment. If you look closely you should see the text 'fetuses   262 obs. of 18 variables' appear under 'Data' in the 'Environment' tab within R studio.  Click on the name 'fetuses' to open a new tab with the data in table form.


### Looking at your data

There are several methods of examining and visualising your data, but here we will stick with the tidyverse way of doing things.  You may already know how to do some of this in base R - it is OK to use methods you are already familiar with, but try not to mix different styles too much.  That leads to messy code, and we want everything to be tidy...

The tidyverse revolves around data frames - these allow us to keep all related data together, in a consistent format where we know that all columns have the same number of rows and each row corresponds to a specific set of observations.  When working with data in the tidyverse, we will therefore nearly always be working within these data frames.  To do something within a data frame, we first type the name of the data frame, then the special 'pipe' or 'chaining' operator %>%, and then a function that we want to apply to the data frame.  For example:


```r
fetuses %>% nrow()
```

```
## [1] 262
```

This takes the data frame fetuses, and applies the function nrow() to the data - the output is the number of rows of the dataset.  In base R we would achieve the same thing using:


```r
nrow(fetuses)
```

```
## [1] 262
```

These two bits of code are in fact equivalent - the %>% operator automatically takes the data frame input from the left hand side and passes it (as the first argument) to the function on the right hand side.  The advantage to using the %>% operator is that we can build up a more complex chain of commands more easily - you will see this later on.  There is even a handy keyboard shortcut to type the operator: command/control + shift + m.

A useful function to find out even more about the structure of the data is str() - this tells us how R is storing each of the columns (called 'variables') within the data frame:


```r
fetuses %>% str()
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	262 obs. of  18 variables:
##  $ parity            : num  3 2 1 1 2 3 2 2 6 4 ...
##  $ age_days          : num  274 196 259 249 221 221 177 82 193 142 ...
##  $ weight_kg         : num  35.5 10.2 27.1 29.1 16.6 ...
##  $ crl_cm            : num  86.8 61.8 89.3 83 75.4 77.7 55.4 10.4 62.4 35.4 ...
##  $ head_width_mm     : num  128 83.8 121 118 100.6 ...
##  $ head_length_mm    : num  239 154 220 215 183 ...
##  $ hair_coronary_band: chr  "Y" "Y" "Y" "Y" ...
##  $ hair_ear          : chr  "Y" "Y" "Y" "Y" ...
##  $ hair_eyelid       : chr  "Y" "Y" "Y" "Y" ...
##  $ hair_tail         : chr  "Y" "Y" "Y" "Y" ...
##  $ hair_hornbud      : chr  "Y" "Y" "Y" "Y" ...
##  $ tactile_muzzle    : chr  "Visible" "Visible" "Visible" "Visible" ...
##  $ tactile_eyebrow   : chr  "Visible" "Visible" "Visible" "Visible" ...
##  $ tactile_eyelash   : chr  "Visible" "Visible" "Visible" "Visible" ...
##  $ eye_op_close      : chr  "Open" "Open" "Open" "Open" ...
##  $ papillae_tongue   : chr  "Whole tongue" "Whole tongue" "Whole tongue" "Whole tongue" ...
##  $ eyelid            : chr  "Y" "Y" "Y" "Y" ...
##  $ sex               : chr  "Female" "Female" "Male" "Male" ...
```

This gives several bits of information:  how many rows (262) and columns (18) are in the data, what each of these columns are called (parity, age_days and so on), how these variables are being stored (we only have num = numeric/number, and chr = character/text in this data frame), and the values stored in the first few rows for each variable.  We can get similar information by just typing the name of the data frame:


```r
fetuses
```

```
## # A tibble: 262 x 18
##    parity age_days weight_kg crl_cm head_width_mm head_length_mm hair_coronary_b… hair_ear
##     <dbl>    <dbl>     <dbl>  <dbl>         <dbl>          <dbl> <chr>            <chr>   
##  1      3      274    35.5     86.8         128            239   Y                Y       
##  2      2      196    10.2     61.8          83.8          154   Y                Y       
##  3      1      259    27.2     89.3         121            220.  Y                Y       
##  4      1      249    29.1     83           118            215   Y                Y       
##  5      2      221    16.6     75.4         101.           183   Y                Y       
##  6      3      221    19.0     77.7          97            189   Y                Y       
##  7      2      177     6.35    55.4          74.5          142.  Y                N       
##  8      2       82     0.046   10.4          24.3           36.8 N                N       
##  9      6      193    11.2     62.4          85.4          153.  Y                N       
## 10      4      142     1.77    35.4          62.4           99.8 N                N       
## # … with 252 more rows, and 10 more variables: hair_eyelid <chr>, hair_tail <chr>,
## #   hair_hornbud <chr>, tactile_muzzle <chr>, tactile_eyebrow <chr>, tactile_eyelash <chr>,
## #   eye_op_close <chr>, papillae_tongue <chr>, eyelid <chr>, sex <chr>
```

But this just shows you the first few variables, so it is usually better to use the str() function.

Finally, we can use the summary() function to see summary statistics for each variable:


```r
fetuses %>% summary()
```

```
##      parity         age_days        weight_kg           crl_cm       head_width_mm   
##  Min.   :1.000   Min.   : 25.00   Min.   : 0.0010   Min.   :  2.00   Min.   :  3.00  
##  1st Qu.:2.000   1st Qu.: 95.25   1st Qu.: 0.2537   1st Qu.: 18.30   1st Qu.: 39.24  
##  Median :3.000   Median :136.00   Median : 1.7805   Median : 35.65   Median : 63.33  
##  Mean   :2.752   Mean   :136.06   Mean   : 5.0029   Mean   : 37.90   Mean   : 62.34  
##  3rd Qu.:4.000   3rd Qu.:175.00   3rd Qu.: 6.9250   3rd Qu.: 54.60   3rd Qu.: 84.02  
##  Max.   :6.000   Max.   :274.00   Max.   :41.6000   Max.   :101.30   Max.   :152.00  
##  head_length_mm   hair_coronary_band   hair_ear         hair_eyelid         hair_tail        
##  Min.   :  5.00   Length:262         Length:262         Length:262         Length:262        
##  1st Qu.: 55.52   Class :character   Class :character   Class :character   Class :character  
##  Median :101.28   Mode  :character   Mode  :character   Mode  :character   Mode  :character  
##  Mean   :103.29                                                                              
##  3rd Qu.:145.54                                                                              
##  Max.   :239.00                                                                              
##  hair_hornbud       tactile_muzzle     tactile_eyebrow    tactile_eyelash    eye_op_close      
##  Length:262         Length:262         Length:262         Length:262         Length:262        
##  Class :character   Class :character   Class :character   Class :character   Class :character  
##  Mode  :character   Mode  :character   Mode  :character   Mode  :character   Mode  :character  
##                                                                                                
##                                                                                                
##                                                                                                
##  papillae_tongue       eyelid              sex           
##  Length:262         Length:262         Length:262        
##  Class :character   Class :character   Class :character  
##  Mode  :character   Mode  :character   Mode  :character  
##                                                          
##                                                          
## 
```

For the numeric variables, we get the range (min and max), as well as median/mean and 1st and 3rd quantiles of the variable.  This is useful to check for extreme values that probably indicate an error.  For the text variables, we don't get any useful information: this is because there is no general way to summarise text variables.  We need to convert them into something else...


### Creating new variables

When reading Excel files, R will (usually) store numbers, dates and date/time variables into the correct format automatically (as long as these variables are formatted correctly in Excel - see above).  But it has no way of correctly formatting our categorical variables, so it leaves these as simple text entries - we need to format them manually within R into 'factor' variables that better represent our categorical variables.  That means we need to alter the data frame within R.  There are several ways to do this, but here we will stick with the tidyverse approach.

The function mutate() allows us to create a new variable based on one or more existing variables.  For example, the following code creates a new variable called crl_mm based on crl_cm:


```r
fetuses %>%
	mutate(crl_mm = crl_cm*10)
```

```
## # A tibble: 262 x 19
##    parity age_days weight_kg crl_cm head_width_mm head_length_mm hair_coronary_b… hair_ear
##     <dbl>    <dbl>     <dbl>  <dbl>         <dbl>          <dbl> <chr>            <chr>   
##  1      3      274    35.5     86.8         128            239   Y                Y       
##  2      2      196    10.2     61.8          83.8          154   Y                Y       
##  3      1      259    27.2     89.3         121            220.  Y                Y       
##  4      1      249    29.1     83           118            215   Y                Y       
##  5      2      221    16.6     75.4         101.           183   Y                Y       
##  6      3      221    19.0     77.7          97            189   Y                Y       
##  7      2      177     6.35    55.4          74.5          142.  Y                N       
##  8      2       82     0.046   10.4          24.3           36.8 N                N       
##  9      6      193    11.2     62.4          85.4          153.  Y                N       
## 10      4      142     1.77    35.4          62.4           99.8 N                N       
## # … with 252 more rows, and 11 more variables: hair_eyelid <chr>, hair_tail <chr>,
## #   hair_hornbud <chr>, tactile_muzzle <chr>, tactile_eyebrow <chr>, tactile_eyelash <chr>,
## #   eye_op_close <chr>, papillae_tongue <chr>, eyelid <chr>, sex <chr>, crl_mm <dbl>
```

There are three things to note here:
- I entered a new line and tab between the chaining operator %>% and the function I am using: this is just to make the code easier to read by separating each part of the code onto a new line rather than having a single, very long line of code
- This is a very simple mutate where we create a new variable called crl_mm and calculate its values by multiplying the existing variable crl_cm by 10.  This creates a new column within the data frame because we don't already have a variable called 'crl_mm' - if you reuse an existing variable name (e.g. crl_mm) then this will replace the previous column within the data frame.
- The mutate function creates a new data frame, but we have not assigned it to anything - so R just shows us the new data frame (which is identical to the old one except for having 19 rather than 18 variables) and then immediately forgets it.

To get R to remember the changes we are making to the data frame we need to assign it to something using the assignment operator.  For example, this code creates a new data frame called fetuses_full:


```r
fetuses_full <- fetuses %>%
	mutate(crl_mm = crl_cm*10)
```

You should now see a new data frame called fetuses_full appear in your R environment as '262 obs. of 19 variables'.  To over-write the existing data frame, we can just assign the new data frame to have the same name as the old one:


```r
fetuses <- fetuses %>%
	mutate(crl_mm = crl_cm*10)

fetuses %>% 
	str()
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	262 obs. of  19 variables:
##  $ parity            : num  3 2 1 1 2 3 2 2 6 4 ...
##  $ age_days          : num  274 196 259 249 221 221 177 82 193 142 ...
##  $ weight_kg         : num  35.5 10.2 27.1 29.1 16.6 ...
##  $ crl_cm            : num  86.8 61.8 89.3 83 75.4 77.7 55.4 10.4 62.4 35.4 ...
##  $ head_width_mm     : num  128 83.8 121 118 100.6 ...
##  $ head_length_mm    : num  239 154 220 215 183 ...
##  $ hair_coronary_band: chr  "Y" "Y" "Y" "Y" ...
##  $ hair_ear          : chr  "Y" "Y" "Y" "Y" ...
##  $ hair_eyelid       : chr  "Y" "Y" "Y" "Y" ...
##  $ hair_tail         : chr  "Y" "Y" "Y" "Y" ...
##  $ hair_hornbud      : chr  "Y" "Y" "Y" "Y" ...
##  $ tactile_muzzle    : chr  "Visible" "Visible" "Visible" "Visible" ...
##  $ tactile_eyebrow   : chr  "Visible" "Visible" "Visible" "Visible" ...
##  $ tactile_eyelash   : chr  "Visible" "Visible" "Visible" "Visible" ...
##  $ eye_op_close      : chr  "Open" "Open" "Open" "Open" ...
##  $ papillae_tongue   : chr  "Whole tongue" "Whole tongue" "Whole tongue" "Whole tongue" ...
##  $ eyelid            : chr  "Y" "Y" "Y" "Y" ...
##  $ sex               : chr  "Female" "Female" "Male" "Male" ...
##  $ crl_mm            : num  868 618 893 830 754 777 554 104 624 354 ...
```

The first line of this code is a very common pattern that we will use repeatedly - it tells R to save a data frame called 'fetuses' which is based on an existing data frame called 'fetuses' but with some function(s) (given on additional lines) applied to it using the chaining operator.


### Selecting and filtering columns or rows

Sometimes we might be reading in a large Excel dataset (with many columns and/or rows), but we don't want to work with the whole dataset.  For example, let's say we are only interested in the parity, age and hair_coronary_band variables.  Rather than working on the large dataset, it might be a good idea to create a second data frame with a more restricted dataset - this will be easier (and faster) to work with, but does not remove any information from the original dataset that we have already copied to fetuses_full.  We can select columns from fetuses (and over-write this data frame) using the select() function:


```r
fetuses <- fetuses %>%
	select(parity, age_days, hair_coronary_band)
```

You should now see that fetuses is 262 obs. or 3 variables (but fetuses_full is still 19 variables so we still have the original data).  This makes it easier to look only at what we want:


```r
fetuses %>%
	summary()
```

```
##      parity         age_days      hair_coronary_band
##  Min.   :1.000   Min.   : 25.00   Length:262        
##  1st Qu.:2.000   1st Qu.: 95.25   Class :character  
##  Median :3.000   Median :136.00   Mode  :character  
##  Mean   :2.752   Mean   :136.06                     
##  3rd Qu.:4.000   3rd Qu.:175.00                     
##  Max.   :6.000   Max.   :274.00
```

We might even go further, and want to exclude the very early fetuses from the dataset we are working with.  The filter() function extracts only rows that meet one or more conditions, e.g.:


```r
fetuses <- fetuses %>%
	filter(age_days >= 50)
```

Now we have only the 243 obs. from fetuses with a gestational age of 50 days or more.

The great thing about the chaining operator is that it is easy to do multiple functions in a row without stopping in between, for example the two bits of R code above is the same as the following single piece of R code:


```r
fetuses <- fetuses_full %>%
	select(parity, age_days, hair_coronary_band) %>%
	filter(age_days >= 50)
```

Note that we can add as many functions to the chain as we want to, as long as we remember to put %>% at the end of every function (except the last one that terminates the pipe).  Let's add a third function near the top of this chain:  we can use mutate to create a new variable that tells us the row number from the original data that each row in the filtered data corresponds to (just in case we need it later):


```r
fetuses <- fetuses_full %>%
	mutate(ID = row_number()) %>%
	select(ID, parity, age_days, hair_coronary_band) %>%
	filter(age_days >= 50)
```

You can see now why it is a good idea to put each function on a new line - otherwise the code would quickly get hard to read!  As long as the %>% operator comes at the end of the line (rather than at the start of the new line), then it all works as expected.  Remember that all the code within the chain (even if on multiple lines) is a single R command, so it won't work if you try to just run the last line (for example).  Fortunately, RStudio is smart enough to know this and will run the entire command if the curser is anywhere on any of the lines within the whole chain of functions.


### Converting text variables into factors

When reading Excel files, R will (usually) store numbers, dates and date/time variables into the correct format automatically (as long as these variables are formatted correctly in Excel - see above).  But it has no way of correctly formatting our categorical variables, so it leaves these as simple text entries - we need to format them manually within R into 'factor' variables that better represent our categorical variables.  There are several ways to do this, but here we will stick with the tidyverse.

We will use 'parse_factor' within a mutate function to convert a character/text variable to a factor.  For example:


```r
fetuses <- fetuses %>%
	mutate(hair_coronary_band_fct = parse_factor(hair_coronary_band))
```

This creates a new variable by turning the text variable hair_coronary_band into a factor.  We can now summarise this more usefully:


```r
fetuses %>%
	summary()
```

```
##        ID            parity         age_days     hair_coronary_band hair_coronary_band_fct
##  Min.   :  1.0   Min.   :1.000   Min.   : 51.0   Length:243         Y: 60                 
##  1st Qu.: 61.5   1st Qu.:2.000   1st Qu.:107.5   Class :character   N:183                 
##  Median :122.0   Median :3.000   Median :144.0   Mode  :character                         
##  Mean   :125.6   Mean   :2.774   Mean   :143.3                                            
##  3rd Qu.:188.5   3rd Qu.:4.000   3rd Qu.:176.5                                            
##  Max.   :262.0   Max.   :6.000   Max.   :274.0
```

Now we get the number of Y and the number of N for hair_coronary_band_fct.  In this case, that works OK - but what would happen if the data had been entered inconsistently, for example if 'n' had been used instead of 'N'?  Let's deliberately introduce an error (don't worry about the code itself - you won't do this in real life!):


```r
fetuses$hair_coronary_band[1] <- 'n'
```

We can see the first observation in hair_coronary_band is now 'n':


```r
fetuses %>% str()
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	243 obs. of  5 variables:
##  $ ID                    : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ parity                : num  3 2 1 1 2 3 2 2 6 4 ...
##  $ age_days              : num  274 196 259 249 221 221 177 82 193 142 ...
##  $ hair_coronary_band    : chr  "n" "Y" "Y" "Y" ...
##  $ hair_coronary_band_fct: Factor w/ 2 levels "Y","N": 1 1 1 1 1 1 1 2 1 2 ...
```

What happens when this is automatically turned into a factor?


```r
fetuses <- fetuses %>%
	mutate(hair_coronary_band_fct = parse_factor(hair_coronary_band))

fetuses %>%
	summary()
```

```
##        ID            parity         age_days     hair_coronary_band hair_coronary_band_fct
##  Min.   :  1.0   Min.   :1.000   Min.   : 51.0   Length:243         n:  1                 
##  1st Qu.: 61.5   1st Qu.:2.000   1st Qu.:107.5   Class :character   Y: 59                 
##  Median :122.0   Median :3.000   Median :144.0   Mode  :character   N:183                 
##  Mean   :125.6   Mean   :2.774   Mean   :143.3                                            
##  3rd Qu.:188.5   3rd Qu.:4.000   3rd Qu.:176.5                                            
##  Max.   :262.0   Max.   :6.000   Max.   :274.0
```

There are now 3 factor levels in hair_coronary_band_fct:  Y, N and n.  Note that N and n are different, but we don't really want them to be!  We can deal with this by first creating the factor as before, and chaining the data frame into a second mutate where we use fct_collapse to manually recode the factor levels so that Y becomes Yes and both n and N become No, like so:


```r
fetuses <- fetuses %>%
	mutate(hair_coronary_band_fct = parse_factor(hair_coronary_band)) %>%
	mutate(hair_coronary_band_fct = fct_collapse(hair_coronary_band_fct, No = c('N', 'n'), Yes='Y'))

fetuses %>%
	summary()
```

```
##        ID            parity         age_days     hair_coronary_band hair_coronary_band_fct
##  Min.   :  1.0   Min.   :1.000   Min.   : 51.0   Length:243         No :184               
##  1st Qu.: 61.5   1st Qu.:2.000   1st Qu.:107.5   Class :character   Yes: 59               
##  Median :122.0   Median :3.000   Median :144.0   Mode  :character                         
##  Mean   :125.6   Mean   :2.774   Mean   :143.3                                            
##  3rd Qu.:188.5   3rd Qu.:4.000   3rd Qu.:176.5                                            
##  Max.   :262.0   Max.   :6.000   Max.   :274.0
```

Notice also that the factor levels have swapped around; No is now the reference category because this came first in the arguments to the fct_collapse function.

In the real world, we won't want to spend time examining all of the factors to make sure that there aren't any typos - particularly when we think the data should be correct to start with!  This can lead to errors, because we might not check as carefully as we should and therefore miss problems with the data.  So it is much better practice to tell parse_factor what to expect by using the levels argument:


```r
fetuses <- fetuses %>%
	mutate(hair_coronary_band_fct = parse_factor(hair_coronary_band, levels=c('N','Y')))
```

```
## Warning: 1 parsing failure.
## row col           expected actual
##   1  -- value in level set      n
```

When we run this code, we get a warning (and some red text) explaining that an unexpected text entry 'n' was observed in row 1, and the resulting variable now has 1 missing value (NA):


```r
fetuses %>%
	summary()
```

```
##        ID            parity         age_days     hair_coronary_band hair_coronary_band_fct
##  Min.   :  1.0   Min.   :1.000   Min.   : 51.0   Length:243         N   :183              
##  1st Qu.: 61.5   1st Qu.:2.000   1st Qu.:107.5   Class :character   Y   : 59              
##  Median :122.0   Median :3.000   Median :144.0   Mode  :character   NA's:  1              
##  Mean   :125.6   Mean   :2.774   Mean   :143.3                                            
##  3rd Qu.:188.5   3rd Qu.:4.000   3rd Qu.:176.5                                            
##  Max.   :262.0   Max.   :6.000   Max.   :274.0
```

This is good because R has automatically alerted us to a problem that we need to fix.  So we can go back and adjust our code so it looks like this:


```r
fetuses <- fetuses %>%
	mutate(hair_coronary_band_fct = parse_factor(hair_coronary_band, levels=c('N','n','Y'))) %>%
	mutate(hair_coronary_band_fct = fct_collapse(hair_coronary_band_fct, No = c('N', 'n'), Yes='Y'))
```

This tells R to expect the 'n' entry, but then immediately re-codes it afterwards - this approach will fix known problems in the data, but still automatically alert us to any new problems that might occur if e.g. we update the data to include more observations (and more mistakes!).  

There is one additional thing to remember with parse_factor - if there were observations in the original data that were missing before making them into a factor, these will be treated differently to observations that were present but corresponded to unknown factor levels.  For example, let's deliberately make the second observation of hair_coronary_band blank (i.e. missing) and then re-run the code with only N and Y as specified levels:


```r
fetuses$hair_coronary_band[2] <- ''

fetuses <- fetuses %>%
	mutate(hair_coronary_band_fct = parse_factor(hair_coronary_band, levels=c('N','Y')))
```

```
## Warning: 1 parsing failure.
## row col           expected actual
##   1  -- value in level set      n
```

We get the red text again (telling us about the 'n' on row 1), but no mention of the missing entry on row 2.  This is because parse_factor has created an explicit factor level for missing data:


```r
fetuses %>%
	summary()
```

```
##        ID            parity         age_days     hair_coronary_band hair_coronary_band_fct
##  Min.   :  1.0   Min.   :1.000   Min.   : 51.0   Length:243         N   :183              
##  1st Qu.: 61.5   1st Qu.:2.000   1st Qu.:107.5   Class :character   Y   : 58              
##  Median :122.0   Median :3.000   Median :144.0   Mode  :character   NA  :  1              
##  Mean   :125.6   Mean   :2.774   Mean   :143.3                      NA's:  1              
##  3rd Qu.:188.5   3rd Qu.:4.000   3rd Qu.:176.5                                            
##  Max.   :262.0   Max.   :6.000   Max.   :274.0
```

We have both the factor level NA (the value that was missing to start with) and NA's (the value of 'n' that became missing because it wasn't included in the allowed levels).  But this subtle distinction is not visible in the data itself:


```r
fetuses
```

```
## # A tibble: 243 x 5
##       ID parity age_days hair_coronary_band hair_coronary_band_fct
##    <int>  <dbl>    <dbl> <chr>              <fct>                 
##  1     1      3      274 n                  <NA>                  
##  2     2      2      196 ""                 <NA>                  
##  3     3      1      259 Y                  Y                     
##  4     4      1      249 Y                  Y                     
##  5     5      2      221 Y                  Y                     
##  6     6      3      221 Y                  Y                     
##  7     7      2      177 Y                  Y                     
##  8     8      2       82 N                  N                     
##  9     9      6      193 Y                  Y                     
## 10    10      4      142 N                  N                     
## # … with 233 more rows
```

Having this distinction can sometimes be useful, but if we want to remove the NA factor level then we can specify the include_na = FALSE argument e.g.:


```r
fetuses <- fetuses %>%
	mutate(hair_coronary_band_fct = parse_factor(hair_coronary_band, levels=c('N','Y'), include_na = FALSE))
```

```
## Warning: 1 parsing failure.
## row col           expected actual
##   1  -- value in level set      n
```

```r
fetuses %>%
	summary()
```

```
##        ID            parity         age_days     hair_coronary_band hair_coronary_band_fct
##  Min.   :  1.0   Min.   :1.000   Min.   : 51.0   Length:243         N   :183              
##  1st Qu.: 61.5   1st Qu.:2.000   1st Qu.:107.5   Class :character   Y   : 58              
##  Median :122.0   Median :3.000   Median :144.0   Mode  :character   NA's:  2              
##  Mean   :125.6   Mean   :2.774   Mean   :143.3                                            
##  3rd Qu.:188.5   3rd Qu.:4.000   3rd Qu.:176.5                                            
##  Max.   :262.0   Max.   :6.000   Max.   :274.0
```

Now there is no distinction between the blank entry and the entry that was originally 'n' - they are both NA's.  Alternatively, you can remove the NA factor level (or any other factor level that you want to) by setting it to NULL within fct_collapse:


```r
fetuses <- fetuses %>%
	mutate(hair_coronary_band = parse_factor(hair_coronary_band, levels=c('N','n','Y'))) %>%
	mutate(hair_coronary_band = fct_collapse(hair_coronary_band, NULL = 'NA', No = c('N', 'n'), Yes='Y'))

fetuses %>%
	summary()
```

```
##        ID            parity         age_days     hair_coronary_band hair_coronary_band_fct
##  Min.   :  1.0   Min.   :1.000   Min.   : 51.0   No  :184           N   :183              
##  1st Qu.: 61.5   1st Qu.:2.000   1st Qu.:107.5   Yes : 58           Y   : 58              
##  Median :122.0   Median :3.000   Median :144.0   NA's:  1           NA's:  2              
##  Mean   :125.6   Mean   :2.774   Mean   :143.3                                            
##  3rd Qu.:188.5   3rd Qu.:4.000   3rd Qu.:176.5                                            
##  Max.   :262.0   Max.   :6.000   Max.   :274.0
```

Notice this time that I have replaced the existing variable hair_coronary_band rather than creating a new variable - generally we will want to do this when creating factors, as the original text entries are no longer needed.  The only difference between these factors is that 'n' was made into 'No' in hair_coronary_band, but left as missing in hair_coronary_band_fct.  But we don't really need hair_coronary_band_fct so it is best to remove it - we can do this using select and a minus symbol in front of the variable name to indicate it is to be removed rather than retained:


```r
fetuses <- fetuses %>%
	select(-hair_coronary_band_fct)

fetuses %>%
	str()
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	243 obs. of  4 variables:
##  $ ID                : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ parity            : num  3 2 1 1 2 3 2 2 6 4 ...
##  $ age_days          : num  274 196 259 249 221 221 177 82 193 142 ...
##  $ hair_coronary_band: Factor w/ 2 levels "No","Yes": 1 NA 2 2 2 2 2 1 2 1 ...
```

Now we have 4 variables: ID, parity, age_days and hair_coronary_band as a factor with No/Yes recordings.


### Converting numbers into factors

as.character if coded as numbers but often better to code as text
Add parity to front of number

ID adding Obs_ to front of number

cut age_days

grouping parity

fetuses %>%
	mutate(ID = parse_factor(str_c('Row_', ID)))
	
OR:

fetuses %>%
	mutate(ID = parse_factor(str_c(ID))) %>%
	mutate(ID = fct_relabel(ID, ~ str_c('Row_', .)))



### More variable types

?parse_date

lubridate::ymd etc

parse_numeric for text to numbers
¨
ordinal

numeric vs integer

logical



### Converting multiple columns at once


### Tidy data:  hair-level obs



### Additional notes

tidyverse

cheat sheets for reading data

Danish encoding

Copy and paste code

When googling include tidyverse word

Use github
