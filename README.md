# Project 1 (2018311639 Yeo Hyunseung)

Built-in modules csv, math, Counter are used. Also, matplotlib is used for plotting the learning curve at TASK 3.

    import csv
    import math
    from collections import Counter
    import matplotlib.pyplot as plt





## TASK 1

First, opened train.csv file and the list of special characters are made. 
    f = open ('train.csv','r')
    train_csv = csv.reader(f)

    train = []
    tokenized_rev = []
    special_chars = " !\"#$%&'()*+,-./:;<=>?@[\]^_`{|}~"


To sort the train data, it is tokenized by splitting each review after converting the words to lowercases and eliminating the special characters.

    for line in train_csv:
        line[1]=line[1].lower()                                 ## Converting into lowercases
        if not line[1] == 'text':                               ## Eliminate the special characters
            for Char in special_chars:
                line[1]=line[1].replace(Char,' ')
            line[1] = line[1].split()
            train.append(line)
            tokenized_rev.append(line[1])                       ## splitting into words
    f.close

    f = open("stopwords.txt",'r')                               ## making stopwords lists
    stopwords = []
    while True:
        line = f.readline().strip()
        if not line:break
        stopwords.append(line)
    f.close
    
'tokenized_words' is the list of total set of tokenized review.
    
    tokenized_words = []                                        ## eliminating stopwords

    for review in tokenized_rev:
        for review_word in review:
            if review_word not in stopwords:
                tokenized_words.append(review_word)

By using count function, top 1000 features are sorted.

    count = Counter(tokenized_words)
    top_thousand = count.most_common(1000)
    selected_f = []
    for words in top_thousand:
        selected_f.append(words[0])                       ## selected features (top 1000 words)

Top 20-50 features are: 'well', 'order', 'told', 'didn', 'going', 'first', 'am', 'love', 'down', 'staff', 'minutes', 'ordered', 'now', 'way', 'day', 'chicken', 'restaurant', 'came', '2', 'nice', 'car', 'take', 'still', 'see', 'asked', 'little', 'store', 'made', 'try', 'want', 'experience'.

    print("TOP 20-50 WORDS: ", selected_f[19:50])                                    ## top 20-50 words 





## TASK 2

All the positive and negative reviews from train.csv are made by list of p_rev_train and n_rev_train. In the train.csv, positive reviews are represented as 5 and negative reviews, 1.

    p_rev_train = []                                        # list (positive reviews)
    n_rev_train = []                                        # list (negative reviews)

    for review in train:
        if review[0] == '5':
            p_rev_train.append(review[1])
        if review[0] == '1':
            n_rev_train.append(review[1])               
            
There are 1970 postive reviews and 2030 negative reviews in train.csv.

To apply Naive rules, we need to find out likelihood of each feature by conditional probability distribute. CPD_pos and CPD_neg shows how many times the features appeared in the positive review and negative review from train.csv each. For example, CPD_pos[19] shows how many times the 20th feature appeared in positive review, which is the word 'well'.

    CPD_pos = []                                
    CPD_neg = []           
    
    for feature in selected_f:               
        cnt_pos = 0
        cnt_neg = 0
        for review_p in p_rev_train:
            cnt_pos += review_p.count(feature)
        for review_n in n_rev_train:
            cnt_neg += review_n.count(feature)
        CPD_pos.append(cnt_pos)
        CPD_neg.append(cnt_neg)


We are going to test with the trained data using test.csv. Repeat the same procedure as train.csv to tokenize the test data.

    f = open ('test.csv','r')                                   
    test_csv = csv.reader(f)

    test = []

    for line in test_csv:
        line[1]=line[1].lower()                                 # Converting into lowercases
        if not line[1] == 'text':                               # Eliminate the special charactors
            for Char in special_chars:
                line[1]=line[1].replace(Char,' ')
            line[1] = line[1].split()                           # Splitting into words
            test.append(line)
    f.close

    for review in test:
        for review_word in review[1]:
            if review_word in stopwords:
                review[1] = [i for i in review[1] if i not in stopwords]

Maximum likelihood estimation is a method of estimation by comparing the likelihood and predicting which class the attribute belongs to. The likelihood is computed as P(F1, F2, ... , F1000 | REVIEW) = P(F1|REVIEW)P(F2|REVIEW)...P(F1000|REVIEW). If the likelihood of a feature is 0, i.e. unseen event, 'Laplace smoothing' is used to prevent these cases. The number of possible case with a feature f_k is 2(to appear or not). Therefore, the likelihood will be 1/(P(REVIEW)+2). 

In python, very small number may not be counted. After computing likelihood, it could be very small number since it is multiplication of numbers less than 1. Therefore, the likelihood is compared by log function. If the likelihood of that the sentence is positive review is larger than the likelihood of negative, the classifier will consider that the review is positive review. Else, it will classify the review negative.

    Estimation = []

    for review in test:
        ll_pos = 0
        ll_neg = 0
        for words in review[1]:
            for features in selected_f:
                if words == features:
                    if CPD_pos[selected_f.index(features)] == 0:
                        ll_pos += math.log10(1/(len(p_rev_train)+2))
                    else:
                        ll_pos += math.log10(CPD_pos[selected_f.index(features)]/len(p_rev_train))
                    if CPD_neg[selected_f.index(features)] == 0:
                        ll_neg += math.log10(1/(len(n_rev_train)+2)) 
                    else:
                        ll_neg += math.log10(CPD_neg[selected_f.index(features)]/len(n_rev_train))
        if ll_pos  > ll_neg :
            Estimation.append('5')
        else:
            Estimation.append('1')

The accuracy formula is: 

$Accuracy = (TruePositive + TrueNegative)/(TruePositive + TrueNegative + FalsePositive + FalseNegative)$

    TRUE_P = 0
    TRUE_N = 0
    FALSE_P = 0
    FALSE_N = 0

    number = 0
    for i in Estimation:
        if i == test[number][0]:
            if i == '5':
                TRUE_P += 1
            else:
                TRUE_N += 1
        else:
            if i == '5':
                FALSE_P += 1
            else:
                FALSE_N += 1
        number += 1

    ACCURACY = (TRUE_P + TRUE_N)/(TRUE_P + TRUE_N + FALSE_P + FALSE_N)
    print('Accuracy: ', ACCURACY*100, '%')

The accuracy of the classifier is 71.39999999999999 %.

## TASK 3


### Training with 10% data set

The number of total review of train.csv was 4000. Now we are going to train again with 10% of this data(400 reviews).

    p_rev10 = []
    n_rev10 = []
    for i in range(400):
        if train[i][0] == '5':
            p_rev10.append(train[i][1])
        else:
            n_rev10.append(train[i][1])                          

    CPD_p10 = []                               
    CPD_n10 = []                                

    for feature in selected_f:               
        cnt_pos = 0
        cnt_neg = 0
        for review_p in p_rev10:
            cnt_pos += review_p.count(feature)
        for review_n in n_rev10:
            cnt_neg += review_n.count(feature)
        CPD_p10.append(cnt_pos)
        CPD_n10.append(cnt_neg)


    Estimation10 = []


    for review in test:
        ll_pos = 0
        ll_neg = 0
        for words in review[1]:
            for features in selected_f:
                if words == features:
                    if CPD_p10[selected_f.index(features)] == 0:
                        ll_pos += math.log10(1/(len(p_rev10)+2))
                    else:
                        ll_pos += math.log10(CPD_p10[selected_f.index(features)]/len(p_rev10))
                    if CPD_n10[selected_f.index(features)] == 0:
                        ll_neg += math.log10(1/(len(n_rev10)+2)) 
                    else:
                        ll_neg += math.log10(CPD_n10[selected_f.index(features)]/len(n_rev10))
        if ll_pos  > ll_neg :
            Estimation10.append('5')
        else:
            Estimation10.append('1')

    TRUE_P = 0
    TRUE_N = 0
    FALSE_P = 0
    FALSE_N = 0

    number = 0
    for i in Estimation10:
        if i == test[number][0]:
            if i == '5':
                TRUE_P += 1
            else:
                TRUE_N += 1
        else:
            if i == '5':
                FALSE_P += 1
            else:
                FALSE_N += 1
        number += 1

    ACCURACY10 = (TRUE_P + TRUE_N)/(TRUE_P + TRUE_N + FALSE_P + FALSE_N)
    print('Accuracy for 10% Data set : ', ACCURACY10*100, '%')

The accuracy is 63.8% with 10% of train data.

By varying the number 10 to 30,50,70 and 400 to 1200,2000,2800, we can obtain the accuracy of classifier with 30%, 50%, 70% of data set. 
As a result, the accuracy for 10%, 30%, 50%, 70%, 100% data is 63.8%, 69.2%, 70.8%, 71.7%, 71.4%.

By using matplotlib, we can show the learning curve of the result. The x lable is amount of training data used in percentage(%), and the y lable shows the accuracy by amount of training data used.

    import matplotlib.pyplot as plt


    data_dict = {'data_x': [10, 30, 50, 70, 100], 'data_y': [ACCURACY10, ACCURACY30, ACCURACY50, ACCURACY70, ACCURACY]}
    plt.xlabel('Amount of training data used (%)')
    plt.ylabel('Accuracy')
    plt.plot('data_x', 'data_y', data=data_dict)
    plt.show()
    plt.show()
    
