# Construction_area_detection

# Introduction
Earth observation is one of the major challenges facing the space industry. From the environment to meteorology and defense, there are many applications for such data. However, the amount of data to be processed is phenomenal. Artificial intelligence appears to be a suitable tool for solving this problem. Such devices are already in place, revolutionizing the aerospace industry in both the public and private sectors. One example is Telespasio, which uses satellite data to detect oil slicks on the surface of the ocean and trace them back to the ship that caused them.

## Description of the Problem
Our aim in this project is to classify satellite images into five categories:

- Demolition
- Road
- Residential
- Commercial
- Industrial
- Mega Projects

To solve this problem, we had to use several parameters that characterized an image. For the same image, we had parameters describing a polygon surrounding the area of interest, parameters describing the variance and mean intensity of each primary color, parameters describing the date and nature of each stage of construction, and more general parameters describing the environment in which the polygon was located.

Firstly, the data had to be processed to make it usable. Then we carried out feature engineering to isolate the most important parameters and create new variables with greater meaning, which would allow us to better characterize the image. Finally, we reduced the size of the data and implemented several machine learning algorithms, which we tested using the cross-validation method to select the best one.

# Feature Engineering

## Data Preprocessing

One of the goals is to generate a matrix suitable for dimensionality reduction. Initially, we need to convert categorical data into numerical representations. For instance, the Label Encoding method was used to deal with the "change-status" features at each date. Each construction label was mapped to a numerical value using a dictionary, ensuring ascending order to retain information about construction progress.

Handling "urban-type" and "geography-type" columns posed challenges since a single image could contain varying quantities of these elements. To address this, a One-Hot Encoding approach was adopted. We created new columns corresponding to each unique element in "urban-type" and "geography-type." For example, if an image contained elements A1 and A3, the respective columns would be populated with 1s and 0s.

Subsequently, attention was directed towards processing dates. Dates, being non-numeric, required conversion. We first converted them into datetime format. Such a format was not suitable with the operations of the rest of the code, so we converted the datetime format to the time since a reference date divided by a chosen time interval. We intentionally chose a time interval of the same order of magnitude as the date variation to avoid arbitrarily crushing the data. The reference was not important since we will then focus only on the difference between two dates.

## Creation of New Features

Additional columns were created to encapsulate potentially insightful information beyond mere date values. Specifically, the focus was on the deltas between consecutive construction dates, indicative of construction progress. Similarly, changes in average intensity and variance of green hues between dates could signal afforestation or deforestation activities, offering valuable insights into building nature. We performed the same process for other colors as a precaution.

Furthermore, features capturing average gray intensity per date and its evolution were introduced, potentially highlighting gray building or road construction activities. We created the feature of the average gray intensity by taking a linear combination of the average intensities of the three primary colors: red, blue, and green. Indeed, each color can be expressed as a linear combination of these three colors in the RGB system.

As a complement, features quantifying construction area and perimeter were incorporated, along with their ratio, providing insights into infrastructure shapes. Indeed, a low value of this ratio suggests an elongated shape, such as that of a road, for example. Additionally, the ratio of area to squared perimeter was computed to yield dimensionless indicators of infrastructure morphology.

## Additional Challenge: Managing Unassigned Information in Data Features

In the course of our project, we encountered a recurring issue where certain feature values could not be assigned, affecting both training and test datasets. This posed a significant challenge, as the machine learning models we employed were incapable of processing entries labeled as 'N/A'. To address this, we adopted a strategy of substituting each 'N/A' with the mean of the respective column.

This approach, while not without its limitations, proved to be an effective compromise. By replacing missing values with the column mean, we were able to maintain the integrity of our dataset without introducing excessive bias or significantly altering the underlying data distribution.

# Model Tuning and Comparison

## Dimension Reduction

### PCA
PCA is a dimension reduction method that seeks to reduce the size of the data while retaining as much information as possible. The idea is to project the data onto significant axes while maximizing the variance between each of the data items. The PCA method also reduces noise. Before implementing the PCA, the dataset had to be normalized. In our case, we decided to retain 95 percent of the energy. The method was necessary before the implementation of the SVM since our dataset was composed of roughly a hundred of features before dimension reduction. The PCA allowed to decrease the number of features until roughly 30.

### LDA
The LDA method reduces the size of the data while maximizing the interclass variance and minimizing the infraclass variance. To work, it therefore requires labeled training data. It is therefore a good reduction method for classification purposes. In FDA, the final dimension is limited by the number of labels of the output minus 1, which is 6-1=5 in that case. So we have chosen a final dimension of 5 in order to keep as much information as possible.

## Model Selection
In order to achieve the best performance, we implemented several algorithms which we tested using the cross-validation method. The algorithms were combined with the two noise reduction methods described above. This section describes the algorithms used and the results obtained. The algorithms have parameters to be optimized, so we generally carried out several tests. The results presented correspond to the algorithm with its optimal parameters.

### K-NN
The K-NN algorithm consists of labeling the data according to its proximity to the training data. The idea is to look at the k closest points and assign the majority label among them. If we choose a k that is too small, our algorithm becomes very sensitive to noise, but if we choose a k that is too large, we consider very distant points (no longer necessarily in the neighborhood) and our algorithm loses its meaning. We chose k=4.

After cross-validation we obtain a score of 0.60 for input data processed by LDA and a score of 0.52 for input data processed by PCA. It is not surprising to have better results with LDA than with PCA. Indeed the k-NN algorithm is sensitive to high dimensionality. This is mainly due to the fact that increasing dimensionality can dilute the measure of distance, making the distances less meaningful.

### SVM
SVM programs try to separate data using hyperplanes. If no clear hyperplane is found, a kernel function can be used to make the data separable. SVMs are less sensitive to high dimensionality compared to K-NN, particularly with the use of kernels to project data into higher-dimensional feature spaces where the data can be more easily separated. As we work in high dimensionality, in our situation we used an RBF kernel function, i.e., a radial basis function.

After cross-validation, we obtained a score of 0.62 for input data processed by LDA and a score of 0.6 for input data processed by PCA. The main problem of SVM was its computational time, especially with PCA.

### Random Forest
The Random Forest algorithm aggregates the results of several decision trees. We chose it because it provides a robust prediction that is less prone to overfitting. In this algorithm, the variable parameter is the number of trees aggregated. Aggregating a large number of trees reduces overfitting and therefore variance, but greatly increases computation time and can increase bias. We adjusted this parameter by trial and error, trying to find the best compromise between results and computation time. We chose n=200.

After cross-validation, we obtain a score of 0.65 for non-reduced data. The result was worse with reduced data. That is not surprising since Random Forest performs well on datasets with high dimensionality by naturally managing the curse of dimensionality and focusing on the most informative features. It captures non-linear relationships and is robust to noise, potentially making dimensionality reduction less critical for maintaining predictive accuracy.

# Conclusion
After testing it on the test data, we obtain a score of 0.82 using Random Forest without reduction and with n=200 trees.

The machine learning algorithms we have put in place significantly increase the probability of classifying an image correctly. However, they are far from perfect and could be improved. For example, with more time, we could have determined the initial parameters of the algorithms more precisely by using gradient descent. We could also probably have prepared the data more efficiently by imagining other functions during the feature engineering phase.
