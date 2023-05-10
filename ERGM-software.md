# ERGM-Softwares

* Author: LH
* Institution: Nanjing Science & Technology University
* Date: May 3, 2023

## 1. R
### 1.1 Rstudio
Download the IDE online. 
### 1.2 Posit Cloud 
Go to the URL https://docs.posit.co/other/rstudio-cloud/.
### 1.3Google Colab
Go to this URL https://colab.to/r.
Run the following code:
```R
# Install the packages
install.packages('readxl', repos = 'http://cran.rstudio.com/')

install.packages('dplyr', repos = 'http://cran.rstudio.com/')

install.packages('ergm', repos = 'http://cran.rstudio.com/')

# Load the packages
library(readxl)
library(dplyr)
library(ergm)
```

```R
# Set up Google Drive
install.packages("googledrive")
library("googledrive")

# Check if is running in Colab and redefine is_interactive()
if (file.exists("/usr/local/lib/python3.6/dist-packages/google/colab/_ipython.py")) {
install.packages("R.utils")
library("R.utils")
library("httr")
my_check <- function() {return(TRUE)}
reassignInPackage("is_interactive", pkgName = "httr", my_check)
options(rlang_interactive=TRUE)
}


cat(system('python3 -c "from google.colab import drive\ndrive.mount()"', intern=TRUE), sep='\n', wait=TRUE)
```
Then start programming as in RStudio.

## 2. Pnet Suits
PNet
MPNet
XPNet


> Written with [StackEdit](https://stackedit.io/).
