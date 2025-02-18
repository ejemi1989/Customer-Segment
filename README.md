





This project was completed as a part of Udacity’s Machine Learning Nanodegree.

## Introduction

This project involves analyzing a dataset that captures the annual spending amounts (in monetary units) of customers across various product categories. The primary objective is to uncover and describe the variations among different types of customers that a wholesale distributor serves. Gaining such insights will help the distributor optimize their delivery services to better address the unique needs of each customer segment.

The dataset used in this analysis is sourced from the UCI Machine Learning Repository. To maintain focus on customer spending patterns, the features ‘Channel’ and ‘Region’ will be excluded, with attention directed solely to the six product categories included in the dataset.

The dataset for this project can be found on the UCI Machine Learning Repository. For the purposes of this project, the features 'Channel' and 'Region' will be excluded in the analysis — with focus instead on the six product categories recorded for customers.

In [9]:

# Import libraries necessary for this project
import numpy as np
import pandas as pd
from IPython.display import display # Allows the use of display() for DataFrames
# Import supplementary visualizations code visuals.py
import visuals as vs
# Pretty display for notebooks
%matplotlib inline
# Load the wholesale customers dataset
try:
    data = pd.read_csv("customers.csv")
    data.drop(['Region', 'Channel'], axis = 1, inplace = True)
    print("Wholesale customers dataset has {} samples with {} features each.".format(*data.shape))
except:
    print("Dataset could not be loaded. Is the dataset missing?")
Wholesale customers dataset has 440 samples with 6 features each.
Data Exploration
In this section, we will begin exploring the data through visualizations and code to understand how each feature is related to the others.

The dataset is composed of six important product categories: ‘Fresh’, ‘Milk’, ‘Grocery’, ‘Frozen’, ‘Detergents_Paper’, and ‘Delicatessen’. The code block below produces a statistical summary for each of the above product categories.

In [10]:

# Display a description of the dataset
display(data.describe())
FreshMilkGroceryFrozenDetergents_PaperDelicatessencount440.000000440.000000440.000000440.000000440.000000440.000000mean12000.2977275796.2659097951.2772733071.9318182881.4931821524.870455std12647.3288657380.3771759503.1628294854.6733334767.8544482820.105937min3.00000055.0000003.00000025.0000003.0000003.00000025%3127.7500001533.0000002153.000000742.250000256.750000408.25000050%8504.0000003627.0000004755.5000001526.000000816.500000965.50000075%16933.7500007190.25000010655.7500003554.2500003922.0000001820.250000max112151.00000073498.00000092780.00000060869.00000040827.00000047943.000000

Selecting Samples
To get a better understanding of the customers and how their data will transform through the analysis, lets select a few sample data points and explore them in more detail.

In [12]:

indices = [26,176,392]
# Create a DataFrame of the chosen samples
samples = pd.DataFrame(data.loc[indices], columns = data.keys()).reset_index(drop = True)
print("Chosen samples of wholesale customers dataset:")
display(samples)
Chosen samples of wholesale customers dataset:
FreshMilkGroceryFrozenDetergents_PaperDelicatessen09898961286131512428331456406958653673681532230251841803600659122654

Guessing Establishments
Based on the total purchase costs for each product category and their statistical descriptions, we can infer the following about the chosen sample customers:

Customer 1 (Index 0):
Likely represents a restaurant.

High purchases in the Frozen category.
Moderate purchases in Fresh and Deli, close to the median values.
These patterns suggest a focus on prepared foods and ingredients often used in restaurants.
Customer 2 (Index 1):
Likely represents a supermarket.

High or near-median purchases across most product categories, except Deli, which is relatively low.
This indicates a broad inventory of goods, typical of a supermarket, though it might lack a deli section.
Customer 3 (Index 2):
Likely represents a café.

High purchases of Milk.
Moderate purchases of Groceries and Deli.
Lower-than-median purchases of Fresh and Frozen products.
These trends align with a café’s reliance on dairy (e.g., milk for coffee) and light groceries, while fresh produce and frozen goods are less central.
Feature Relevance
One interesting thought to consider is if one (or more) of the six product categories is actually relevant for understanding customer purchasing. That is to say, is it possible to determine whether customers purchasing some amount of one category of products will necessarily purchase some proportional amount of another category of products? We can make this determination quite easily by training a supervised regression learner on a subset of the data with one feature removed, and then score how well that model can predict the removed feature.

Lets do this for the ‘Milk’ feature.

In [15]:

from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeRegressor
new_data = data.drop(['Milk'],axis=1)
X_train, X_test, y_train, y_test = train_test_split(new_data,data['Milk'],test_size=0.25,random_state=101)
regressor = DecisionTreeRegressor(random_state=101).fit(X_train,y_train)
score = regressor.score(X_test,y_test)
print(score)
0.29571438444092835
Feature Relevance Prediction
Our attempt to predict the Milk feature (annual spending on milk products) using the other product categories yielded an R2R²R2 score of 0.2957, which indicates a weak relationship between Milk and the other features. This result suggests that Milk purchases cannot be reliably inferred from spending in other categories.

Thus, the Milk feature provides unique information about customer spending habits and plays an essential role in the dataset. Its inclusion helps in accurately representing the variability in customer behavior, as its spending pattern is not well-captured by other product features.

Visualizing Feature Distributions
To better understand the relationships between the six product categories, we can construct a scatter matrix to examine pairwise relationships. The scatter matrix will help us observe:

Correlations:

If Milk is independent of other features, the scatter plots for Milk against other features will not exhibit clear trends or patterns.
Strong correlations between features will manifest as linear or structured patterns in their respective scatter plots.
Feature Relevance:

If Milk shows weak or no correlation with other features, it reaffirms that Milk is a standalone feature contributing unique information.
In [17]:

# Produce a scatter matrix for each pair of features in the data
pd.plotting.scatter_matrix(data, alpha = 0.3, figsize = (14,8), diagonal = 'kde');
Correlations
The scatter matrix reveals a few notable correlations between product categories:

Milk and Groceries:
There is a mild positive correlation between spending on Milk and Groceries, suggesting that customers who purchase more groceries tend to spend somewhat more on milk products as well.

Milk and Detergents_Paper:
A mild positive correlation also exists between Milk and Detergents_Paper, indicating some overlap in purchasing patterns for these categories.

Groceries and Detergents_Paper:
A stronger correlation is observed here, reflecting that customers spending more on groceries also tend to spend proportionally more on detergents and paper products.

These correlations confirm that while some relationships exist, Milk is not strongly correlated with most other features in the dataset, reinforcing its role as a distinct contributor to understanding customer behavior

Data Preprocessing
Preprocessing the Data
Preprocessing ensures that the dataset is well-prepared for analysis and modeling, making the results more meaningful and reliable. In this section, we will focus on feature scaling and outlier detection to improve the representation of customer behavior.

Feature Scaling
When dealing with financial data, where features are often skewed and vary greatly in magnitude, applying a non-linear transformation is essential for normalizing the data. Two common approaches are:

Box-Cox Transformation:

This statistical method identifies the best power transformation to reduce skewness and approximate a normal distribution.
It is particularly useful when the data contains negative values, as it can accommodate a broader range of distributions.
Natural Logarithm Transformation:

A simpler and effective method for reducing skewness, especially when the data is strictly positive.
It compresses large values, bringing extreme outliers closer to the median while retaining the relative scale.
Given that financial data is typically non-negative, the natural logarithm is often sufficient for scaling.

Steps for Scaling:
Apply Log Transformation:
Transform each feature in the dataset using the natural logarithm:

xscaled=ln⁡(x+1)x_{\text{scaled}} = \ln(x + 1)xscaled​=ln(x+1)

Adding 1 ensures that zeros in the data are handled appropriately.

Verify Results:
After transformation:

Reassess the distributions of the features.
Check for reduced skewness and more symmetric distributions.
Outlier Detection
Outliers can disproportionately influence the analysis and skew results. Detecting and optionally removing outliers involves:

Identify Outliers:

Use Interquartile Range (IQR) to flag extreme values.
Outliers are defined as points outside the range: [Q1−1.5⋅IQR,Q3+1.5⋅IQR][Q1–1.5 \cdot IQR, Q3 + 1.5 \cdot IQR][Q1−1.5⋅IQR,Q3+1.5⋅IQR] where Q1Q1Q1 and Q3Q3Q3 are the first and third quartiles, respectively.
Handle Outliers:

Remove them from the dataset if they are deemed to represent noise or irrelevant data.
Retain them if they are meaningful and represent a distinct customer group.
In [19]:

# Scale the data using the natural logarithm
log_data = data.apply(lambda x: np.log(x))
# Scale the sample data using the natural logarithm
log_samples = samples.apply(lambda x: np.log(x))
# Produce a scatter matrix for each pair of newly-transformed features
pd.plotting.scatter_matrix(log_data, alpha = 0.3, figsize = (14,8), diagonal = 'kde');
Observation
After applying a natural logarithm scaling to the data, the distribution of each feature should appear much more normal.

Let’s check out our log transformed samples.

In [20]:

# Display the log-transformed sample data
display(log_samples)
FreshMilkGroceryFrozenDetergents_PaperDelicatessen09.2000886.8679747.9589268.0554755.4889386.725034110.7285408.8476478.7850818.9049027.3343295.43807926.2499758.3380678.1886896.4907244.8040216.483107

Outlier Detection
Detecting Outliers Using Tukey’s Method
Identifying and handling outliers is crucial in data preprocessing, as they can distort analyses and lead to misleading conclusions. Tukey’s Method is a robust technique for detecting outliers, defined as data points lying outside a specific range based on the interquartile range (IQR).

Steps for Outlier Detection:
Calculate the IQR:
For each feature:

IQR=Q3−Q1\text{IQR} = Q3 — Q1IQR=Q3−Q1

where Q1Q1Q1 (25th percentile) and Q3Q3Q3 (75th percentile) represent the lower and upper quartiles, respectively.

Determine Outlier Step:
Multiply the IQR by a factor of 1.5:

Outlier Step=1.5⋅IQR\text{Outlier Step} = 1.5 \cdot \text{IQR}Outlier Step=1.5⋅IQR

Define Outlier Range:
Identify data points falling outside the range:

[Q1−Outlier Step,Q3+Outlier Step][Q1 — \text{Outlier Step}, Q3 + \text{Outlier Step}][Q1−Outlier Step,Q3+Outlier Step]

Values below Q1−Outlier StepQ1 — \text{Outlier Step}Q1−Outlier Step or above Q3+Outlier StepQ3 + \text{Outlier Step}Q3+Outlier Step are classified as outliers.
Flag Outliers:
For each feature, count and store data points flagged as outliers.

In [22]:

# Select the indices for data points you wish to remove
outliers  = []
# For each feature find the data points with extreme high or low values
for feature in log_data.keys():
    
    # Calculate Q1 (25th percentile of the data) for the given feature
    Q1 = np.percentile(log_data[feature],25)
    
    # Calculate Q3 (75th percentile of the data) for the given feature
    Q3 = np.percentile(log_data[feature],75)
    
    # Use the interquartile range to calculate an outlier step (1.5 times the interquartile range)
    step = (Q3-Q1) * 1.5
    
    # Display the outliers
    print("Data points considered outliers for the feature '{}':".format(feature))
    out = log_data[~((log_data[feature] >= Q1 - step) & (log_data[feature] <= Q3 + step))]
    display(out)
    outliers = outliers + list(out.index.values)
    
#Creating list of more outliers which are the same for multiple features.
outliers = list(set([x for x in outliers if outliers.count(x) > 1]))    
print("Outliers: {}".format(outliers))
# Remove the outliers, if any were specified 
good_data = log_data.drop(log_data.index[outliers]).reset_index(drop = True)
print("The good dataset now has {} observations after removing outliers.".format(len(good_data)))
Data points considered outliers for the feature 'Fresh':
FreshMilkGroceryFrozenDetergents_PaperDelicatessen654.4426519.95032310.7326513.58351910.0953887.260523662.1972257.3356348.9115305.1647868.1513333.295837815.3890729.1632499.5751925.6454478.9641845.049856951.0986127.9793398.7406576.0867755.4071726.563856963.1354947.8694029.0018394.9767348.2620435.3798971284.9416429.0878348.2487914.9558276.9679091.0986121715.29831710.1605309.8942456.4785109.0794348.7403371935.1929578.1562239.9179826.8658918.6337316.5012902182.8903728.9231919.6293807.1585148.4757468.7596693045.0814048.91731110.1175106.4248699.3744137.7873823055.4930619.4680019.0883996.6833618.2710375.3518583381.0986125.8081428.8566619.6550902.7080506.3099183534.7621748.7425749.9618985.4293469.0690077.0130163555.2470246.5889267.6068855.5012585.2149364.8441873573.6109187.15070110.0110864.9199818.8168534.7004804124.5747118.1900779.4254524.5849677.9963174.127134

Data points considered outliers for the feature 'Milk':
FreshMilkGroceryFrozenDetergents_PaperDelicatessen8610.03998311.20501310.3770476.8946709.9069816.805723986.2205904.7184996.6567276.7968244.0253524.8828021546.4329404.0073334.9199814.3174881.9459102.07944235610.0295034.8978405.3844958.0573772.1972256.306275

Data points considered outliers for the feature 'Grocery':
FreshMilkGroceryFrozenDetergents_PaperDelicatessen759.9231927.0361481.0986128.3909491.0986126.8824371546.4329404.0073334.9199814.3174881.9459102.079442

Data points considered outliers for the feature 'Frozen':
FreshMilkGroceryFrozenDetergents_PaperDelicatessen388.4318539.6632619.7237033.4965088.8473606.070738578.5972979.2036189.2578923.6375868.9322137.156177654.4426519.95032310.7326513.58351910.0953887.26052314510.0005699.03408010.4571433.7376709.4407388.3961551757.7591878.9676329.3821063.9512448.3418877.4366172646.9782149.1777149.6450414.1108748.6961767.14282732510.3956509.7281819.51973511.0164797.1483468.6321284208.4020078.5690269.4900153.2188768.8273217.2392154299.0603317.4673718.1831183.8501484.4308177.8244464397.9327217.4372067.8280384.1743876.1675163.951244

Data points considered outliers for the feature 'Detergents_Paper':
FreshMilkGroceryFrozenDetergents_PaperDelicatessen759.9231927.0361481.0986128.3909491.0986126.8824371619.4281906.2915695.6454476.9957661.0986127.711101

Data points considered outliers for the feature 'Delicatessen':
FreshMilkGroceryFrozenDetergents_PaperDelicatessen662.1972257.3356348.9115305.1647868.1513333.2958371097.2485049.72489910.2745686.5117456.7286291.0986121284.9416429.0878348.2487914.9558276.9679091.0986121378.0349558.9971479.0218406.4937546.5806393.58351914210.5196468.8751479.0183328.0047002.9957321.0986121546.4329404.0073334.9199814.3174881.9459102.07944218310.51452910.6908089.91195210.5059995.47646410.7777681845.7899606.8221978.4574434.3040655.8111412.3978951877.7989338.9874479.1920758.7433728.1487351.0986122036.3681876.5294197.7034596.1506036.8606642.8903722336.8710918.5139888.1065156.8426836.0137151.94591028510.6029656.4614688.1886896.9488976.0776422.89037228910.6639665.6559926.1548587.2356193.4657363.0910423437.4318928.84850910.1779327.2834489.6465933.610918

Outliers: [128, 65, 66, 75, 154]
The good dataset now has 435 observations after removing outliers.
After inspecting the dataset, we found:

No outliers exist within the sample.
A total of 5 data points are considered outliers for more than one feature.
To preserve information while mitigating the impact of extreme values:

Only multi-feature outliers are removed, as they are more likely to skew the analysis significantly.
These outliers can also be analyzed separately to understand their causes and patterns, but they are excluded from aggregate analyses for reliability.
Feature Transformation

To uncover the underlying structure of the dataset and identify key patterns, we use Principal Component Analysis (PCA).

Steps for PCA Analysis:
Prepare Data:

Ensure the data has been scaled (e.g., using the natural logarithm).
Remove any outliers deemed unsuitable for aggregate analysis.
Apply PCA:

PCA identifies linear combinations of features (principal components) that maximize variance.
These components are orthogonal and represent uncorrelated dimensions of the data.
Explained Variance Ratio:

PCA provides the explained variance ratio for each principal component, indicating how much of the total variance is captured.
The sum of the explained variance ratios across components shows how much of the dataset’s variability is preserved by the selected components.
Dimensionality Reduction:

Retain components that collectively explain a significant portion of the variance (e.g., 95%).
The reduced dataset provides a simpler, compact representation while retaining meaningful information.
In [23]:

from sklearn.decomposition import PCA
# Apply PCA by fitting the good data with the same number of dimensions as features
pca = PCA().fit(good_data)
# Transform log_samples using the PCA fit above
pca_samples = pca.transform(log_samples)
# Generate PCA results plot
pca_results = vs.pca_results(good_data, pca)
The first and second features, in total, explain approx. 70.8% of the variance in our data.

The first four features, in total, explain approx. 93.11% of the variance.

In terms of customer spending,

Dimension 1 has a high positive weight for Milk, Grocery, and Detergents_Paper features. This might represent Hotels, where these items are usually needed for the guests.
Dimension 2 has a high positive weight for Fresh, Frozen, and Delicatessen. This dimension might represent ‘restaurants’, where these items are used for ingredients in cooking dishes.
Dimension 3 has a high positive weight for Deli and Frozen features, and a low posiive weight for Milk, but has negative weights for everything else. This dimension might represent Delis.
Dimension 4 has positive weights for Frozen,Detergents_Paper and Groceries, while being negative for Fresh and Deli. It’s a bit tricky to pin this segment down, but I do believe that there are shops that sell frozen goods exclusively.
Let’s see how the log-transformed sample data has changed after having a PCA transformation applied to it in six dimensions.

In [24]:

# Display sample log-data after having a PCA transformation applied
display(pd.DataFrame(np.round(pca_samples, 4), columns = pca_results.index.values))
Dimension 1Dimension 2Dimension 3Dimension 4Dimension 5Dimension 601.9083–0.37650.19240.1502–0.38520.53671–0.0349–1.6819–1.71151.66130.5394–0.154820.99552.31691.7454–0.45691.2462–0.0669

Dimensionality Reduction
When using principal component analysis, one of the main goals is to reduce the dimensionality of the data — in effect, reducing the complexity of the problem. Dimensionality reduction comes at a cost: Fewer dimensions used implies less of the total variance in the data is being explained. Because of this, the cumulative explained variance ratio is extremely important for knowing how many dimensions are necessary for the problem. Additionally, if a signifiant amount of variance is explained by only two or three dimensions, the reduced data can be visualized afterwards.

In [25]:

# Apply PCA by fitting the good data with only two dimensions
pca = PCA(n_components=2).fit(good_data)
# Transform the good data using the PCA fit above
reduced_data = pca.transform(good_data)
# Transform log_samples using the PCA fit above
pca_samples = pca.transform(log_samples)
# Create a DataFrame for the reduced data
reduced_data = pd.DataFrame(reduced_data, columns = ['Dimension 1', 'Dimension 2'])
Let’s see how the log-transformed sample data has changed after having a PCA transformation applied to it using only two dimensions.

In [26]:

# Display sample log-data after applying PCA transformation in two dimensions
display(pd.DataFrame(np.round(pca_samples, 4), columns = ['Dimension 1', 'Dimension 2']))
Dimension 1Dimension 201.9083–0.37651–0.0349–1.681920.99552.3169

Visualizing a Biplot
A biplot is a scatterplot where each data point is represented by its scores along the principal components. The axes are the principal components (in this case Dimension 1 and Dimension 2). In addition, the biplot shows the projection of the original features along the components. A biplot can help us interpret the reduced dimensions of the data, and discover relationships between the principal components and original features.

Run the code cell below to produce a biplot of the reduced-dimension data.

In [27]:

# Create a biplot
vs.biplot(good_data, reduced_data, pca)
Out[27]:

<AxesSubplot:title={'center':'PC plane with original feature projections.'}, xlabel='Dimension 1', ylabel='Dimension 2'>
Once we have the original feature projections (in red), it is easier to interpret the relative position of each data point in the scatterplot. For instance, a point the lower right corner of the figure will likely correspond to a customer that spends a lot on 'Milk', 'Grocery' and 'Detergents_Paper', but not so much on the other product categories.

Clustering
In this section, we will choose to use either a K-Means clustering algorithm or a Gaussian Mixture Model clustering algorithm to identify the various customer segments hidden in the data. We will then recover specific data points from the clusters to understand their significance by transforming them back into their original dimension and scale.

K-Means or Gaussian Mixture Model?
From what we know of both models.

Advantages of K-Means clustering:

Simple, easy to implement and interpret results.
Good for hard cluster assignments i.e. when a data point only belongs to one cluster over the others.
Advantages of Gaussian Mixture Model clustering:

Good for estimating soft clusters i.e. we’re not sure if a point belongs to one cluster over another.
Does not bias the cluster sizes to have specific structures in the cluster that may or may not exist.
Gven what we know about the wholesale customer data so far, we’ll chose to use Gaussian Mixture Model clustering over K-Means. This is because there might be some hidden patterns in the data that we may miss by assigning only one cluster to each data point. For example, let’s take the case of the Supermarket customer in our sample: while doing PCA, it had similar and high positive weights for multiple dimensions, i.e. it didn’t belong to one dimension over the other. So a supermarket may be a combination of a fresh produce store/grocery store/frozen goods store.

We’ll choose GMM, so that we don’t miss cases like these.

Creating Clusters
Depending on the problem, the number of clusters that we expect to be in the data may already be known. When the number of clusters is not known a priori, there is no guarantee that a given number of clusters best segments the data, since it is unclear what structure exists in the data — if any. However, we can quantify the “goodness” of a clustering by calculating each data point’s silhouette coefficient. The silhouette coefficient for a data point measures how similar it is to its assigned cluster from -1 (dissimilar) to 1 (similar). Calculating the mean silhouette coefficient provides for a simple scoring method of a given clustering.

In [30]:

n_clusters = [8,6,4,3,2]
from sklearn.mixture import GaussianMixture
from sklearn.metrics import silhouette_score
for n in n_clusters:
    
    # Apply your clustering algorithm of choice to the reduced data 
    clusterer = GaussianMixture(n_components=n).fit(reduced_data)
    # Predict the cluster for each data point
    preds = clusterer.predict(reduced_data)
    # Find the cluster centers
    centers = clusterer.means_
    # Predict the cluster for each transformed sample data point
    sample_preds = clusterer.predict(pca_samples)
    # Calculate the mean silhouette coefficient for the number of clusters chosen
    score = silhouette_score(reduced_data,preds)
    
    print("The silhouette_score for {} clusters is {}".format(n,score))
The silhouette_score for 8 clusters is 0.32618482000675497
The silhouette_score for 6 clusters is 0.18837864109184613
The silhouette_score for 4 clusters is 0.2755484038855516
The silhouette_score for 3 clusters is 0.3042099068049294
The silhouette_score for 2 clusters is 0.42191684646261496
/Users/sajal/.pyenv/versions/3.10.0/envs/ds_portfolio/lib/python3.10/site-packages/sklearn/base.py:450: UserWarning: X does not have valid feature names, but GaussianMixture was fitted with feature names
  warnings.warn(
/Users/sajal/.pyenv/versions/3.10.0/envs/ds_portfolio/lib/python3.10/site-packages/sklearn/base.py:450: UserWarning: X does not have valid feature names, but GaussianMixture was fitted with feature names
  warnings.warn(
/Users/sajal/.pyenv/versions/3.10.0/envs/ds_portfolio/lib/python3.10/site-packages/sklearn/base.py:450: UserWarning: X does not have valid feature names, but GaussianMixture was fitted with feature names
  warnings.warn(
/Users/sajal/.pyenv/versions/3.10.0/envs/ds_portfolio/lib/python3.10/site-packages/sklearn/base.py:450: UserWarning: X does not have valid feature names, but GaussianMixture was fitted with feature names
  warnings.warn(
/Users/sajal/.pyenv/versions/3.10.0/envs/ds_portfolio/lib/python3.10/site-packages/sklearn/base.py:450: UserWarning: X does not have valid feature names, but GaussianMixture was fitted with feature names
  warnings.warn(
Of the several cluster numbers tried, 2 clusters had the best silhouette score.

Cluster Visualization
In [31]:

# Display the results of the clustering from implementation
vs.cluster_results(reduced_data, preds, centers, pca_samples)
Data Recovery
Each cluster present in the visualization above has a central point. These centers (or means) are not specifically data points from the data, but rather the averages of all the data points predicted in the respective clusters. For the problem of creating customer segments, a cluster’s center point corresponds to the average customer of that segment. Since the data is currently reduced in dimension and scaled by a logarithm, we can recover the representative customer spending from these data points by applying the inverse transformations.

In [32]:

# Inverse transform the centers
log_centers = pca.inverse_transform(centers)
# Exponentiate the centers
true_centers = np.exp(log_centers)
# Display the true centers
segments = ['Segment {}'.format(i) for i in range(0,len(centers))]
true_centers = pd.DataFrame(np.round(true_centers), columns = data.keys())
true_centers.index = segments
display(true_centers)
FreshMilkGroceryFrozenDetergents_PaperDelicatessenSegment 08953.02114.02765.02075.0353.0732.0Segment 13552.07837.012219.0870.04696.0962.0

An interesting observation here could be, considering the total purchase cost of each product category for the representative data points above, and referencing the statistical description of the dataset at the beginning of this project, what set of establishments could each of the customer segments represent?

Taking an educated guess,

Segment 0: This segment best represents supermarkets. They spend a higher than median amount on Milk, Grocery, Detergents_Paper and Deli, which are both essential to be stocked in such places.
Segment 1: This segment best represents restaurants. Their spend on Fresh, and Frozen is higher than the median, and lower, but still close to median on Deli. Their spend on Milk, Grocery and Detergents_Paper is lower than median, which adds to our assessment.
Let’s find which cluster each sample point is predicted to be.

In [34]:

# Display the predictions
for i, pred in enumerate(sample_preds):
    print("Sample point", i, "predicted to be in Cluster", pred)
Sample point 0 predicted to be in Cluster 0
Sample point 1 predicted to be in Cluster 0
Sample point 2 predicted to be in Cluster 0
Our guesses for Sample points 0,1, and 2 were restaurants, supermarket and cafe. It seems like we’re close on the predictions for sample points 0 and 2, while incorrect, or rather inconsistent, with our predictions for sample point 1. Looking at the visualization for our cluster in the previous section, it could be that sample 1 is the point close to the boundary of both clusters.

Conclusion and Implications: How to use this knowledge?
In this final section, we will investigate ways that you can make use of the clustered data. First, we will consider how the different groups of customers, the customer segments, may be affected differently by a specific delivery scheme. Then, we will consider how giving a label to each customer (which segment that customer belongs to) can provide for additional features about the customer data.

Companies will often run A/B tests when making small changes to their products or services to determine whether making that change will affect its customers positively or negatively. The wholesale distributor is considering changing its delivery service from currently 5 days a week to 3 days a week. However, the distributor will only make this change in delivery service for customers that react positively.

How can the wholesale distributor use the customer segments to determine which customers, if any, would react positively to the change in delivery service?
Making the change to the delivery service means that products will be delivered fewer times in a week.

The wholesale distributor can identify the clusters to conduct the A/B test on, but the test should be done on one cluster at a time because the two clusters represent different types of customers, so their delivery needs might be different, and their reaction to change will, thus, be different. In other words, the control and experiment groups should be from the same cluster, at a time.

Additional structure is derived from originally unlabeled data when using clustering techniques. Since each customer has a customer segment it best identifies with (depending on the clustering algorithm applied), we can consider ‘customer segment’ as an engineered feature for the data. Assume the wholesale distributor recently acquired ten new customers and each provided estimates for anticipated annual spending of each product category. Knowing these estimates, the wholesale distributor wants to classify each new customer to a customer segment to determine the most appropriate delivery service.

How can the wholesale distributor label the new customers using only their estimated product spending and the* customer segment *data?
To label the new customers, the distributor will first need to build and train a supervised learner on the data that we labeled through clustering. The data to fit will be the estimated spends, and the target variable will be the customer segment i.e. 0 or 1 (i.e. grocery store or restaurant). They can then use the classifier to predict segments for new incoming data.







