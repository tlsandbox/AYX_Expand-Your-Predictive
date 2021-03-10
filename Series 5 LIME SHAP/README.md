# Expand Your Predictive Palette V: No More Black Box with LIME
https://community.alteryx.com/t5/Data-Science/Expand-Your-Predictive-Palette-V-No-More-Black-Box-with-LIME/ba-p/656918

Background
In the first year of the data mining project (yes, still called data mining at that time), our team spent weeks collecting data, building pipelines and tuning parameters. At last, we came up with a relatively stable and accurate model for our client. We were thrilled and excited to present the result to their board of directors. One hour of presentation had passed ...

Fair enough. We were using ensemble methods, and the client was entirely new to the predictive world. So we revised our deck and provided explanations. The client started to understand the concept and the ROI as a whole. However, when we touched on the model explanation, there was a gap between their industrial experience and our engineered features, making them perceive our model as a black box. At that moment, we lost their trust.

Trust
This incident exemplifies @SusanCS 's latest shoutout to model interpretability.

With the growing complexity of algorithms, it’s common to find ourselves stuck in the middle between model accuracy and interpretability. To bridge the gap and increase business adoption or regulatory acceptance, many great minds are inventing tools to break down the black box. One famous package is called LIME. :lemon:
 
New to local explanation methods like LIME and SHAP? Go check out Susan’s blog, and you are well covered. All local explainable tools share the same objectives:
 
Deciding if one should trust a prediction
Choosing between models
Improving an untrustworthy classifier
 
To demonstrate that, we use the 2015 Taiwan credit card dataset from Kaggle as a Probability of Default (PD) use case, where our classification target is the column DEFAULT. More details on the metadata can be found on the Kaggle page.
 
Let’s download the workflow and start.
 
Global
After installing the packages, run the workflow and take a look at the first section.
 
Generally, there are two approaches to explain the model GLOBALLY:
 
Variable Importance Plot
Partial Effect Plot

In the Alteryx world, we provide a wide range of tool sets. In this case, the Random Forest model report lists the top 3 variables impacting the probability of default (PD) rate:
 
Pay_0 = Latest Payment Status
Bill_AMT1 = Latest Bill Statement Amount
Limit Balance = Credit Card Threshold
 
Meanwhile, Logistic Regression provides a Conditional Density Plot, which outlines each variable’s global contribution towards our target, e.g., The higher the Limit Balance, the higher the PD rate.
 
 
Local
Although ensemble-based algorithms like Random Forest are fast and powerful, they are not easily interpretable. Read more in @SydneyF's post: an Introduction to Random Forest.
 
Furthermore, our audience in real life often cares more about their specific business scenarios rather than the overall picture, asking questions like:
 
Could we look at the top 1% of customers/products/locations or particular client 14756?
For new clients without default history, how does our model affect each of them separately?
What are the shared features across the five Default cases last month?
 
From a particular client segment to a fixed time horizon, these are the daily life questions you cannot solve by global methods, but rather need a local one. Here we go to the second part.
 

LIME
Input
 
Using the macro is straightforward, with just three steps:
 
Step 1: Connect your model object to M; Training dataset to D; Local / Specific dataset to S
 
Step 2: Select your model category: Classification or Regression
 
Step 3: Select the number of features and permutations based on your reporting preference and hardware performance
 
Click run and you will have two outputs: data and a report.
 
Data output
 
A summary table contains every feature being used and their weighting, coefficient and description in every individual case! Plus, you could further aggregate and sort the data as below:
 
Pay_0 = Latest Payment Status remains the most important factor for our top 10 clients from the feature frequency ranking. We also found out Pay_3 and Pay_2 = July & August Payment Status are exceptionally useful in this portfolio.
 
How about client 14756? Let’s use the report option.
 
Report output
 
Inside the LIME report, two charts are presented: Case Bar and Feature Heat Map.
The bi-directional bar chart is useful to investigate cases. In the example of client 14756, the strongest drivers leading to default are none of the factors mentioned above but instead Pay_AMT4, even though the client had paid duly in September (Pay_0 = -1)!
 
To get a better idea of the recent default cases, let’s scroll down to the third section. We are tuning the auto label to a specific label this time. If we input the target variable value as 1, LIME will construct an interpretable representation of only the Default value.
 
Click the report option one more time; the feature heat map (screenshot above) helps us identify the features shared across all default cases. Clients paying duly in the current month (PAY_0 = 0) and May (PAY_5 = 0) are the crucial factors bringing down the PD rate.
 
Although this is a new batch of scoring data, the LIME local observation is aligned with our global model, a healthy signal. Conversely, if we spot any patterns contradicting the global findings, we should investigate further, as this might indicate a data shift!
 
OK, so far, we've achieved 2 out of 3 objectives. How about choosing between models? What are the algorithms this macro supports? Glad you asked.
 
 
Expand
The LIME package by design is highly dynamic, which allows users to add any external library. For this LIME tool, it supports 11 predictive tools from our predictive palette!
 
Perhaps you are uncovering linear/non-linear relationships with Count and Spline Regressions, or testing three sets of classifiers with SVM, Naive Bayes and Gradient Boosted Machine. With LIME, you can generate multiple feature summary tables and reports in one shot for local and case detailed comparison.
 
That’s it for LIME. Give it a try on your workflow, and surprise your audience by showing that the predictive model is not so complex!
 
For geeks, below are some coding snippets for LIME customization.
 
 
Adding New Algorithms
 
Keen to build out support for more algorithms? Here is the template. Within the R macro, there are examples as well.
 
 
 ### Create /New Package/ Functions
model_type./New Package/ <- function(x, ...) {
  return("classification")
}

predict_model./New Package/ <- function(x, newdata, ...) {
  pred <- predict(x, newdata, na.action = na.pass)
  return(as.data.frame(pred))
}
 
 
 
Tuning Parameters
 
LIME has two other key parameters for tuning:
 
Distribution Function: Gower (Default), Euclidean, Manhattan, etc.
Feature Selection Methods: Auto (Default), Forward Selection, Ridge Regression, Lasso, Tree, etc.
Feel free to add the parameter under the “explain” line.
 
 
#Distribution Function & Feature Selection parameters
explain(x, explainer..., feature_select = "auto", dist_fun = "gower")
 
 
 
Bonus: SHAP
A year ago, I wrote an XGBoost macro, and admittedly there are still lots of improvements that can be done, especially on the reporting layer. Unlike most of the predictive algorithms, XGBoost requires a data matrix rather than a tabular format. Instead of pressing the LIME too hard, I decided to power up the original macro by introducing another popular package: SHAP or Shapley Additive Explanations.
 
While LIME builds a linear model, SHAP calculates an average marginal contribution score called Shapley Value on each prediction. The advantage of this is maintaining global consistency while interpreting individual records. The downside? It’s relatively slower than LIME, so pay attention to the data size when using SHAP.
 
If you download the XGBoost macro in the Alteryx Galley again, run the workflow and you will see two new charts under the XGBoost Python reporting output.
 
Here are some highlights:
 
SHAP Summary Plot (Top Left)
 
Variables are ranked in descending order of importance
Exact cut red and blue colors indicate local and global models' alignment
In this example, Distance and Sale align, but the Profit column produces a mixed signal
 
SHAP Force Plot (Bottom)
 
Base Value is a predicted value without any feature influence, e.g., -0.8517
f(x) is the final predicted value after all features are added, e.g., 3.45
Red = Increased Value; Blue = Decreased Value. Field 2 to Field 8 all push the Base Value higher and to the right

The SHAP plot by itself is interactive JavaScript, which can run within the Jupyter Notebook. For instance, the top right is SHAP All Cases Force Plot. You can hover your mouse over each record (X-axis) to examine its respective feature effect and its predicted value (Y-axis). Make sure to open up the macro and experiment with the script.
 
Last but not least, there is a feature waterfall chart in the XGBoost R tool. Currently, I disable that as it requires offline package installation. Feel free to "uncomment" it and PM me if you encounter any issues.
Additional Resources:
"Why Should I Trust You?": Explaining the Predictions of Any Classifier
The 4 types of additive Feature Importances
LIME vs. SHAP
