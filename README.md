# Project 1 (2018311639 Yeo Hyunseung)

Built-in modules csv, math, Counter are used.

    import csv
    import math
    from collections import Counter






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
            tokenized_rev.append(line[1])                       ## tokenize into words
    f.close

    f = open("stopwords.txt",'r')                               ## making stopwords lists
    stopwords = []
    while True:
        line = f.readline().strip()
        if not line:break
        stopwords.append(line)
    f.close
    
tokenized_words[] is the list of total set of tokenized review.
    
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





# TASK 2

p_rev_train = []                                        ## list (positive reviews)
n_rev_train = []                                        ## list (negative reviews)

for review in train:
    if review[0] == '5':
        p_rev_train.append(review[1])
    if review[0] == '1':
        n_rev_train.append(review[1])                          # There are 1970 postive reviews and 2030 negative reviews in train data

prob_rev = [len(p_rev_train)/4000,len(n_rev_train)/4000]      # prob_rev = [P(R=positive), P(R=negative)]


CPD_pos = []                                # number of times the features are used in positive reviews
CPD_neg = []                                # number of times the features are used in positive reviews

for feature in selected_f:               
    cnt_pos = 0
    cnt_neg = 0
    
    for review_p in p_rev_train:
        cnt_pos += review_p.count(feature)
    for review_n in n_rev_train:
        cnt_neg += review_n.count(feature)
    CPD_pos.append(cnt_pos)
    CPD_neg.append(cnt_neg)




f = open ('test.csv','r')                                   ## TEST WITH 'test.csv'
test_csv = csv.reader(f)

test = []

for line in test_csv:
    line[1]=line[1].lower()                                 ## Converting into lowercases
    if not line[1] == 'text':                               ## Eliminate the special charactors
        for Char in special_chars:
            line[1]=line[1].replace(Char,' ')
        line[1] = line[1].split()
        test.append(line)
f.close


for review in test:
    for review_word in review[1]:
        if review_word in stopwords:
            review[1] = [i for i in review[1] if i not in stopwords]


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


### TASK 3 ###


# Training with 10% data set
'''
p_rev10 = []
n_rev10 = []
for i in range(400):
    if train[i][0] == '5':
        p_rev10.append(review[1])
    else:
        n_rev10.append(review[1])                          


CPD_p10 = []                                # number of times the features are used in positive reviews
CPD_n10 = []                                # number of times the features are used in positive reviews

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
'''
# Training with 30% data set
p_rev30 = []
n_rev30 = []
for i in range(1200):
    if train[i][0] == '5':
        p_rev30.append(review[1])
    else:
        n_rev30.append(review[1])                         


CPD_p30 = []                                # number of times the features are used in positive reviews
CPD_n30 = []                                # number of times the features are used in positive reviews

for feature in selected_f:               
    cnt_pos = 0
    cnt_neg = 0
    
    for review_p in p_rev30:
        cnt_pos += review_p.count(feature)
    for review_n in n_rev30:
        cnt_neg += review_n.count(feature)
    CPD_p30.append(cnt_pos)
    CPD_n30.append(cnt_neg)

Estimation30 = []


for review in test:
    ll_pos = 0
    ll_neg = 0
    for words in review[1]:
        for features in selected_f:
            if words == features:
                if CPD_p30[selected_f.index(features)] == 0:
                    ll_pos += math.log10(1/(len(p_rev30)+2))
                else:
                    ll_pos += math.log10(CPD_p30[selected_f.index(features)]/len(p_rev30))
                if CPD_n30[selected_f.index(features)] == 0:
                    ll_neg += math.log10(1/(len(n_rev30)+2)) 
                else:
                    ll_neg += math.log10(CPD_n30[selected_f.index(features)]/len(n_rev30))
    if ll_pos  > ll_neg :
        Estimation30.append('5')
    else:
        Estimation30.append('1')

TRUE_P = 0
TRUE_N = 0
FALSE_P = 0
FALSE_N = 0

number = 0
for i in Estimation30:
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

ACCURACY30 = (TRUE_P + TRUE_N)/(TRUE_P + TRUE_N + FALSE_P + FALSE_N)
print('Accuracy for 30% Data set : ', ACCURACY30*100, '%')


# Training with 50% data set
p_rev50 = []
n_rev50 = []
for i in range(2000):
    if train[i][0] == '5':
        p_rev50.append(review[1])
    else:
        n_rev50.append(review[1])                         


CPD_p50 = []                                # number of times the features are used in positive reviews
CPD_n50 = []                                # number of times the features are used in positive reviews

for feature in selected_f:               
    cnt_pos = 0
    cnt_neg = 0
    
    for review_p in p_rev50:
        cnt_pos += review_p.count(feature)
    for review_n in n_rev50:
        cnt_neg += review_n.count(feature)
    CPD_p50.append(cnt_pos)
    CPD_n50.append(cnt_neg)

Estimation50 = []


for review in test:
    ll_pos = 0
    ll_neg = 0
    for words in review[1]:
        for features in selected_f:
            if words == features:
                if CPD_p50[selected_f.index(features)] == 0:
                    ll_pos += math.log10(1/(len(p_rev50)+2))
                else:
                    ll_pos += math.log10(CPD_p50[selected_f.index(features)]/len(p_rev50))
                if CPD_n50[selected_f.index(features)] == 0:
                    ll_neg += math.log10(1/(len(n_rev50)+2)) 
                else:
                    ll_neg += math.log10(CPD_n50[selected_f.index(features)]/len(n_rev50))
    if ll_pos  > ll_neg :
        Estimation50.append('5')
    else:
        Estimation50.append('1')

TRUE_P = 0
TRUE_N = 0
FALSE_P = 0
FALSE_N = 0

number = 0
for i in Estimation50:
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

ACCURACY50 = (TRUE_P + TRUE_N)/(TRUE_P + TRUE_N + FALSE_P + FALSE_N)
print('Accuracy for 50% Data set : ', ACCURACY50*100, '%')

# Training with 70% data set
p_rev70 = []
n_rev70 = []
for i in range(2800):
    if train[i][0] == '5':
        p_rev70.append(review[1])
    else:
        n_rev70.append(review[1])                         


CPD_p70 = []                                # number of times the features are used in positive reviews
CPD_n70 = []                                # number of times the features are used in positive reviews

for feature in selected_f:               
    cnt_pos = 0
    cnt_neg = 0
    
    for review_p in p_rev70:
        cnt_pos += review_p.count(feature)
    for review_n in n_rev70:
        cnt_neg += review_n.count(feature)
    CPD_p70.append(cnt_pos)
    CPD_n70.append(cnt_neg)

Estimation70 = []


for review in test:
    ll_pos = 0
    ll_neg = 0
    for words in review[1]:
        for features in selected_f:
            if words == features:
                if CPD_p70[selected_f.index(features)] == 0:
                    ll_pos += math.log10(1/(len(p_rev70)+2))
                else:
                    ll_pos += math.log10(CPD_p70[selected_f.index(features)]/len(p_rev70))
                if CPD_n70[selected_f.index(features)] == 0:
                    ll_neg += math.log10(1/(len(n_rev70)+2)) 
                else:
                    ll_neg += math.log10(CPD_n70[selected_f.index(features)]/len(n_rev70))
    if ll_pos  > ll_neg :
        Estimation70.append('5')
    else:
        Estimation70.append('1')

TRUE_P = 0
TRUE_N = 0
FALSE_P = 0
FALSE_N = 0

number = 0
for i in Estimation70:
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

ACCURACY70 = (TRUE_P + TRUE_N)/(TRUE_P + TRUE_N + FALSE_P + FALSE_N)
print('Accuracy for 70% Data set : ', ACCURACY70*100, '%')
