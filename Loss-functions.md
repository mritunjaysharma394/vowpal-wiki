Given a prediction \(p\) and a label \(y\), a loss function ![loss function](http://i.imgur.com/0Vt5OC3.png "\ell(p,y)") measures the discrepancy between the algorithm's prediction and the desired output. VW currently supports the following loss functions, with squared loss being the default:

|Loss|Function|Minimizer|Example usage|
|---|---|---|---|---|---|
|Squared|![squared loss function](http://i.imgur.com/LXWLynq.png "\ell(p,y)=\frac{1}{2}(p-y)^2")|Expectation (mean)|Regression<br>_Expected return on stock_|
|Quantile|![quantile loss function](http://i.imgur.com/naPtxt9.png "\ell(p,y)=\tau(y-p)\mathbb{I}(y \ge p) +(1-\tau)(p-y)\mathbb{I}(y \leq p)")|Median|Regression<br>_What is a typical price for a house?_|
|Logistic|![logistic loss function](http://i.imgur.com/E7WAZzw.png "\ell(p,y)=\log(1+\exp(-yp))")|Probability|Classification<br>_Probability of click on ad_|
|Hinge|![hinge loss function](http://i.imgur.com/Q7SU0Bu.png "\ell(p,y)=\max(0,1-yp)")|0-1 approximation|Classification<br>_Is the digit a 7?_|
|Classic|Squared loss without<br> [importance weight aware updates](http://arxiv.org/abs/1011.1576)|Expectation (mean)|Regression<br>_squared loss often performs better than classic._|

To select a loss function in VW see the [[Command line arguments]] guide.  The Logistic and Hinge loss are for binary classification only, and thus all samples must have class "-1" or "1". More information on loss function semantics can be found in this [video course on Online Linear Learning](http://techtalks.tv/talks/online-linear-learning-part-1/57924/).

## Which loss function should I use?

* If the problem is a **binary classification** (i.e. labels are -1 and +1) your choices should be Logistic or Hinge loss (although Squared loss may work as well). If you want VW to report the 0-1 loss instead of the logistic/hinge loss, add `--binary`.
**Example:** spam vs non-spam, odds of click vs no-click.
* For **binary classification** where you need to know the posterior probabilities, use `--loss-function=logistic --link=logistic`.
* If the problem is a **regression problem**, meaning the target label you're trying to predict is a real value -- you should be using Squared or Quantile loss.
**Example:** revenue, height, weight.
If you're trying to minimize the mean error, use squared-loss. See: http://en.wikipedia.org/wiki/Least_squares .
If OTOH you're trying to predict rank/order and you don't mind the mean error to increase as long as you get the relative order correct, you need to minimize the error vs the median (or any other quantile), in this case, you should use quantile-loss.
See: http://en.wikipedia.org/wiki/Quantile_regression