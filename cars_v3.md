Cars dataset
================

Ubiqum Code Academy
-------------------

Example of solution

Preprocessing
-------------

### Load data and change variable types

``` r
pacman::p_load(ggplot2)

cars <- read.csv("C:/Users/gabri/Desktop/Ubiqum/R/Data_Analytics_Predicting_Customer_Preference/Task_1/cars.csv")

cars$speed.of.car <- as.numeric(cars$speed.of.car)
cars$distance.of.car <- as.numeric(cars$distance.of.car)
```

### Exploratory analysis

In the initial exploration you can already estimate an outlier

``` r
plot(cars$speed.of.car, cars$distance.of.car)
```

![](cars_v3_files/figure-markdown_github/plot%201-1.png)

Check for distributions
-----------------------

``` r
box_plot <- boxplot(cars[, c("distance.of.car","speed.of.car")])
```

![](cars_v3_files/figure-markdown_github/boxplot-1.png)

Exclude outlier
---------------

``` r
cars <- cars[which(cars$distance.of.car != box_plot$out),]
```

Modeling
--------

### Split data

``` r
set.seed(314)
train_size <- round(nrow(cars)*0.7)
test_size <- nrow(cars)-train_size
training_indices <- sample(seq_len(nrow(cars)), size = train_size)
train_set <- cars[training_indices,]

test_set <- cars[-training_indices,]
```

### Train a model

``` r
lm <- lm(distance.of.car~ speed.of.car,train_set)

Pred_dist <- predict(lm,test_set)

test_set$Pred_dist <- Pred_dist
```

### Plot model

``` r
ggplot(lm$model, aes_string(x = names(lm$model)[2], y = names(lm$model)[1])) + 
  geom_point() +
  stat_smooth(method = "lm", col = "red") +
  labs(title = paste("Adj R2 = ",signif(summary(lm)$adj.r.squared, 5),
                     "Intercept =",signif(lm$coef[[1]],5 ),
                     " Slope =",signif(lm$coef[[2]], 5),
                     " P =",signif(summary(lm)$coef[2,4], 5)))
```

![](cars_v3_files/figure-markdown_github/plot%20lm-1.png)

The intercept in the model above is at negative levels, which doesn't make logical sense. The car should be driving at a speed of -25 in order to achieve a distance of zero. Instead, we will try to fix the intercept at zero. Below, we can see the fit becomes worse when we fix the intercept at zero and keep the model linear.

``` r
lm_intercept <- lm(distance.of.car~ 0+speed.of.car,train_set)
Pred_dist_intercept <- predict(lm_intercept,test_set)


test_set <- cbind(test_set, Pred_dist_intercept)

ggplot(lm_intercept$model, aes_string(x = names(lm_intercept$model)[2], y = names(lm_intercept$model)[1])) + 
  geom_point() +
  stat_smooth(method = "lm", col = "red") +labs(title = paste("Adj R2 = ",signif(summary(lm_intercept)$adj.r.squared, 5),
                     "Intercept = 0 ",
                     " Slope =",signif(lm_intercept$coef[[1]], 5),
                     " P =",signif(summary(lm_intercept)$coef[1,3], 4)))
```

![](cars_v3_files/figure-markdown_github/model%20intercept-1.png)

``` r
test_set$abs_error_lm <- abs(test_set$distance.of.car - test_set$Pred_dist)
test_set$rel_error_lm <- test_set$abs_error_lm/test_set$distance.of.car


test_set$abs_error_model_intercept <- abs(test_set$distance.of.car - test_set$Pred_dist_intercept)
test_set$rel_error_model_intercept <- test_set$abs_error_model_intercept/test_set$distance.of.car

MAE_lm <- mean(test_set$abs_error_lm) 

MAE_model_intercept <- mean(test_set$abs_error_model_intercept)
```

Error Analysis
--------------

### Relative error

``` r
plot(test_set$speed.of.car, test_set$rel_error_lm, main = "Relative error")
```

![](cars_v3_files/figure-markdown_github/plot%202-1.png)

``` r
plot(test_set$speed.of.car, test_set$abs_error_lm, main = "Absolute error")
```

![](cars_v3_files/figure-markdown_github/plot%203-1.png)
