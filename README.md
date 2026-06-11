# Phishing Email Detection

![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)
![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-Machine%20Learning-orange.svg)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Processing-lightgrey.svg)

This is a machine learning pipeline I built to detect phishing emails for a course project. Phishing attacks rely heavily on manipulating text to trick users, so I built an NLP model that was both highly accurate and highly interpretable to get a look under the hood and see *why* it flagged an email phishing or legitimate.

## The Data
Link to dataset: https://www.kaggle.com/datasets/naserabdullahalam/phishing-email-dataset/data (Download zip file only)

I combined several open-source datasets (like Enron, Nazario, and SpamAssassin) to create a single dataset of about 82,000 emails. 

One of the first things I noticed during EDA (Exploratory Data Analysis) was that a few emails were massive (some were system error logs over 4 million characters long). I had to truncate the dataset at the 99th percentile for text length. The extreme outliers would have crashed the TF-IDF vectorizer later on if not addressed.

<img width="1031" height="547" alt="image" src="https://github.com/user-attachments/assets/9843d437-c4a7-4f72-86b6-5f4a3daa39b7" />
<img width="1023" height="547" alt="image" src="https://github.com/user-attachments/assets/e6c4b968-eb89-4f85-9467-340ae927a2a5" />

## My Approach

### 1. Smart Preprocessing
Before cleaning, its important to extract key information which would have been otherwsie losed. Things like money symbols (`$`), phone numbers, and a high ratio of ALL CAPS are massive red flags. 

I extracted those into custom boolean features *first*. After that, I ran the text through a standard NLP cleaning pipeline: stripping HTML tags, removing a custom list of 275+ internet stop-words, and lemmatizing the words. 

You can clearly see the difference in vocabulary between the two classes after cleaning:

<img width="1990" height="802" alt="image" src="https://github.com/user-attachments/assets/f2436c2c-b3f9-47d7-97df-2552c83a6bf5" />

### 2. Modeling
* **Vectorization:** I used TF-IDF to turn the text into a numerical matrix, capping it at 5,000 features (unigrams and bigrams) to keep the model memory-efficient.
* **Clustering (Unsupervised):** I ran K-Means to see if the data naturally grouped together. Plotting the Elbow Method showed a natural drop-off at `k=3`. 
<img width="868" height="547" alt="image" src="https://github.com/user-attachments/assets/eb54c3b8-3457-4d82-a071-d45de3dec07e" />

* **Classification (Supervised):** I tested a few baseline models (Naive Bayes, Logistic Regression, Random Forest). Linear SVM outperformed the rest, so I ran a Grid Search with 5-fold cross-validation to tune the hyperparameters (`C=1`, `class_weight='balanced'`, `loss='squared_hinge'`).

## The Results
The tuned Linear SVM performed incredibly well on the unseen test set:
* **F1-Score:** 98%
* **Precision / Recall:** 98%

For this project, I built a machine learning filter that catches phishing emails with a 98% success rate. I specifically extracted scam behaviors like money symbols and ALL CAPS before running the rest of the emails through a text-cleaning pipeline. I ultimately went with a tuned Linear SVM model because it didn't just give me high accuracy; it allowed me to actually look under the hood and see exactly which words were triggering the spam filter, proving that it learned real security rules instead of just guessing.
