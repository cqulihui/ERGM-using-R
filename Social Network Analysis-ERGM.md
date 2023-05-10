# Exponential Random Graph Models
## Specifying and Fitting ERG models with R

* Author: Hui
* Institution: Nanjing Science & Technology University
* Time: May 3, 2023
* Note: This file introduces how to specify and fit ERG models (which assume social selection processes) with R. 

### 1. Model Specification
#### 1.1 Load the packages
```R
library(ergm)
library(dplyr)
library(readxl)
```
When the package `ergm` is loaded, another package `network` will also be loaded. (The package `dplyr` is used to manipulate data. The package `readxl` is used to read data from xlsx files. This pakcage can be replaced with other packages.)
#### 1.2 Construct the network
Before specifying the ERG model, a network should be constructed.
```R
# read node list (with nodal attributes)
nodelist<-read_excel("filepath\\filename.xlsx")

# read edge list
edgelist<-read_excel("filepath\\filename.xlsx")
names(edgelist)<-c("from", "to")

# the network
nettest<-as.network(edgelist, directed=TRUE, vertices=nodelist, multiple=FALSE)
```

**It is very important that the missing ties should be clearly specified.** 
```R
# Indicate that which ties are missing
# For instance, set the last few rows in the adjacency matrix to be "NA".
# Assume that there are 74 nodes in the network.
i=65
while(i<75){
   nettest[i,,add.edges=TRUE]<-NA
   i = i + 1
}

# Show the info about the network
nettest
```
#### 1.3 The dyadic covariates
Set the matrix of dyadic covariates.
```R
# For instance
# dyadic covariate: positive sentment sent to a colleage by an employee
# the data in the excel file should be a matrix
matrixname <- read_excel("filepath\\filename.xlsx")
colnames(matrixname) <- nodelist$id
matrixname <- as.matrix(positive)
```
#### 1.4 Specify the ERG model
When specifying the effects, refer to the help info about ergm terms.
```R
# use the following commond to get info about terms in ERG model
help("ergm-terms")
```

```R
# An example
model01 <- as.formula(lnet ~ 
                        edges + 
                        twopath + 
                        gwidegree(0.693, fixed = TRUE) + 
                        gwodegree(0.693, fixed = TRUE) + 
                        gwesp(0.693, fixed = TRUE) + 
                        gwdsp(0.693, fixed = TRUE) + 
                        dgwesp(0.693, fixed = TRUE, type = "OSP") + 
                        dgwesp(0.693, fixed = TRUE, type = "ISP") + 
                        nodeicov("node.attribute1") + 
                        nodeocov("node.attribute1") + 
                        nodeicov("node.attribute2") + 
                        nodeocov("node.attribute2") + 
                        edgecov(matrixname) + 
                        nodeifactor("node.attribute3") + 
                        nodeofactor("node.attribute3") + 
                        nodematch("node.attribute3"))
# "matrixname" in the formular is a matrix which shows the values of a dyadic covariabte.
```
Refer to the book **Exponential Random Graph Models for Social Networks** to learn why the parameters in gwdsp, gwesp etc. are set to be 0.693 (6.6.4 Social Circuit Models, Page 71). 
But usually, one can choose different values of parameters in gwdsp, gwesp etc. It might be useful to start from a value, like 0.1, and then slightly change the values of the parameters. For instance, one can choose 0.1 and has a first try. Next time, one can change the value to 0.2, and so on. 
Refer to the appendix of the following book:
`Harris, J. K. (2013). _An introduction to exponential random graph modeling_ (Vol. 173). Sage Publications.`
The appendix of the book is available online. Go to the URL: https://us.sagepub.com/en-us/nam/book/introduction-exponential-random-graph-modeling#resources
 

#### 1.5 Missing ties
When there are missing ties, one can choose a **log-likelihood based method (Handcock and Gile, 2007)** and a **bayesian estimation method (Koskinen, Robins and Pattison, 2013)**. 
* A log-likelihood based method is available in R statnet suits. The missing ties should be set to be "NA". Refer to the URL https://statnet.org/workshop-ergm/ergm_tutorial.html#The_statnet_Project for more information. 
* The software MPNet provides a bayesian estiamtion method. Refer to the MPNet user manual to learn how to estimate an ERG model with missing ties. 

`Handcock, M. S., & Gile, K. (2007). Modeling social networks with sampled or missing data. _Center for Statistics in the Social Sciences, Univ. Washington. Available at http://www. csss. washington. edu/Papers_.`

`Koskinen, J. H., Robins, G. L., Wang, P., & Pattison, P. E. (2013). Bayesian analysis for partially observed network data, missing ties, attributes and actors. _Social networks_, _35_(4), 514-527.`

### 2.Model Fitting
#### 2.1 The command
Use the following command to fit an ERG model:
```R
fit01.0 <- ergm(ergmodel)

# show the results
summary(fit01.0)
```

The default  method to estimate an ERG model in R is Importance sampling method. One can change the default method. 
For example, one can choose to use Stochastic Approximation mehod to estimate the parameters in ERG model. 
```R
fit01 <- ergm(ergmodel, control = control.ergm(main.method = "Stochastic-Approximation"))
```
One can use other parameters in ergm() to change the estimation process. Use the following command to find help information:
```R
help("ergm")
```

#### 2.2 If the model does not converge
When **"Stochastic-Approximation" method** is used to estimate the model, one may find that the theta (i.e., the coefficients) are too far away from the MLE (i.e., maximum likelihood estimation). **Then the estimation should be repeated. The last obtained estimate should be used as the initial value for the parameter in the next round of estimation. To do this, use the following command:**
```R
fit01 <- ergm(ergmodel, control = control.ergm(init = coef(fit01))
```

**If reapted estaimations fail, one can try to do conditional estimation. **
>For example, the density of the graphs may be fixed to the denstiy for the observation. This is akin to treating the edge parameter as nuisance parameter that is of no interest for estimation. Other conditions to facilitate convergence include fixing the degree distributions (in- and out-degrees) or treating some ties or actors as exogenous to the model. 

`(Lusher, D., Koskinen, J., & Robins, G. (Eds.). (2013). _Exponential random graph models for social networks: Theory, methods, and applications_. Cambridge University Press.)`

### 3. Model Convergence and GOF
#### 3.1 Model convergence
Use the following command to check the convergence of the model:
```R
mcmc.diagnostics(fit01)
```
The pictures in the result may be difficult to be shown in the "Plot" panel. A user can choose to input the pictures in a pdf file.
```R
pdf("filepath\\filename.pdf")
mcmc.diagnostics(fit01)

# To stop the input process
dev.off()
```
#### 3.2 GOF
Then use the following command to check the gof of the model:
```R
goffit <- gof(fit01)
goffit
```
One can change the burn-in and interval when simulating graphs with gof(). For example, 
```R
gof <- gof(fit01.0, 
       control= control.gof.ergm(MCMC.burnin = 100000,
                                 MCMC.interval = 100000,
                                 nsim = 1000))
```

Refer to the URL https://statnet.org/workshop-ergm/ergm_tutorial.html#The_statnet_Project and Goodreau et al. (2008) for more information. 
`(Goodreau, S. M., Handcock, M. S., Hunter, D. R., Butts, C. T., & Morris, M. (2008). A statnet tutorial. _Journal of statistical software_, _24_, 1-26.)`

> Written with [StackEdit](https://stackedit.io/).
