Given a prediction \(p\) and a label \(y\), a loss function \(\ell(p,y)\) measures the discrepancy between the algorithm's prediction and the desired output. VW currently supports the following loss functions, with squared loss being the default:

|Loss|Function|Minimizer|Example usage|
|---|---|---|---|---|---|
|Squared|![squared loss function](http://i.imgur.com/2lg4e4O.png "\ell(p,y)=\frac{1}{2}(p-y)^2")|Expectation|Regression<br>_Expected return on stock_|
|Quantile|![quantile loss function](http://i.imgur.com/c8IoTVV.png "\ell(p,y)=\tau(p-y)\mathbb{I}(y \le p) +(1-\tau)(y-p)\mathbb{I}(y \geq p)")|Mean|Regression<br>_What is a typical price for a house?_|
|Logistic|![logistic loss function](http://i.imgur.com/Q6AA6E0.png "\ell(p,y)=\log(1+\exp(-yp))")|Probability|Classification<br>_Probability of click on ad_|
|Hinge|![hinge loss function](http://i.imgur.com/3jvIoEN.png "\ell(p,y)=\max(0,1-yp)")|0-1 approximation|Classification<br>_Is the digit a 7?_|
|Classic|Squared loss without [importance weight aware updates](http://arxiv.org/abs/1011.1576)|Expectation|Regression<br>_Often squared loss performs better._|

To select a loss function in VW see the [[Command line arguments]] guide.  The Logistic and Hinge loss are for binary classification only, and thus all samples must have class "-1" or "1". More information on loss function semantics can be found in this [video course on Online Linear Learning](http://techtalks.tv/talks/online-linear-learning-part-1/57924/).

## Which loss function should I use?

* If the problem is a **binary classification problem** your choices should be Logistic or hinge loss.
**Example:** spam vs non-spam, odds of click vs no-click.  (*Q:* when should hinge-loss be used vs logistic?)
* If the problem is a **regression problem**, meaning the target label you're trying to predict is a real value -- you should be using Squared or Quantile loss.
**Example:** revenue, height, weight.
If you're trying to minimize the mean error, use squared-loss. See: http://en.wikipedia.org/wiki/Least_squares .
If OTOH you're trying to predict rank/order and you don't mind the mean error to increase as long as you get the relative order correct, you need to minimize the error vs the median (or any other quantile), in this case, you should use quantile-loss.
See: http://en.wikipedia.org/wiki/Quantile_regression