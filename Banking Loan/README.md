# H2O generated POJO model WebApp

This project shows a generated Java POJO being called using a REST API from a JavaScript Web application.

The application simulates the experience of a consumer applying for a loan.  
The consumer provides some information about themselves and is either offered a loan or denied.

### Processes

(Front-end)   

1.  Web browser

(Back-end)   

1.  Jetty servlet container

> Note:  Not to be confused with the H2O embedded web port (default 54321) which is also powered by Jetty.


## Files

(Offline)
* build.gradle
* data/loan.csv
* script.R
* script.py

(Front-end)
* src/main/webapp/index.html
* src/main/webapp/app.js

(Back-end)
* src/main/java/org/gradle/PredictServlet.java
* lib/h2o-genmodel.jar (downloaded)
* src/main/java/org/gradle/BadLoanModel.java (generated)
* src/main/java/org/gradle/InterestRateModel.java (generated)


##### Create the gradle wrapper to get a stable version of gradle.

```
$ gradle wrapper
```

##### Build the project.

```
$ ./gradlew build                    # Run R script to generate POJOs
$ ./gradlew build -PusePython=true   # Run Python script to generate POJOs
```

##### Deploy the .war file in a Jetty servlet container.

```
$ ./gradlew jettyRunWar -x generateModels
```

(If you don't include the -x generateModels above, you will build the models again, which is time consuming.)

##### Visit the webapp in a browser.

<http://localhost:8080/>


## Underneath the hood

Make a prediction with curl and get a JSON response.

```
$ curl "localhost:8080/predict?loan_amnt=10000&term=36+months&emp_length=5&home_ownership=RENT&annual_inc=60000&verification_status=verified&purpose=debt_consolidation&addr_state=FL&dti=3&delinq_2yrs=0&revol_util=35&total_acc=4&longest_credit_length=10"
{
  "labelIndex" : 0,
  "label" : "0",
  "classProbabilities" : [
    0.8581645524025798,
    0.14183544759742012
  ],

  "interestRate" : 12.079950220424134
}
```

Notes:

* classProbabilities[1] is .1418.  This is the probability of a bad loan.
* The threshold is the max-F1 calculated for the model, in this case approximately .20.
* A label of '1' means the loan is predicted bad.
* A label of '0' means the loan is not predicted bad.
* If the loan is not predicted bad, then use the interest rate prediction to suggest an offered rate to the loan applicant.


```
$ curl "localhost:8080/predict?loan_amnt=10000&term=36+months&emp_length=5&home_ownership=RENT&annual_inc=60000&verification_status=blahblah&purpose=debt_consolidation&addr_state=FL&dti=3&delinq_2yrs=0&revol_util=35&total_acc=4&longest_credit_length=10"
[... HTTP error response simplified below ...]
Unknown categorical level (verification_status,blahblah)
```


## Data

The original data can be downloaded at <https://www.lendingclub.com/info/download-data.action> and it is all the data 
from 2007 to June 30, 2015.  This does not incorporate the declined loan data set which does not have the same feature set.  
Note that some munging was done to distill the data to what is in this git repository.


## References

The starting point for this project was taken from the h2o and gradle distribution.  
It shows how to do basic war and jetty plugin operations.

1. <https://services.gradle.org/distributions/gradle-2.7-all.zip>
2. unzip gradle-2.7-all
3. cd gradle-2.7/samples/webApplication/customized

