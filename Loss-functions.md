VW currently supports these loss functions:

1. Squared loss 
\[\ell(p,y)=\frac{1}{2}(p-y)^2\]
2. Logistic loss
\[\ell(p,y)=\log(1+\exp(-yp))\]
3. Hinge loss
\[\ell(p,y)=\max(0,1-yp)\]
4. \(\tau\)-Quantile loss
\[\ell(p,y)=\tau(p-y)\mathbb{I}(y<p) +(1-\tau)(y-p)\mathbb{I}(y \geq p) \]