---
layout: post
title: Using Neural Networks and Genetic Algorithms to Predict Foreign Exchange Rates
cover: /assets/images/Forex2.png
color: blue
categories:
- science
---

A methodology for using machine learning techniques to make predictions on forex markets

---

## Abstract

The purpose of this project was to design a methodology for the use of neural networks to predict the value movements of foreign exchange rates. Neural networks have gained attention due to their ability to make predictions on nonlinear data. This technique has been applied in the area of financial markets, where neural networks have been able to make fairly accurate predictions. In other fields, genetic algorithms have been shown to be capable of improving the results of neural networks by determining the optimal architecture of neural networks. In this project, this approach of using genetic algorithms to design neural network architecture was evaluated for networks being used to make predictions on financial data. It proved successful, with high prediction accuracies for  the British Pound, the Japanese Yen, and the German Mark. Additionally, the networks had the ability to consistently outperform the market in simulations, achieving an average 71.3% accuracy for the German Mark, 68.2% for the British Pound, and 68.4% for the Japanese Yen, all of which are greater than the 50% minimum needed to show predictive ability. It is clear from these results that using neural networks and genetic algorithms provides an adaptable method for currency value prediction.

[1]: #1
[2]: #2
[3]: #3
[4]: #4
[5]: #5
[6]: #6
[7]: #7
[8]: #8
[9]: #9
[10]: #10
[11]: #11
[12]: #12
[lecun2012efficient]: #lecun2012efficient
[glorot2010understanding]: #glorot2010understanding

## Introduction

Neural networks are computer models, implemented in either hardware or software, that are designed to mimic the structure of simple biological systems [\[1\]][1]. This gives them predictive abilities for data which would normally not fit with other models, such as decision tree induction and discriminant analysis [\[2\]][2]. There are many different forms of neural network, such as probabilistic [\[3\]][3] and recurrent networks [\[4\]][4]. While these models have their advantages and disadvantages, backpropagation networks are commonly used, both for general-purpose and more specialized problems [\[5\]][5]. Backpropagation networks function by having the weights between their nodes adjusted based on the error of the outputs. Neural networks can be used for many tasks, as they are able to accurately make predictions based on limited data sets. These tasks include forecasting rainfall [\[6\]][6], predicting the physical measurements of children [\[5\]][5], determining how female frogs will respond to certain calls [\[4\]][4], determining whether an infection is symptomatic of pneumonia [\[8\]][8], predicting financial time series [\[9\]][9], gaining a competitive advantage in emerging markets [\[5\]][5], and forecasting currency exchange rates [\[10\]][10]. This variety of uses shows the versatility of even relatively-simple neural networks for modeling complex data.

<a name="NN"></a>
![A diagram of a simple neural network](https://upload.wikimedia.org/wikipedia/commons/e/e4/Artificial_neural_network.svg)
*Figure 1: A diagram of a simple neural network*

Neural networks are made up of a series of interconnected nodes [\[1\]][1]. These nodes are essentially analogous to neurons in a brain, and the connections between them are analogous to synapses. In a backpropagation network, nodes are arranged into several layers. There is an input layer, which has a number of nodes equal to the number of variables that are being fed into the network. The connections between the nodes are assigned a weight, and this defines how much the values of the nodes on the next layer are influenced by the nodes to which they are connected. Figure [1](#NN) shows a graphical representation of a simple backpropagation network. The circles represent nodes, and the lines are the connections between them. The arrows show the direction in which data flows. Since it only flows forward, this network is a feedforward network. In order to train these networks, the weights of neurons are adjusted based on the error of the network's output when compared to the expected values. Through this process, the network is expected to become more and more accurate. Problems arise, however, when the network is overtrained, and just "memorizes" the expected values instead of creating a valid model. This problem of network design has been facing machine learning researchers since the field's inception.

Neural networks are defined by many properties other than their nodes and weights. One of the most important of these is the activation function. There are two common activation functions: hyperbolic tangent (tanh) and the standard logistical function (sigmoid) [\[14\]][lecun2012efficient]. These two differ in one key property: the former is symmetrical around zero, while the latter is symmetrical around 0.5. For most cases, tanh is preferred, as the sigmoid function may lead to unintentional biases. However, for this experiment, both were included as options, since it is possible for networks with the standard logistical function to perform better in some cases.

Another important property is bias. This measure can be seen as a transformation of the activation function [\[14\]][lecun2012efficient]. So, if the activation function were to yield a result of 0 given a specific input, with a bias of +0.1, it would instead yield 0.1. This can be beneficial by making it easier for the network to approximate the values.

Additionally, the learning rate and momentum of networks must be considered. Both of these are important in determining how quickly the network learns, as well as whether it converges on local minima [\[14\]][lecun2012efficient]. The learning rate is any number between zero and one, and it defines how much the weights are changed at the end of each epoch. A value that is too high can lead to small variations causing the network to change rapidly, whereas one that is too low can prevent the training form having the intended effect. The momentum is also a value between zero and one, and adds this fraction of the previous weight update to the present one. A high value may speed up learning, but may also lead to the system being unstable.

One notable development in neural networks has been that of deep learning. This field is centered on the use of many hidden variables in statistical models, allowing for more complex calculations to be done on data [\[13\]][glorot2010understanding]. While deep learning is not exclusive to neural networks, it is most commonly used with them. The common methodology for deep learning, in which data is passed through various "autoencoders," or smaller neural networks trained to transform the data in a specific way, was not known until 2006; prior to this, it was essentially an accepted fact that "shallow" neural networks were superior for learning [\[1\]][1]. In this study, shallow networks were used, due to their simplicity; however, deep networks are worth mentioning as an alternative architecture due to their popularity.

Genetic algorithms provide a way to alleviate this problem. Genetic algorithms are heuristic search algorithms that operate on the idea of survival of the fittest. This allows the algorithm to determine the optimal solution to a problem [\[4\]][4]. There are a series of generations, each with a group of chromosomes each with its own parameters. This group of chromosomes makes up the population, the size of which is usually limited. To produce the next generation, the parameters are crossed over between organisms as if they were chromosomes, and there is a chance of the parameters mutating. Then, the newly-created organisms compete directly against one another to determine which will continue on to the next generation. Eventually, this is expected to result in an organism with the ideal parameters for its task. This makes them ideal for use in conjunction with genetic algorithms in order to optimize the networks, "pruning" away excess nodes [\[11\]][11]. This process not only makes the network simpler, and thus more efficient, but can also dramatically improve accuracy by making the network only take relevant data into account. This is a tried and proven approach, having yielded good results in predicting rainfall [\[6\]][6], checking for cases of diseases [\[8\]][8][\[11\]][11], and many more situations. Genetic algorithms are a valuable tool in machine learning, and especially neural networks.

<a name="random-walk"></a>
![An example of a random-walk model](https://upload.wikimedia.org/wikipedia/en/3/38/Pi_stock.svg)

*Figure 2: An example of a random-walk model*

In finance, there are two types of market: efficient and inefficient [\[5\]][5]. Efficient markets are those that follow a random-walk model; that is, their movements are entirely unpredictable and random. Figure [2](#random-walk) shows an example of a random-walk model as it would appear in a graph of a stock's value. These markets are typically those that are more well-established, such as the Dow Jones Industrial Average and the Nikkei Average. The other type of market, inefficient markets, is not random, and, through the use of complex models, predictable. These are typically, but not always, newer, or emerging markets, such as those in the Pacific Rim. However, age is not the only factor in the efficiency of markets; for example, foreign currency exchange markets have been shown to be inefficient due to their decentralized nature [\[12\]][12]. Despite their non-random nature, most inefficient markets are still too complex to be modeled effectively using only simple tools, making neural networks ideal. Their ability to use many sources of data has been shown to give them the ability to make accurate, and thus profitable, predictions of market movements [\[5\]][5]. Additionally, past research has shown that the predictions of neural networks can be improved through the inclusion of extramarket data; that is, data from markets external, but related to, the market being studied [\[5\]][5]. While these data are difficult to include using traditional methods, neural networks are able to include them easily; the only change that needs to occur is an increase in the number of input nodes to accommodate these new inputs. This has achieved excellent results in the past; in one study, a researcher was able to make market movement predictions with up to a 62.93% accuracy by including extramarket data [\[5\]][5]. The only problem with this approach is the actual design of the networks; selecting which markets to include can greatly affect the results, but must be done by hand, making the process slow and prone to human error.

Sets of linear data are called time series. Thus, the information given to networks upon which they make predictions are financial time series. These provide a unique problem in neural networks in that one must select how far back in time the data extend. While it is natural to assume that more data automatically equates to higher prediction accuracy, this is not always true. In fact, utilizing data extending more than a few years back in time may adversely affect results. This is known as the Time-Series Recency Effect, and previous tests on financial data have shown that the best results are obtained by using data extending no further backward in time than a decade [\[10\]][10]. This counterintuitive effect makes the design of the networks more difficult, but could be resolved using genetic algorithms.

## Problem

As stated previously, previous attempts to predict the movements of currency exchange rates have been successful, with predictions of Pacific Rim markets achieving up to 62.93% accuracy. However, they required the researcher to select the included variables, which is time consuming, inefficient, and prone to human error. Combined with the Time-Series Recency Effect, designing neural networks to make predictions on such chaotic data as financial time series can be a daunting task. However, by introducing genetic algorithms to prune nodes, an appropriate time scale, as well as meaningful inputs, can be selected automatically. This methodology has not yet been applied to financial time series, and may be able to produce higher prediction accuracy than previous research.

## Hypothesis

If backpropagation neural networks are used in conjunction with genetic algorithms to prune their nodes, then the network will be able to predict market movements with an accuracy exceeding 50%, indicating that it has a competitive advantage.

## Methods

<a name="data"></a>

               GERMANY -- SPOT EXCHANGE RATE, DM/US$
               -------------------------------------

     1-Jan-90                                     ND
     2-Jan-90                                 1.7088
     3-Jan-90                                 1.7208
     4-Jan-90                                 1.6835
     5-Jan-90                                 1.6805
     8-Jan-90                                 1.6688
     9-Jan-90                                 1.6821
    10-Jan-90                                 1.6800
    11-Jan-90                                 1.6820
    12-Jan-90                                 1.6795
    15-Jan-90                                     ND
    16-Jan-90                                 1.6948
    17-Jan-90                                 1.6905
    18-Jan-90                                 1.7085

*Figure 3: An example of the data provided by the Federal Reserve*

The methodology is closely modeled on that of prior research. To allow for easier comparisons to prior research, networks were made using data from the British Pound, German Mark, and Japanese Yen. While the Mark has been superseded by the Euro, there is a greater amount of data available for the Mark. Predictions for the Euro could also be complicated by its very nature; its movements are tied to many economies, whereas the Mark is tied solely to Germany. Tests have also been done on the French Franc, Swiss Franc, and Italian Lira, and so these data may be used if needed. All data were obtained from the archives maintained by the United States Federal Reserve System, which has data extending from the 1970s to the present. An example of the format of this data is shown in Figure [3](#data).

In order to test the hypothesis, the neural networks and genetic algorithms had to be implemented in software. A variety of neural network and genetic algorithm software libraries were examined, and ultimately PyBrain was chosen for the former, and Pyevolve for the latter. Both of these libraries are for the Python programming language, which was chosen due to its simplicity, its support for object-orientation, and its large number of available libraries.

After selecting these libraries, the software had to be written. In order to deal with the complexity of the task, the approach taken was to start with very simple software and to expand it from there. Ultimately, in order to facilitate the use of genetic algorithms, an object-oriented approach was taken, in which each network was an instance of a class. This allowed for the encapsulation of the network's properties. It also made it easy to save individual instances for later testing using Python's built-in libraries.

To evaluate the networks, the measure of Mean Absolute Error (MAE) was chosen. This measure can be defined as the sum of the absolute values of the differences between predicted and actual values. It can be expressed mathematically as

![MAE = \frac{1}{n}\sum\limits_{i=1}^n |f_i - y_i|](http://latex.codecogs.com/svg.latex?%5Cinline%20%5Chuge%20MAE%20%3D%20%5Cfrac%7B1%7D%7Bn%7D%5Csum%5Climits_%7Bi%3D1%7D%5En%20%7Cf_i%20-%20y_i%7C)

where *f<sub>i</sub>* is the predicted value, and *y<sub>i</sub>* is the true value. This measure was chosen because it is relatively simple to compute, and because it can be used to compare the performance of two networks of the same type well.

A series of neural networks with random initial weights and time frames were generated for each type of currency. The networks were defined by the genetic algorithm's chromosomes, with a population size of eighty. The chromosome contained parameters that set the number of hidden nodes, the number of epochs for which the network was trained, the presence of a bias, the momentum, and whether the network used a tanh or sigmoid activation function. The data from each type of currency were then parsed and fed into these networks. The networks were trained on the data for a number of epochs defined by their chromosome. The networks were then tested on a subset of the data that had not been used in their training to determine the accuracy of their predictions. These accuracy measurements were then compared to those of the other neural networks, and the genetic algorithm selected those that would pass down their "genes." These genes controlled the parameters of the network, including how much data should be included, what length of data should be used, and the momentum of the network. This was repeated for several epochs until the results plateaued. The normalized mean square error and the directional prediction accuracy were both recorded. Following the selection of the best predictive network, the network was used in a simulation to test its ability to actually turn a profit. The simulation began by giving ten thousand United States Dollars to both algorithms as initial capital. A buy/hold method, in which stocks were bought at the market price at the beginning of the simulation and then sold at the end, was used as a control, indicating market growth. The neural networks were tested by having the networks make predictions one day in advance of the current simulated time. If the network predicted that the price was going to rise, it would buy currency; if instead it predicted that the price was going to fall, it would sell currency instead.

The source code for all programs used in this study are available in the appendix at the end of this paper.

<a name="code"></a>

    Testing on data:
    out:     [0     ]
    correct: [0     ]
    error:  0.00000000
    out:     [0     ]
    correct: [0     ]
    error:  0.00000000
    out:     [1     ]
    correct: [1     ]
    error:  0.00000000
    out:     [1     ]
    correct: [1     ]
    error:  0.00000000
    All errors: [2.2186712959340957e-31, 9.8607613152626476e-32, 9.8607613152626476e-32, 3.944304526105059e-31]
    Average error: 2.03378202127e-31
    ('Max error:', 3.944304526105059e-31, 'Median error:', 2.2186712959340957e-31)

*Figure 4: An example of PyBrain's output*

This was done on a standard computer; no special equipment was needed. However, the tests would run faster on a machine with better hardware. The libraries used for this project are open source and fully independent of this research. An example of the output of a PyBrain network is shown in Figure [4](#code). This network was trained to recognize XNOR, a type of logic gate, and was able to make predictions with no error. The library is unable to use multithreading, and so individual core speed mattered more than the number of cores of the hardware, which was considered in hardware selection.

## Results

<a name="table:results"></a>

| Currency      | MAE           | Mean Acc.  | Median Acc. | Network Properties                                                   |
|:-------------:|:-------------:|:----------:|:-----------:|:-------------------------------------------------------------------- |
| German Mark   | 0.00848347    | 71.3076%   | 84.7458%    | Bias: False</br> Epochs: 10</br> Momentum: 0.4</br> Function: tanh   |
| British Pound | 0.0169253     | 68.1598%   | 74.5763%    | Bias: False</br> Epochs: 7</br> Momentum: 0.5</br> Function: sigmoid |
| Japanese Yen  | 6.23147143    | 68.4019%   | 72.0339%    | Bias: True</br> Epochs: 7</br> Momentum: 0.1</br> Function: sigmoid  |
*Table 1: The best MAE, the mean and median accuracies, and the properties of the network used for each currency*

<a name="table:buyhold"></a>

| Currency      | Buy/Hold  | Prediction |
|:-------------:|:---------:|:----------:|
| German Mark   | $8469.54  | $9040.24   |
| British Pound | $10648.08 | $11041.07  |
| Japanese Yen  | $9220.76  | $10735.13  |
*Table 2: Comparison of buy/hold and predictive profits*

<a name="mark_results"></a>
![The predicted and actual values of the German Mark](/assets/images/mark2_2.svg)

*Figure 5: The predicted and actual values of the German Mark*

<a name="gbp_results"></a>
![Predictions for the British Pound](/assets/images/gbp3_2.svg)

*Figure 6: Predictions for the British Pound*

<a name="yen_results"></a>
![Predictions for the Japanese Yen](/assets/images/yen2.svg)

*Figure 7: Predictions for the Japanese Yen*

The program was run four times for each currency, with the result that most closely modeled the results being taken for analysis. This network was then run again an arbitrary number of times for the purpose of examining data. The MAE and movement prediction accuracy, of these tests can be seen in Table [1](#table:results). Then, in order to better gauge the predictive accuracies of the networks, the selected network was then run eight times and the average accuracy was taken.

Then, this same network was used in the simulation. The results can be seen in Table [2](#table:buyhold).

## Discussion/Conclusion

It is clear from these results that the hypothesis was supported. The movement prediction accuracy of the networks was impressive; previous studies seemed to only reach accuracies in the 60-70% range, this methodology reached accuracies well into the 80-90% range. This suggests that these neural networks have a clear predictive advantage, since anything significantly above 50% accuracy in this measurement suggests that the methodology is better than random, as predicted in the hypothesis.

The Japanese yen was handled strangely by the networks. For one thing, it had an extremely high MAE; most of the time, this value should be between zero and one, but the network continuously generated values between five and seven. In predicting values, as seen in Figure [7](#yen_results), the network would return only one value for the whole dataset. This is quite probably because the network had both a bias and an extremely low momentum, the combination of which could lead to a local minimum. It is still unclear why this state was continually selected for by the genetic algorithm.

However, another notable part of the tests was the volatility of the prediction accuracy. The highest and lowest accuracies are both included in Table [1](table:results). Take, for instance, the results for the pound; while its highest value is 94% prediction accuracy, the lowest value was 30%, too low even to have any predictive ability. This is an important consideration in using this methodology for making predictions.

In the simulation, the networks performed well. The networks' performance were compared to a buy-hold strategy. This strategy is the simplest investment strategy possible, in which an investor simply buys shares and then cashes out at the end of the period. By comparing the network to this strategy, it allows for control of against the market's growth. As shown in Table [2](#table:buyhold), the network consistently outperformed the market, although not necessarily making a profit, as seen in the case of the mark. This suggests that this methodology provides an effective technique for investors seeking to gain money from an investment.

In order to determine the significance of the movement prediction results, a Wilcoxon Signed-Rank Test was conducted. This test was chosen due to the small sample size of the data. The test compared these results with a hypothetical population representing the null hypothesis, in which truly random choices were made, resulting in a 50% accuracy rate. For each of the currencies, the median prediction accuracy was far greater than 50%, and the *p*-value for each currency was less than 0.05, showing that the results are statistically significant. The specific *p*-values are recorded in Table [3](#table:pval).

These results suggest that combining neural networks and genetic algorithms provides an effective methodology for predicting financial time series. The predictive accuracy of the networks was well above the 50% necessary to be more effective than a randomness. Additionally, this suggests that the currency markets tested, at least in the time frame from which data were taken, are not efficient. This leaves the possibility of further study in this area in the future.

<a name="table:pval"></a>

| Currency      | *p*-value  |
|:-------------:|:----------:|
| German Mark   | 0.03148953 |
| British Pound | 0.04548447 |
| Japanese Yen  | 0.02126124 |
*p-values for each currency. All are &le; 0.05*

## Significance and Application

This research introduces a novel approach to predicting market movements. Foreign exchange markets are already established to be inefficient, making them ideal for predictions. With genetic algorithms to automate the process of designing the networks, this research provides the opportunity to create an approach to market predictions that can be applied nearly universally. This gives it the potential to create a powerful tool for investors who wish to enter the volatile currency exchange market, or even other markets. The movements of markets once considered too efficient to accurately forecast may be predictable using this methodology, since it does not rely on any measure of linearity. It is clear that this methodology can be widely applied and can be applied to valuable tools for investors and economists alike.

## Future Studies

From this starting point, this research can be extended to other areas. One obvious place is into other currencies. For example, Bitcoin, a growing virtual currency, is known to be a very volatile commodity. Perhaps this methodology could be used to make it a safer, more predictable investment, or even to detect any underlying inefficiencies in the currency. It could also be applied to currencies from different time periods, including more recent data.

Another potential way to give more meaningful results would be to allow more epochs, both for the genetic algorithm and for the neural network. This could not be done due to hardware limitations, but more thorough testing could be done. This could potentially lead to higher and more consistent accuracy. In addition, this could lead to the Japanese yen network escaping its local minimum. Expanding on this, different architectures and activation functions could be used; perhaps deep learning could lead to more accurate and less volatile predictions.

Genetic algorithms are not the only optimization algorithms of their kind. There are numerous other algorithms that can serve the same purpose, such as particle swarm optimization and ant colony optimization. These could be used instead of genetic algorithms, and could potentially lead to improved results.

Finally, in the future, a similar methodology could include extramarket data. This has been shown in other studies to improve the prediction capabilities of neural networks, and so by combining this with the genetic algorithm, it is possible to improve the networks' predictive capabilities.

## Appendix

The code used to run these experiments is available on github.

* [Main Program](https://github.com/colatkinson/science_research/blob/master/ga_and_nn.py)

* [Simulation](https://github.com/colatkinson/science_research/blob/master/simulation.py)

## Bibliography

1. <a name="1" href="https://doi.org/10.2307/2584344">Gupta, A. & Lam, M. S. (1996). Estimating missing values using neural networks. *The Journal of the Operational Research Society, 47*, 229-238.</a>

2. <a name="2" href="https://doi.org/10.2307/2584215">Curram, Stephen P. & Mingers, John (1994). Neural Networks, Decision Tree Induction and Discriminant Analysis: An Empirical Comparison. *The Journal of the Operational Research Society, 45*, 440-450.</a>

3. <a name="3" href="https://doi.org/10.1016/j.neunet.2009.05.003">"Hojjat Adeli & Ashif Panakkat" ("2009"). "A probabilistic neural network for earthquake magnitude prediction ". *"Neural Networks ", "22"*, "1018-1024".</a>

4. <a name="4" href="https://doi.org/10.1098/rspb.1998.0293">Phelps, S. M. & Ryan, M. J. (1998). Neural networks predict response biases of female túngara frogs. *Proceedings of the Royal Society of Biology, 265*, 279-285.</a>

5. <a name="5" href="https://www.jstor.org/stable/40398437">Walczak, Steven (1999). Gaining Competitive Advantage for Trading in Emerging Capital Markets with Neural Networks. *Journal of Management Information Systems, 16*, 177-192.</a>

6. <a name="6" href="https://doi.org/10.1016/j.eswa.2007.08.033">"M. Nasseri, K. Asghari & M.J. Abedini" ("2008"). "Optimized scenario for rainfall forecasting using genetic algorithm coupled with artificial neural network ". *"Expert Systems with Applications ", "35"*, "1415-1421".</a>

7. <a name="7" href="https://doi.org/10.1016/j.apergo.2012.01.007">"Salah R. Agha & Mohammed J. Alnahhal" ("2012"). "Neural network and multiple linear regression to predict school children dimensions for ergonomic school furniture design ". *"Applied Ergonomics ", "43"*, "979-984".</a>

8. <a name="8" href="https://doi.org/10.1016/S0933-3657(03)00065-4">Heckerling, P. S., Gerber, B. S., Tape, T. G. & Wigton, R. S. (2004). Use of genetic algorithms for neural networks to predict community-acquired pneumonia. *Artificial Intelligence in Medicine, 30*, 71-84.</a>

9. <a name="9" href="https://doi.org/10.1016/j.asoc.2008.08.001">Yu, L., Wang, S. & Lai, K. K. (2009). A neural-network-based nonlinear metamodeling approach to financial time series forecasting. *Applied Soft Computing, 9*, 563-574.</a>

10. <a name="10" href="https://www.jstor.org/stable/40398510">Walczak, S. (2001). An empirical analysis of data requirements for financial forecasting with neural networks. *Journal of Management Information Systems, 17*, 203-222.</a>

11. <a name="11" href="https://doi.org/10.1016/j.neunet.2011.06.003">"Dimitrios Mantzaris, George Anastassopoulos & Adam Adamopoulos" ("2011"). "Genetic algorithm pruning of probabilistic neural networks in medical disease estimation ". *"Neural Networks ", "24"*, "831-835".</a>

12. <a name="12" href="https://doi.org/10.1016/0261-5606(94)90012-4">Flood, M. D. (1991). Market structure and inefficiency in the foreign exchange market. *Journal of International Money and Finance, 13*, 131-158.</a>

13. <a name="glorot2010understanding" href="http://machinelearning.wustl.edu/mlpapers/paper_files/AISTATS2010_GlorotB10.pdf">Glorot, X., & Bengio, Y. (2010). Understanding the difficulty of training deep feedforward neural networks. In *International conference on artificial intelligence and statistics* (pp. 249–256).</a>

14. <a name="lecun2012efficient" href="https://doi.org/10.1007/3-540-49430-8_2">LeCun, Y. A., Bottou, L., Orr, G. B., & Müller, K.-R. (2012). Efficient backprop. In *Neural networks: Tricks of the trade* (pp. 9–48). Springer.</a>
