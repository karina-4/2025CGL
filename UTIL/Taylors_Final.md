Taylors_Law_Final_Knit
================
Soren Maret
2025-04-24

Chapter_01

\##GOAL##: Do a proof of concept that their is a scaling (power law)
relationship in the gene expression pattern of dox exposed RNA-seq data.

Specifically I will investigate if Taylors Law, an imperical law from
ecology, which explains the scaling relationship between the mean and
standard deviation presents in our data.

Taylors law is expressed in the following way:

sd = b \* mean^a

This related the standard deviation (stdev) to the mean by 2 constants b
and a where a is our scaling value and b is a normaling constant

We can rearrange this to using logarithms to get these values by
themselves. to do this, since our bases are not equal, we will use the
natural logarithm (ln):

ln(sd) = a \* ln(mean) + ln(b)

Notice how this is the equation of a line. we can derive the values for
a (slope) and b (y-intercept), which are the biologically informative
ones, by running a linear regression with high enough statistical
signifigance (measured by the regression constant R)

for further reading see the resources below:
<https://necsi.edu/power-law>
<https://www.nature.com/articles/s41562-016-0012>
<https://journals.aps.org/prx/abstract/10.1103/PhysRevX.12.011051#>:~:text=Popular%20Summary,varies%20from%20cell%20to%20cell.

In our case we will examine the mean expression of a gene compaired to
its standard deviation. to do this we will use our counts matrix
produced from our nextflow run.

Lets load in some data shall we? To do a proof of concept I will run
both human and mouse data in parallel.

``` r
Human_counts <- read_tsv("/scratch/Shares/rinnclass/MASTER_CLASS/STUDENTS/genehomies/DATA/HUMAN/salmon.merged.gene_counts.tsv")
```

    ## Rows: 60649 Columns: 41
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: "\t"
    ## chr  (2): gene_id, gene_name
    ## dbl (39): gfp_0_1, gfp_0_2, gfp_0_3, gfp_0_4, gfp_0_5, gfp_0.5_1, gfp_0.5_2,...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
Mouse_counts <- read_tsv("/scratch/Shares/rinnclass/MASTER_CLASS/STUDENTS/genehomies/DATA/MOUSE/salmon.merged.gene_counts.tsv")
```

    ## Rows: 55401 Columns: 17
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: "\t"
    ## chr  (2): gene_id, gene_name
    ## dbl (15): WT_0_1, WT_0_2, WT_0_3, WT_12_1, WT_12_2, WT_12_3, WT_24_1, WT_24_...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#cool! now lets remove the logical values to our results dataframe which we ill call taylor_values_human or taylor_values_mouse

#lets do this for humans:
#making a dataframe that contains our "gene_id" collumn:
taylor_values_human <- Human_counts["gene_id"] 
#now lets move gene_names over as well
taylor_values_human <- cbind(taylor_values_human, Human_counts[c("gene_name")])
#looks just like our g2s.df from earlier this semester!

#now for the mice: 
taylor_values_mouse <- Mouse_counts["gene_id"] 
#now lets move gene_names over as well
taylor_values_mouse <- cbind(taylor_values_mouse, Mouse_counts[c("gene_name")])
#great! this all worked out nicely!
```

Now we have our new dataframes which we can load all of our means and
standard deviations into so we can later run analysis. in order to
successfully do these calculations we need to remove the charaters and
make our dataframes numeric.We can do this in any order.

``` r
#first, lets remove the collumns gene_name and gene_id as they are characters and will just become N/As
#removes gene_id
Human_counts <- Human_counts[, !(names(Human_counts)) %in% "gene_id"]
#removes gene_name
Human_counts <- Human_counts[, !(names(Human_counts)) %in% "gene_name"]
#makes out counts matrix numeric
Human_counts <- data.frame(lapply(Human_counts, as.numeric))

#we will do the same for our mouse data:
Mouse_counts <- Mouse_counts[, !(names(Mouse_counts)) %in% "gene_id"]
#removes gene_name
Mouse_counts <- Mouse_counts[, !(names(Mouse_counts)) %in% "gene_name"]
#makes out counts matrix numeric
Mouse_counts <- data.frame(lapply(Mouse_counts, as.numeric))
```

Great! all is well with the cosmos. Now Taylors Law looks at 2 things an
independent variable: the mean of a gene (in our case, often its a taxa)
and an indepdent variable: the standard deviation of that gene. so we
need to calculate these values as values in our enviornment.

``` r
#here is the anaylsis for the human counts: 
means_human <- rowMeans(Human_counts)
#now for the standard deviation accross the rows: 
sd_human <- apply(Human_counts, 1, sd)

#now we can do the same for our mouse data
means_mouse <- rowMeans(Mouse_counts)
#now for the standard deviation accross the rows: 
sd_mouse <- apply(Mouse_counts, 1, sd)

#Sweet!! now lets plot this just to see whats up?
plot(means_human, sd_human)
```

![](Taylors_Final_files/figure-gfm/taking%20the%20means%20and%20standard%20deviations%20+%20plotting-1.png)<!-- -->

``` r
plot(means_mouse, sd_mouse)
```

![](Taylors_Final_files/figure-gfm/taking%20the%20means%20and%20standard%20deviations%20+%20plotting-2.png)<!-- -->

``` r
#cool! our data is linear but the values varie massively! but on the bright side this is a proof of concept!!!
```

\##RESULT##: 1. our RNA-seq data is explained by taylors law. 2. most of
the genes are expressed at a low level, with fewer and fewer genes being
expressed at very high levels (orders of magnitude more) 3. the human
and mouse data have different slopes (values of coefficient “a”.

Now lets move onto making this all analyzable in chapter 02: but first,
some housekeeping:

``` r
taylor_values_human <- cbind(taylor_values_human, mean = means_human, sd = sd_human)

taylor_values_mouse <- cbind(taylor_values_mouse, mean = means_mouse, sd = sd_mouse)

save(Human_counts, Mouse_counts, taylor_values_human, taylor_values_mouse, means_human, means_mouse, sd_human, sd_mouse, file = "/scratch/Shares/rinnclass/MASTER_CLASS/STUDENTS/genehomies/RESULTS/TPL_ANALYSIS/RData/Chapter_01.RData")
#cleaning
rm(list = ls())
```

Chapter_02

\##GOAL##: Do all realivent transformations to our data such as log
transformations and sort out all genes that didn’t change or returned a
0.0000 for their mean.

``` r
load("/scratch/Shares/rinnclass/MASTER_CLASS/STUDENTS/genehomies/RESULTS/TPL_ANALYSIS/RData/Chapter_01.RData")
##our envionrment should be identical to where we left of in chapter_01
```

Great! recall how in our plots that we generated in chapter 01 were
super dense close to zero and thinned out as we went away from the
x-axis? This indicates that few genes are very highly expressed.
precisely, if there were 10 genes in our data, then 50% of our
expression would be from one gene.

To get around this we need to log transform our data using the natural
logarithm to get our slope and intercept values described in chapter 01.

``` r
#we will create value lists that we will merge into our taylor_values dataframes later
#in R the log function is the natural log, how lovely!
#heres the log_mean:
log_m_human <- log(means_human)
#heres the log_sd:
log_sd_human <- log(sd_human)

#Now we can just repeate this for mice
#heres the log_mean:
log_m_mouse <- log(means_mouse)
#heres the log_sd:
log_sd_mouse <- log(sd_mouse)
```

Great, we could do the gut check plotting we did earlier or we could
just read eveything into our taylot_values dataframes just to make life
easy

``` r
#for humans
taylor_values_human <- taylor_values_human %>%
  mutate(log_m = log_m_human)
taylor_values_human <- taylor_values_human %>%
  mutate(log_sd = log_sd_human)
#for mice
taylor_values_mouse <- taylor_values_mouse %>%
  mutate(log_m = log_m_mouse)
taylor_values_mouse <- taylor_values_mouse %>%
  mutate(log_sd = log_sd_mouse)
```

Note how some rows have returned a inf or -inf value for their results.
This is fine as ggplot just ignore these, but we should remove it. To do
this we will just remove all rows in our mean or standard deviation
collums that are zero. While we are at it lets rename our dataframes
after the sorting.

``` r
#humans first!
taylor_values_human_02 <- taylor_values_human[taylor_values_human$mean != 0, ]
taylor_values_human_02 <- taylor_values_human[taylor_values_human$sd != 0, ]
#now mice!
taylor_values_mouse_02 <- taylor_values_mouse[taylor_values_mouse$mean != 0, ]
taylor_values_mouse_02 <- taylor_values_mouse[taylor_values_mouse$sd != 0, ]
```

That got rid of a bunch of our data. now we can run our regressions. For
this we will use ggplot2 in chapter_03!

``` r
save(taylor_values_human_02, taylor_values_mouse_02, file = "/scratch/Shares/rinnclass/MASTER_CLASS/STUDENTS/genehomies/RESULTS/TPL_ANALYSIS/RData/Chapter_02.RData")

rm(list = ls())
```

Chapter_03

\##GOAL##: We Will now run regressions on our log transformed data to
get our parameters “a” and “b”. This will be our main result, but then I
will do additional analysis that i will expand upon for our final
project!

``` r
load("/scratch/Shares/rinnclass/MASTER_CLASS/STUDENTS/genehomies/RESULTS/TPL_ANALYSIS/RData/Chapter_02.RData")
```

Cool lets get down to it. I will slowly build up our analysis using
ggplot2 to do our analysis. recall from chapter one that there are 2
equivilent forms of taylors law:

01: exponential: sd = b \* mean^a 02: linear: ln(sd) = a \* ln(mean) +
ln(b)

In chapter_01 we saw that our data was linear, but some genes were
expressed at a very high level (that 1:10 ratio discussed in
chapter_02). In Chapter_02 we applied logs to our mean values and sd
values and put them into our taylor values dataframes. we then sorted
out any that were Zero. this will decrease the error at the fringes of
our data.

I will run this analysis on our mouse data first to build up. to begin I
will simply plot our log transformed values against one another.

``` r
ggplot(taylor_values_mouse_02, aes(x = log_m, y = log_sd)) +
  geom_point() +
  labs(title = "Gene Mean and Standard Deviation relationship", x = "ln(mean)", y = "ln(sd)") +
  theme_minimal()
```

![](Taylors_Final_files/figure-gfm/intial%20plot%20reveals%20linear%20relationships%20conservation-1.png)<!-- -->

\##RESULT##: Our data is linearly related, this is exactly what we
wanted to see! This means that nothing was lost in the transformation or
in the sorting. There is a very clear cutoff on the top of our data
which is the result of the 0.0000 sorting we did in chapter_02

Now lets add a regression line and see if we can perform a regression

``` r
ggplot(taylor_values_mouse_02, aes(x = log_m, y = log_sd)) +
  geom_point() +
  geom_smooth(method = "glm", color = "red", se = TRUE) +  # Regression line
  labs(title = "Gene Mean and Standard Deviation relationship", x = "ln(mean)", y = "ln(sd)") +
 theme_minimal()
```

    ## `geom_smooth()` using formula = 'y ~ x'

![](Taylors_Final_files/figure-gfm/adding%20regression%20line-1.png)<!-- -->
\##RESULT##: Our line appears to nicely fit our data with a slightly
negative slope from our data.

Is our line statistically signifigant? in order to test this we can ask
ggplot to calculate an regression coefficient R which will be between
0.0 and 1.0 with 0 being no relationship and 1 being perfectly related.

Becuase I dont want to install ggpubr I will go about calculating this
the hard way:

``` r
#Calculating the coorelations coefficient: 
cor_value <- cor(taylor_values_mouse_02$log_m, taylor_values_mouse_02$log_sd, use = "complete.obs")
#adding it to our plot:
ggplot(taylor_values_mouse_02, aes(x = log_m, y = log_sd)) +
  geom_point() +
  geom_smooth(method = "glm", color = "red", se = TRUE) +  # Regression line
  annotate("text", x = 1, y = 8, label = paste("R =", round(cor_value, 2)), size = 5) + #R^2 value display
  labs(title = "Gene Mean and Standard Deviation relationship", x = "ln(mean)", y = "ln(sd)") +
 theme_minimal()
```

    ## `geom_smooth()` using formula = 'y ~ x'

![](Taylors_Final_files/figure-gfm/more%20statistics-1.png)<!-- -->
\##RESULT##: Our data are almost perfectly related meaning that a change
in mean is almsot completely reflected as a change in sd

The cutoff for signifigance with R^2 is usually around 0.9 meaning that
our data are signifigant.

Now that we have established that we can move onto extracting the “a”
and “b” values by asking ggplot2 to add an equation for our line. But
becuase I am lazy and dont want to install ggpubr i will have base R
calculate the equation of my line.

``` r
lm_fit <- lm(log_sd ~ log_m, data = taylor_values_mouse_02)
eq <- paste("y =", round(coef(lm_fit)[2], 2), "* x +", round(coef(lm_fit)[1], 2))

# Compute Pearson correlation
cor_value <- cor(taylor_values_mouse_02$log_m, taylor_values_mouse_02$log_sd, use = "complete.obs")

# Plot
ggplot(taylor_values_mouse_02, aes(x = log_m, y = log_sd)) +
  geom_point() +
  geom_smooth(method = "glm", color = "red", se = TRUE) +
  annotate("text", x = 1, y = 8, label = paste("R =", round(cor_value, 2)), size = 5) +
  annotate("text", x = 1, y = 9, label = eq, size = 5) + # Regression equation
  labs(title = "Gene Mean and Standard Deviation relationship", x = "ln(mean)", y = "ln(sd)") +
  theme_minimal()
```

    ## `geom_smooth()` using formula = 'y ~ x'

![](Taylors_Final_files/figure-gfm/adding%20everything%20in-1.png)<!-- -->
\##RESULT##: We now have our values for “a” and “b”. a = 0.72 b = e^0.21
= 1.23. Where e is eulers exponential constant and 0.21 is the log value
of b.

Now lets make it look nice:

``` r
ggplot(taylor_values_mouse_02, aes(x = log_m, y = log_sd)) +
  geom_point() +
  geom_smooth(method = "glm", color = "red", se = TRUE) +  # Regression line
 annotate("text", x = 1, y = 8, label = paste("R =", round(cor_value, 2)), size = 5) +
  annotate("text", x = 1, y = 9, label = eq, size = 5) + # Regression equation
  labs(title = "Gene Mean and Standard Deviation relationship", x = "ln(mean)", y = "ln(sd)") +
 theme_set(theme_minimal() +
   theme(
              text = element_text(size = 14, family = "Georgia"),
              plot.title = element_text(face = "bold", hjust = 0.5),
              axis.text = element_text(color = "black"),
              panel.grid.major = element_line(color = "gray80"),
              panel.grid.minor = element_blank()
            ))
```

    ## `geom_smooth()` using formula = 'y ~ x'

![](Taylors_Final_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

Awesome! Now lets just use this final equation to plot our human data
and extract those values:

``` r
#calculate the line: 
lm_fit_human <- lm(log_sd ~ log_m, data = taylor_values_human_02)
eq_human <- paste("y =", round(coef(lm_fit_human)[2], 2), "* x +", round(coef(lm_fit)[1], 2))

# Compute Pearson correlation
cor_value_human <- cor(taylor_values_human_02$log_m, taylor_values_human_02$log_sd, use = "complete.obs")

ggplot(taylor_values_human_02, aes(x = log_m, y = log_sd)) +
  geom_point() +
  geom_smooth(method = "glm", color = "red", se = TRUE) +  # Regression line
 annotate("text", x = 1, y = 8, label = paste("R =", round(cor_value_human, 2)), size = 5) +
  annotate("text", x = 1, y = 9, label = eq_human, size = 5) + # Regression equation
  labs(title = "Gene Mean and Standard Deviation relationship", x = "ln(mean)", y = "ln(sd)") +
 theme_set(theme_minimal() +
   theme(
              text = element_text(size = 14, family = "Georgia"),
              plot.title = element_text(face = "bold", hjust = 0.5),
              axis.text = element_text(color = "black"),
              panel.grid.major = element_line(color = "gray80"),
              panel.grid.minor = element_blank()
            ))
```

    ## `geom_smooth()` using formula = 'y ~ x'

![](Taylors_Final_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
#lets just save all new files here: 
save(taylor_values_human_02, taylor_values_mouse_02, lm_fit, lm_fit_human, cor_value, cor_value_human, eq, eq_human, file = "/scratch/Shares/rinnclass/MASTER_CLASS/STUDENTS/genehomies/RESULTS/TPL_ANALYSIS/RData/Chapter_03.RData")

rm(list = ls())
```

\##RESULT## Our values are identical for our cooreltation coefficient
and our A and B values! this means that these datasets are similar in
terms of realative expression level and mean vs sd relationship! very
unexpected.

\##FUTURE_DIRECTIONS## 1. Now that we know that these distrbutions are
similar, I want to look at which genes are being highly expressed and
examine if they are paralogues to one another. 2. I want to do
timecourse analysis on these data and determine the change in the
coefficients over time. 3. Investigate isomorphic distributions in other
feilds with these values.

Chapter_04.1

``` r
#loading in data
Mouse_counts <- read_tsv("/scratch/Shares/rinnclass/MASTER_CLASS/STUDENTS/genehomies/DATA/MOUSE/salmon.merged.gene_counts.tsv")
```

    ## Rows: 55401 Columns: 17
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: "\t"
    ## chr  (2): gene_id, gene_name
    ## dbl (15): WT_0_1, WT_0_2, WT_0_3, WT_12_1, WT_12_2, WT_12_3, WT_24_1, WT_24_...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#using grep to seperate all timecourse values into seperate dataframes based on time point
counts_0 <- Mouse_counts[, grep("^WT_0_", colnames(Mouse_counts))]
counts_12 <- Mouse_counts[, grep("^WT_12_", colnames(Mouse_counts))]
counts_24 <- Mouse_counts[, grep("^WT_24_", colnames(Mouse_counts))]
counts_48 <- Mouse_counts[, grep("^WT_48_", colnames(Mouse_counts))]
counts_96 <- Mouse_counts[, grep("^WT_96_", colnames(Mouse_counts))]
```

``` r
# making a g2s fir downstream analysis and filtering
g2s <- Mouse_counts[, c(1,2)]
#making mouse counts numeric
Mouse_counts <- Mouse_counts[, -c(1,2)]
```

``` r
#lets write a function that can do all of our calculations for us to save few days worth of time.
taylor_func <- function(df) {
  #check if dataframe is numeric
  df_numeric <- df[sapply(df, is.numeric)]
  #do mean and standard deviation
  row_mean <- rowMeans(df_numeric, na.rm = TRUE)
  row_sd <- apply(df_numeric, 1, sd, na.rm = TRUE)
  #take logs of these values for actual analysis
  ln_mean <- log(row_mean)
  ln_sd <- log(row_sd)
  
  # Combine results into dataframe
  result_df <- data.frame(
    row_mean = row_mean,
    row_sd = row_sd,
    log_m = ln_mean,
    log_sd = ln_sd
  )
  
  # Remove rows where row_mean is 0
  result_df <- result_df[result_df$row_mean !=0, ]
  #return our results to the global enviornment.
  return(result_df)
}
```

``` r
# this efficiency stuff is actually so fun, here is a nested for loop that will retrieve all of the parameters for taylors law and run some statistics for us

# I want to do this globally for all dataframes in my enviornment. lets get them
df_names <- ls()[sapply(ls(), function(x) is.data.frame(get(x)))]

#lets initilaize a tibble.
regression_results <- tibble(
  Dataset = character(),            
  Slope = numeric(),                # regression slope (a in y = ax + b)
  Intercept = numeric(),            # regression intercept (b in y = ax + b)
  R_squared = numeric(),            # R² value 
  P_value = numeric(),              # p-value from the regression slope
  T_value_vs_first = numeric(),     # t-stat comparing slope to that of first model
  P_value_vs_first = numeric(),     # p-value of the t-test comparing slopes
  time = numeric()
)


# Initialize variables to store the first model's parameters (for p_val)
first_model <- NULL  # used to store the first model object
b1 <- NULL           # slope from the first model
se1 <- NULL          # standard error of slope from first model
df1 <- NULL          # degrees of freedom of first model

# Loop through each dataframe name in df_names
for (i in seq_along(df_names)) {
  df_name <- df_names[i]          # current dataframe name as string
  df <- get(df_name)              # retrieve dataframe object from the environment
  df <- taylor_func(df)           # apply your preprocessing/statistics function

  # Check if the expected columns exist before running regression
  if (all(c("log_m", "log_sd") %in% colnames(df))) {
    # Filter out incomplete or infinite values
    valid <- complete.cases(df$log_m, df$log_sd) &
             !is.infinite(df$log_m) &
             !is.infinite(df$log_sd)

    # Continue only if enough valid data points exist
    if (sum(valid) >= 2) {
      # Perform linear regression: log_sd ~ log_m
      model <- lm(log_sd ~ log_m, data = df[valid, ])

      # Extract regression slope (a), intercept (b), R², and p-value
      a <- coef(model)[["log_m"]]
      b <- coef(model)[["(Intercept)"]]
      r2 <- summary(model)$r.squared
      p_val <- coef(summary(model))["log_m", "Pr(>|t|)"]

      # If this is the first model, store its values for later comparison
      if (is.null(first_model)) {
        first_model <- model
        b1 <- a
        se1 <- coef(summary(model))["log_m", "Std. Error"]
        df1 <- df.residual(model)

        # No comparison yet for first model
        t_val <- NA
        p_val_vs_first <- NA
      } else {
        # For subsequent models, extract slope and SE
        b2 <- a
        se2 <- coef(summary(model))["log_m", "Std. Error"]
        df2 <- df.residual(model)

        # Calculate t-statistic to compare slope with first model's slope
        t_val <- (b1 - b2) / sqrt(se1^2 + se2^2)
        df_comp <- min(df1, df2)  # conservative degrees of freedom
        p_val_vs_first <- 2 * pt(-abs(t_val), df = df_comp)  # two-tailed p-value
      }

      # Add results to the output tibble
      regression_results <- add_row(
        regression_results,
        Dataset = df_name,
        Slope = a,
        Intercept = b,
        R_squared = r2,
        P_value = p_val,
        T_value_vs_first = t_val,
        P_value_vs_first = p_val_vs_first,
        time = i
      )
    } else {
      # Warn if there aren't enough points to perform regression
      warning(paste("Skipping", df_name, "- not enough valid points."))
    }

  } else {
    # Warn if expected columns are missing
    warning(paste("Skipping", df_name, "- missing log_m or log_sd columns."))
  }
}

# Print out the full regression summary table
regression_results <- regression_results %>% filter(row_number() <= n()-1)

print(regression_results)
```

    ## # A tibble: 5 × 8
    ##   Dataset   Slope Intercept R_squared P_value T_value_vs_first P_value_vs_first
    ##   <chr>     <dbl>     <dbl>     <dbl>   <dbl>            <dbl>            <dbl>
    ## 1 counts_0  0.717    -0.240     0.938       0            NA          NA        
    ## 2 counts_12 0.812    -0.195     0.958       0           -63.3         0        
    ## 3 counts_24 0.668    -0.172     0.921       0            29.9         4.19e-193
    ## 4 counts_48 0.710    -0.220     0.932       0             4.05        5.11e-  5
    ## 5 counts_96 0.785    -0.205     0.953       0           -44.6         0        
    ## # ℹ 1 more variable: time <dbl>

\##RESULTS## 1. Super significant results in our slope regression
parameter here likely driven by a high number of genes. 2. A
consistently high coorelation value tracks with our slope parameter
value which is curious.

``` r
#Lets take a look at our data graphically!
# slope over time
ggplot(regression_results, mapping = aes(x = time, y = Slope)) + 
  geom_point() +
  geom_smooth(formula = 'y ~ x', method = "loess")
```

![](Taylors_Final_files/figure-gfm/Plotting-1.png)<!-- -->

``` r
# intercept over time
ggplot(regression_results, mapping = aes(x = time, y = Intercept)) + 
  geom_point() +
  geom_smooth(formula = 'y ~ x', method = "loess")
```

![](Taylors_Final_files/figure-gfm/Plotting-2.png)<!-- -->

``` r
# R^2 over time
ggplot(regression_results, mapping = aes(x = time, y = R_squared)) + 
  geom_point() +
  geom_smooth(formula = 'y ~ x', method = "loess")
```

![](Taylors_Final_files/figure-gfm/Plotting-3.png)<!-- -->

``` r
# P val over time
ggplot(regression_results, mapping = aes(x = time, y = P_value_vs_first)) + 
  geom_point() +
  geom_smooth(formula = 'y ~ x', method = "loess")
```

![](Taylors_Final_files/figure-gfm/Plotting-4.png)<!-- -->

``` r
# T val over time
ggplot(regression_results, mapping = aes(x = time, y = T_value_vs_first)) + 
  geom_point() +
  geom_smooth(formula = 'y ~ x', method = "loess")
```

![](Taylors_Final_files/figure-gfm/Plotting-5.png)<!-- -->

``` r
# R^2 vs slope
ggplot(regression_results, mapping = aes(x = R_squared, y = Slope)) +
  geom_point() +
  geom_smooth(formula = 'y ~ x', method = "lm")
```

![](Taylors_Final_files/figure-gfm/Plotting-6.png)<!-- -->

``` r
# Loop over all objects with names that contain "counts_"
for (obj_name in ls(pattern = "^counts_")) {
  
  # Get the object
  df <- get(obj_name)
  
  # Check it's a data frame
  if (is.data.frame(df)) {
    
    # Identify only the numeric columns (e.g., expression counts)
    numeric_cols <- sapply(df, is.numeric)
    
    # Compute stats only on numeric columns
    row_mean <- rowMeans(df[, numeric_cols], na.rm = TRUE)
    row_sd   <- apply(df[, numeric_cols], 1, sd, na.rm = TRUE)
    row_se   <- row_sd / sqrt(rowSums(!is.na(df[, numeric_cols])))
    
    # Append new columns to the original dataframe
    df$row_mean <- row_mean
    df$row_sd   <- row_sd
    df$row_se   <- row_se

    # Assign the modified dataframe back to the original name
    assign(obj_name, df)
  }
}
```

``` r
save(g2s, regression_results, taylor_func, counts_0, counts_12, counts_24, counts_48, counts_96, file = "/scratch/Shares/rinnclass/MASTER_CLASS/STUDENTS/genehomies/RESULTS/TPL_ANALYSIS/RData/chapter_04.RData")

rm(list = ls())
```

Chapter_04.2

lets do the same thing for our human data, I wont, for the sake of time,
continue with this analysis.

``` r
human_counts <- read_tsv("/scratch/Shares/rinnclass/MASTER_CLASS/STUDENTS/genehomies/DATA/HUMAN/salmon.merged.gene_counts.tsv")
```

    ## Rows: 60649 Columns: 41
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: "\t"
    ## chr  (2): gene_id, gene_name
    ## dbl (39): gfp_0_1, gfp_0_2, gfp_0_3, gfp_0_4, gfp_0_5, gfp_0.5_1, gfp_0.5_2,...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
counts_0 <- human_counts[, grep("^gfp_0_", colnames(human_counts))]
counts_0.5 <- human_counts[, grep("^gfp_0.5_", colnames(human_counts))]
counts_1 <- human_counts[, grep("^gfp_1_", colnames(human_counts))]
counts_1.5 <- human_counts[, grep("^gfp_1.5_", colnames(human_counts))]
counts_2 <- human_counts[, grep("^gfp_2_", colnames(human_counts))]
counts_2.5 <- human_counts[, grep("^gfp_2.5_", colnames(human_counts))]
counts_3 <- human_counts[, grep("^gfp_3_", colnames(human_counts))]
counts_3.5 <- human_counts[, grep("^gfp_3.5_", colnames(human_counts))]
counts_4 <- human_counts[, grep("^gfp_4_", colnames(human_counts))]
counts_4.5 <- human_counts[, grep("^gfp_4.5_", colnames(human_counts))]
counts_5 <- human_counts[, grep("^gfp_5_", colnames(human_counts))]
counts_5.5 <- human_counts[, grep("^gfp_5.5_", colnames(human_counts))]
counts_12 <- human_counts[, grep("^gfp_12_", colnames(human_counts))]
counts_24 <- human_counts[, grep("^gfp_24_", colnames(human_counts))]
counts_48 <- human_counts[, grep("^gfp_48_", colnames(human_counts))]
counts_96 <- human_counts[, grep("^gfp_96_", colnames(human_counts))]
```

``` r
g2s <- human_counts[, c(1,2)]

human_counts <- human_counts[, -c(1,2)]
```

``` r
taylor_func <- function(df) {
  
  df_numeric <- df[sapply(df, is.numeric)]
  
  row_mean <- rowMeans(df_numeric, na.rm = TRUE)
  row_sd <- apply(df_numeric, 1, sd, na.rm = TRUE)
  
  ln_mean <- log(row_mean)
  ln_sd <- log(row_sd)
  
  # Combine results into dataframe
  result_df <- data.frame(
    row_mean = row_mean,
    row_sd = row_sd,
    log_m = ln_mean,
    log_sd = ln_sd
  )
  
  # Remove rows where row_mean is 0
  result_df <- result_df[result_df$row_mean !=0, ]
  
  return(result_df)
}
```

``` r
df_names <- ls()[sapply(ls(), function(x) is.data.frame(get(x)))]

hm_regression_results <- tibble(
  Dataset = character(),            
  Slope = numeric(),                # regression slope (a in y = ax + b)
  Intercept = numeric(),            # regression intercept (b in y = ax + b)
  R_squared = numeric(),            # R² value 
  P_value = numeric(),              # p-value from the regression slope
  T_value_vs_first = numeric(),     # t-stat comparing slope to that of first model
  P_value_vs_first = numeric(),     # p-value of the t-test comparing slopes
  time = numeric()
)


# Initialize variables to store the first model's parameters
first_model <- NULL  # used to store the first model object
b1 <- NULL           # slope from the first model
se1 <- NULL          # standard error of slope from first model
df1 <- NULL          # degrees of freedom of first model

# Loop through each dataframe name in df_names
for (i in seq_along(df_names)) {
  df_name <- df_names[i]          # current dataframe name as string
  df <- get(df_name)              # retrieve dataframe object from the environment
  df <- taylor_func(df)           # apply your preprocessing/statistics function

  # Check if the expected columns exist before running regression
  if (all(c("log_m", "log_sd") %in% colnames(df))) {
    # Filter out incomplete or infinite values
    valid <- complete.cases(df$log_m, df$log_sd) &
             !is.infinite(df$log_m) &
             !is.infinite(df$log_sd)

    # Continue only if enough valid data points exist
    if (sum(valid) >= 2) {
      # Perform linear regression: log_sd ~ log_m
      model <- lm(log_sd ~ log_m, data = df[valid, ])

      # Extract regression slope (a), intercept (b), R², and p-value
      a <- coef(model)[["log_m"]]
      b <- coef(model)[["(Intercept)"]]
      r2 <- summary(model)$r.squared
      p_val <- coef(summary(model))["log_m", "Pr(>|t|)"]

      # If this is the first model, store its values for later comparison
      if (is.null(first_model)) {
        first_model <- model
        b1 <- a
        se1 <- coef(summary(model))["log_m", "Std. Error"]
        df1 <- df.residual(model)

        # No comparison yet for first model
        t_val <- NA
        p_val_vs_first <- NA
      } else {
        # For subsequent models, extract slope and SE
        b2 <- a
        se2 <- coef(summary(model))["log_m", "Std. Error"]
        df2 <- df.residual(model)

        # Calculate t-statistic to compare slope with first model's slope
        t_val <- (b1 - b2) / sqrt(se1^2 + se2^2)
        df_comp <- min(df1, df2)  # conservative degrees of freedom
        p_val_vs_first <- 2 * pt(-abs(t_val), df = df_comp)  # two-tailed p-value
      }

      # Add results to the output tibble
      hm_regression_results <- add_row(
        hm_regression_results,
        Dataset = df_name,
        Slope = a,
        Intercept = b,
        R_squared = r2,
        P_value = p_val,
        T_value_vs_first = t_val,
        P_value_vs_first = p_val_vs_first,
        time = i
      )
    } else {
      # Warn if there aren't enough points to perform regression
      warning(paste("Skipping", df_name, "- not enough valid points."))
    }

  } else {
    # Warn if expected columns are missing
    warning(paste("Skipping", df_name, "- missing log_m or log_sd columns."))
  }
}

# Print out the full regression summary table
hm_regression_results <- hm_regression_results %>% filter(row_number() <= n()-1)

print(hm_regression_results)
```

    ## # A tibble: 16 × 8
    ##    Dataset   Slope Intercept R_squared P_value T_value_vs_first P_value_vs_first
    ##    <chr>     <dbl>     <dbl>     <dbl>   <dbl>            <dbl>            <dbl>
    ##  1 counts_0  0.793   0.0675      0.951       0             NA         NA        
    ##  2 counts_0… 0.859  -0.255       0.939       0            -41.6        0        
    ##  3 counts_1  0.660  -0.223       0.818       0             63.8        0        
    ##  4 counts_1… 0.689  -0.273       0.845       0             52.2        0        
    ##  5 counts_12 0.559   0.00435     0.879       0            160.         0        
    ##  6 counts_2  0.522  -0.0551      0.722       0            128.         0        
    ##  7 counts_2… 0.645  -0.226       0.822       0             73.5        0        
    ##  8 counts_24 0.588  -0.0358      0.893       0            142.         0        
    ##  9 counts_3  0.603  -0.132       0.743       0             83.0        0        
    ## 10 counts_3… 0.546  -0.0823      0.730       0            113.         0        
    ## 11 counts_4  0.742  -0.302       0.871       0             26.5        1.24e-152
    ## 12 counts_4… 0.592  -0.164       0.765       0             94.0        0        
    ## 13 counts_48 0.669  -0.0917      0.895       0             79.4        0        
    ## 14 counts_5  0.700  -0.284       0.847       0             46.6        0        
    ## 15 counts_5… 0.521  -0.0566      0.715       0            127.         0        
    ## 16 counts_96 0.705  -0.138       0.923       0             61.0        0        
    ## # ℹ 1 more variable: time <dbl>

``` r
#cool, this is very much like our mouse data.
rm(list = ls())
```

Chapter_05

I ran some fun python simulations to see about aspects of taylors law
coefficients and their relationships that I felt like including here.
Reticulate and I have an oppostional relationship, so for all intents
and purposes please copy, paste, and run these locally if you are
curious.

Chapter_06

I thought it would be fun to finish out the semester by seeing if the
orthologues we identified are driving the statistically significant
feature of Taylors law we are observing in mice. I will omit the human
case because there is just so much more work than there remains time
for, but hopefully these tools would be easily adaptable to such
analysis.

``` r
rm(list = ls())
#laoding data
load("/scratch/Shares/rinnclass/MASTER_CLASS/STUDENTS/genehomies/RESULTS/MOUSEVSHUMAN/orthologs_results.RData")

load("/scratch/Shares/rinnclass/MASTER_CLASS/STUDENTS/genehomies/RESULTS/TPL_ANALYSIS/RData/chapter_04.RData")
```

``` r
# Get all objects in the environment with names starting with 'counts_'
counts_names <- ls(pattern = "^counts_")

# Loop through each counts_ dataframe and prepend g2s
for (name in counts_names) {
  counts_df <- get(name)
  
  # Combine g2s and the counts dataframe (column-wise)
  combined_df <- cbind(g2s, counts_df)
  
  # Reassign back to the original name
  assign(name, combined_df, envir = .GlobalEnv)
}
##
# Get all objects in the environment with names starting with 'counts_'
counts_names <- ls(pattern = "^counts_")

# Loop through each counts_ dataframe
for (name in counts_names) {
  df <- get(name)
  
  # Check if the required columns exist
  if (all(c("row_mean", "row_sd") %in% colnames(df))) {
    df$ln_mean <- log(df$row_mean)
    df$ln_sd <- log(df$row_sd)
    
    # Reassign the updated dataframe back to the original variable name
    assign(name, df, envir = .GlobalEnv)
  } else {
    warning(paste("Skipping", name, "- missing 'row_mean' or 'row_sd'"))
  }
}
```

``` r
# Extract the list of gene names from your reference dataframe
# These are the gene names you want to filter and sort by
ordered_gene_names <- conserved_changing_orthologues_dereplicated$Mouse.gene.name
# Find all objects in the environment whose names start with "counts_"
# These are the dataframes you want to process
counts_names <- ls(pattern = "^counts_")

# Loop through each counts_ dataframe by name
for (name in counts_names) {
  
  # Get the actual dataframe object using the name
  df <- get(name)
  
  # Check if the dataframe contains a column named 'gene_name'
  if ("gene_name" %in% colnames(df)) {
    
    # Filter the dataframe to keep only rows where gene_name is in the reference list
    df_filtered <- df[df$gene_name %in% ordered_gene_names, ]
    
    # Convert gene_name to a factor with levels set to the reference order
    # This will allow us to sort it exactly as in ordered_gene_names
    df_filtered$gene_name <- factor(df_filtered$gene_name, levels = ordered_gene_names)
    
    # Sort the dataframe by gene_name using the factor levels
    df_sorted <- df_filtered[order(df_filtered$gene_name), ]
    
    # Assign the sorted dataframe back to its original name in the global environment
    assign(name, df_sorted, envir = .GlobalEnv)
    
  } else {
    # If the dataframe doesn't have a 'gene_name' column, issue a warning and skip it
    warning(paste("Skipping", name, "- missing 'gene_name' column"))
  }
}
```

``` r
# Find only data frames whose names start with "counts_"
df_names <- ls(pattern = "^counts_")
df_names <- df_names[sapply(df_names, function(x) is.data.frame(get(x)))]

# Initialize results tibble
ortho_reg_results <- tibble(
  Dataset = character(),            
  Slope = numeric(),                
  Intercept = numeric(),           
  R_squared = numeric(),           
  P_value = numeric(),             
  T_value_vs_first = numeric(),    
  P_value_vs_first = numeric(),    
  time = numeric()
)

# Initialize variables to store the first model's parameters
first_model <- NULL
b1 <- NULL
se1 <- NULL
df1 <- NULL

# Loop through each "counts_" dataframe
for (i in seq_along(df_names)) {
  df_name <- df_names[i]
  df <- get(df_name)

  # Proceed if ln_mean and log_sd exist
  if (all(c("ln_mean", "ln_sd") %in% colnames(df))) {
    valid <- complete.cases(df$ln_mean, df$ln_sd) &
             !is.infinite(df$ln_mean) &
             !is.infinite(df$ln_sd)

    if (sum(valid) >= 2) {
      model <- lm(ln_sd ~ ln_mean, data = df[valid, ])

      a <- coef(model)[["ln_mean"]]
      b <- coef(model)[["(Intercept)"]]
      r2 <- summary(model)$r.squared
      p_val <- coef(summary(model))["ln_mean", "Pr(>|t|)"]

      if (is.null(first_model)) {
        first_model <- model
        b1 <- a
        se1 <- coef(summary(model))["ln_mean", "Std. Error"]
        df1 <- df.residual(model)

        t_val <- NA
        p_val_vs_first <- NA
      } else {
        b2 <- a
        se2 <- coef(summary(model))["ln_mean", "Std. Error"]
        df2 <- df.residual(model)

        t_val <- (b1 - b2) / sqrt(se1^2 + se2^2)
        df_comp <- min(df1, df2)
        p_val_vs_first <- 2 * pt(-abs(t_val), df = df_comp)
      }

      # Append results
      ortho_reg_results <- add_row(
        ortho_reg_results,
        Dataset = df_name,
        Slope = a,
        Intercept = b,
        R_squared = r2,
        P_value = p_val,
        T_value_vs_first = t_val,
        P_value_vs_first = p_val_vs_first,
        time = i
      )
    } else {
      warning(paste("Skipping", df_name, "- not enough valid points."))
    }
  } else {
    warning(paste("Skipping", df_name, "- missing ln_mean or ln_sd columns."))
  }
}

# Remove the last row if needed (e.g., to skip a known empty one)
ortho_reg_results <- ortho_reg_results %>% filter(row_number() <= n() - 1)

# Output final results
print(ortho_reg_results)
```

    ## # A tibble: 5 × 8
    ##   Dataset   Slope Intercept R_squared  P_value T_value_vs_first P_value_vs_first
    ##   <chr>     <dbl>     <dbl>     <dbl>    <dbl>            <dbl>            <dbl>
    ## 1 counts_0  0.776    -0.859     0.752 7.94e-12           NA               NA    
    ## 2 counts_12 0.930    -1.04      0.801 1.80e-13           -1.39             0.173
    ## 3 counts_24 0.726    -0.653     0.781 9.32e-13            0.495            0.624
    ## 4 counts_48 0.693    -0.569     0.706 1.48e-10            0.772            0.446
    ## 5 counts_96 0.812    -0.614     0.761 4.33e-12           -0.321            0.750
    ## # ℹ 1 more variable: time <dbl>

\##RESULTS##: for our mouse data, the orthologues are potentially
driving the taylors law skew in the A regression parameter. More
statistics would be good, but are not permitted by time. Would be
willing to continue this into the future!
