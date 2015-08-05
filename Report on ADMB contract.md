# Report on contract from ADMB Foundation staff


## Terms of reference
The Joint Research Centre (JRC) fish stock assessment model, developed under the scope of
the a4a Initiative, uses R and ADMB to fit a statistical model to fisheries data. The ADMB 
routines are used to run the minimization required to find the parameters of the model. The 
present contract aims to revise the ADMB code used by the a4a stock assessment framework, 
in particular with regards to: 

1. Revise the ADMB code to identify any bugs and suggest fixes.   
2. Propose improvements to increase the robustness of the fit and the computation of the hessian.    
3. Propose improvements in efficiency in terms of memory usage.      

During the execution of this purchase order, the Contractor will contact by email the JRC 
technical responsible and provide a commented tpl file with the additions/improvements 
suggested. The comments will have to explain in detail all the code changes. 
Within 3 months after the sending date of the purchase order, the Contractor must provide a report. 

The payment will be made within 30 (thirty) days from receipt of the invoice provided the
final version of the report has been accepted by the JRC technical responsible.

## Preliminaries
To familiarize this modeling framework, the documentation for FLa4a was reviewed and examples successfully run.

## Help with a4a.tpl code
On working through the example probles...specifically the shketest and ple4test which had different but similar issues. This can be seen at 9a533685... basically, the key code mod is shown below        
```  
    // init_vector qpar(1,noQpar,1)           
    init_bounded_vector qpar(1,noQpar,-10,10,1)        
    // init_vector vpar(1,noVpar,2)        
    init_bounded_vector vpar(1,noVpar,-10,10,2)        
```
resulted in converged models in both pathological examples--unfortunately results and interpretation becomes invalid since they end up at their bounds (some elements do).

So the question is how to resolve. Well the reason q or variance terms would go large or small on their own is likely a scaling problem so ideally, something in the input data should check that scales are reasonable numerically for computational purposes (incidentally, I changed all instances of 
``` exp()``` to ``` mfexp()``` to give some robustness when say, the optimization tests something ridiculous (e.g., exp(58)). Perhaps nudging these arbitrary bounds to be -15,15 or a little higher would result in enough latitude for the estimation not to fall over, but then I'm less certain about how the vpar vector in particular should be scaled...normally I would put things in the standard deviation scale for what seem to be log-scale parameters[?]...

In the shketest figure below you can see the 2nd vpar parameter settled on the bound (and has a stupidly tiny std error--because of being on the bound).

![image](https://cloud.githubusercontent.com/assets/2715618/8097448/31922980-0f97-11e5-8ff4-d1b7aec2ef4e.png)


Another route would be to put a reasonable penalty on these types of parameters until the last phase (which might be one more than the maximum phase).

Haven't thought much more about "optimum" phasing here but it did strike me as odd to have the vpar in the 1st phase, seems having a base "ballpark" sigma of say 0.2 to start out would be reasonable, then estimate the values based on the data in later phases (easy to implement).

Anyway, all for today. I put the test directories in the inst/admb folder of the admb_dev branch (need to delete later; just put them for examining). The kludgy steps I followed to run a test example were:

1. navigate to the FLa4a/inst/admb directory in a terminal
2. compile a4a.tpl: ```admb a4a```
3. copy the exe into a folder to test, e.g.: ```copy admb.exe shketest```   
4. navigate to shketest ```cd shketest```
5. run the model ```a4a -nox``` (-nox option distills what's most essential in checking convergence progress, imho).







