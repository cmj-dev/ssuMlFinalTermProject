# Machine Learning Final Term Project

### 20180823 최민제

# Summary

I use question text information and student’s selection rate to train logistic classification model for each data. By five trained model for each experts, I ensemble them to get best result.

With this prediction I could get each of the five experts view when they measure the test quality with question data and they all have different sight on scoring.

# Introduction

This is a competition to find a model for identifying which problem is good, because when students solve a problem and want to look at the data to measure student learning, the quality of the problem is important for quality of data.

So with the images of question and data of students solution, we have to measure is that a good question.

According to Guide for Diagnostic Questions *1 , I try to measure is the question is unambiguous and ask for a single skill/concept.

# Methods

### Github repo : https://github.com/cmj-dev/ssuMlFinalTermProject

To measure question quality, I use ocr to get text data from images of question. And calculate entropy of student’s selection of choices except answer to measure is that ambiguous question.

In the previous term project, I predict that ocr data doesn’t have significant effect beacuse of ocr’s bad accuracy. So, In this time, I use **google cloud api** which has better accuracy. Also, It give us the location of text box like the screenshot below. So I use google cloud api to extract the information from question image.

![Example of google colud API](Machine%20Learning%20Final%20Term%20Project%2021e48f39ee7e43d0b6d569152d11e461/Untitled.png)

Example of google colud API

## 1. preprocessing

### Entropy

If students' choices tended to be biased except for the answer, I thought the question was likely to be unclear. So I used entropy, which allows us to see the bias toward choices other than the answer option, to learn from that.

The expression below shows how I calculate Entropy of choices except answer.

$$
H = \sum p(x)log_2p(x)
$$

And this time, I also add entropy about the answer selection rate. Which could be the predictor to estimate the question difficulty.

### OCR

There will be a relation between the text information and quality of question. So, I use Google Cloud Vision API to get text information from image of questions. And with this text informations, extract text length, alphabet word count, math sign word count, numeric word count, average word length for logistic regression.

And Google Cloud API also give us the text box information. And I use this information to get the variance of text box in X axis and Y axis. With this variances it was expected to use this for assuming question readability.

### Result Dataset

With these processes, I could get the 10-column datasets.

| AnswerProba | ElseEntropy | AnswerEntropy | stdX | stdY | text_length | alphaWordCount | signWordCount | numericWordCount | avg_word_length |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0.443457 | 1.316125 | 0.990755 | 170.009451 | 129.982344 | 148.0 | 27.0 | 5.0 | 1.0 | 5.481481 |
| 0.571429 | 0.650022 | 0.985228 | 165.986461 | 111.304075 | 53.0 | 9.0 | 1.0 | 8.0 | 5.888889 |
| 0.385214 | 1.560775 | 0.961641 | 197.194749 | 133.458086 | 100.0 | 11.0 | 26.0 | 4.0 | 9.090909 |
| 0.808757 | 1.564606 | 0.704063 | 188.986115 | 143.703961 | 60.0 | 11.0 | 5.0 | 4.0 | 5.454545 |
| 0.401408 | 1.568387 | 0.971769 | 190.131950 | 111.250841 | 160.0 | 42.0 | 5.0 | 0.0 | 3.809524 |

## 2. Training

Because the prediction before has significant enough,  I use the logistic regression model. However, this time, I train five model which could represent each experts.

Based on Entropy and OCR dataset, there are 10 predictors to use. However we only have 49 train dataset to train model. So I use model selection method.

- **Subset Selection Method**
    
    First, I apply subset selection method to model. Because there are only 10 predictors, it is enough to use optimal subset selection method. So, I get all combination of predictors and cross validate each combination of predictors with LOOCV.
    
- **L1 Regularization**
    
    I also apply the L1 regularization to get sparse model. I tuned the hyper parameter of regularization term from 0.01 to 100 and got the best lambda for each five model.
    

And with each model selection method, I use ensemble method to get the unbiased model. Because we have five models for each model selection method, I ensemble all of the five models which trained with same model selection methods. And to get the more models, I also ensemble the five models except one. For example, it there is five models (T1, T2, T3, T4, T5) for each experts, I ensemble the T2, T3, T4, T5 model and don’t ensemble the T1. With this ensemble term, I could get the better validation value with less variance between experts’ prediction accuracy.

## 3. Evaluating

To evaluate the model, I follow the method in Guide for Diagnostic Questions *1 , when the left and right question is provided, I calculate each question’s score and compare them. And for each experts T1, T2, T3, T4, T5, get the accuracy and use best accuracy between that five.

# Results

## Subset Selection Method

As I explained at Methods section, I get the best subset selection with the optimal way. With this subset selection, we could understand the brief outline of each expert’s differenct view.

|  | AnswerProba | ElseEntropy | AnswerEntropy | stdX | stdY | text_length | alphaWordCount | signWordCount | numericWordCount | avg_word_length |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | O |  | O | O |  |  | O |  |  |  |
| T2 | O | O | O | O | O | O |  | O |  | O |
| T3 | O | O |  |  |  |  |  |  | O |  |
| T4 | O | O |  | O |  |  |  | O |  |  |
| T5 | O | O |  |  |  | O | O |  |  |  |

![Untitled](Machine%20Learning%20Final%20Term%20Project%2021e48f39ee7e43d0b6d569152d11e461/Untitled%201.png)

![Untitled](Machine%20Learning%20Final%20Term%20Project%2021e48f39ee7e43d0b6d569152d11e461/Untitled%202.png)

And with the subset selected, I get the each model’s accuracy table in public dataset. And with the model ensemble, we could get the model with less variance. However, all models except trained with T4 expert data, couldn’t get the good accuracy on T4’s dataset.

## L1 Regularization

So, I also use L1 regularization to get better score for all experts. With the cross validation method to got the best hyper parameter of regularization term. With this method, I could get the hyper parameter that C=1 for T1, T2, T5 and C=10 for T3, T4. And I train the model with this regularization term and got the coefficients below. As we could see, each model has different coefficients.

|  | AnswerProba | ElseEntropy | AnswerEntropy | stdX | stdY | text_length | alphaWordCount | signWordCount | numericWordCount | avg_word_length |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | -0.381696 | 0.230434 | 0.270120 | -0.831403 | -0.524845 | 0.000000 | 0.004012 | 0.000000 | -0.386122 | -0.766746 |
| T2 | -0.189342 | 0.673956 | 1.006314 | 0.134794 | -0.317528 | 0.000000 | 0.077005 | -0.106850 | -0.140836 | 0.000000 |
| T3 | -1.450799 | 1.976090 | 1.029475 | 0.000000 | 0.000000 | 0.000000 | 1.121267 | -0.336207 | -2.372145 | -0.242680 |
| T4 | -0.373092 | -0.467943 | -0.141692 | -0.654178 | 0.535668 | -3.967957 | 4.339352 | 0.635372 | -0.445553 | 1.429802 |
| T5 | -0.646155 | 0.076294 | 0.132745 | 0.000000 | -0.011642 | 0.000000 | 0.131144 | 0.000000 | 0.000000 | -0.821677 |

![Untitled](Machine%20Learning%20Final%20Term%20Project%2021e48f39ee7e43d0b6d569152d11e461/Untitled%203.png)

![Untitled](Machine%20Learning%20Final%20Term%20Project%2021e48f39ee7e43d0b6d569152d11e461/Untitled%204.png)

![Untitled](Machine%20Learning%20Final%20Term%20Project%2021e48f39ee7e43d0b6d569152d11e461/Untitled%205.png)

And I did ensemble to get less variance model. And differently than before, when we eliminate T1 and ensemble them, It have great score for all experts.

So, I use model which trained with L1 regularization term and ensembled except T1 expert model.

And here is the Test Score of **L1 Regularization W/O T1 Model**

| T1 | T2 | T3 | T4 | T5 |
| --- | --- | --- | --- | --- |
| 0.68 | 0.64 | 0.64 | 0.72 | 0.84 |

We could get **score 0.84** which is the significant score than before.

# Discussion

With this result, we could get the insights that because each experts have different views, it is hard to guess the accurate question quality based on only one expert’s question quality predictions. As we can see with first model selection method, except Answer Rate and Entropy of selection data, we have to use different predictors to get better cross validation results. And by ensemble we can also get the good prediction value despite the absence of exact labels of that experts.

## Some missing points

### Sparsity of dataset

In this dataset there are a lot of data I couldn’t use because of the sparsity of data. For example, I guess that there will be the relation between number of subjects which question included and question’s quality based on whether question ask for a single skill/concept.

As we can see at the bar plot below, only 1.4% of questions got mutiple level 3 subject. And in train dataset, only one question has subject number large than 4. So I couldn’t use this data to train the model.

And also, I tried to use the student’s confidence to measure the ambiguity of questions. However, there are too many nan value I also couldn’t use that for trian data.

![Untitled](Machine%20Learning%20Final%20Term%20Project%2021e48f39ee7e43d0b6d569152d11e461/Untitled%206.png)

# Conclusion

With this project I find that there is some relationship between entropy of choices and question quality and also there is some relationships between text information and question quality however the degree of the relationships is depend on each experts.

In this term project I tried to extract text from question image with Google Cloud Vision API. However, that api couldn’t give the right order of text so the text inform only used to calculate numeric value such as text length or token length. So, In further step we could develop the algorithm to align the text box with right order for question images and apply that text information on Neural Network model.

# References

1. [Instructions and Guide for Diagnostic Questions: The NeurIPS 2020 Education Challenge](https://arxiv.org/abs/2007.12061)
2. [Google Cloud Vision API](https://cloud.google.com/vision/docs/features-list?hl=ko)