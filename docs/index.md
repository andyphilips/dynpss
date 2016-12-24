### dynpss
Stata module to dynamically simulate autoregressive distributed lag (ARDL) models.

### Download 
You can download the most recent version of `dynpss` on the project site [here](https://github.com/andyphilips/dynpss/archive/master.zip). This program is part of a suite that also includes `pssbounds` (Philips 2016c), a Stata module to display the necessary critical values to conduct the Pesaran, Shin and Smith (2001) bounds test for cointegration. 

`dynpss` IS STILL UNDER DEVELOPMENT. Feel free to use the program, but you are encouraged to contact the author with any issues or bugs you find. A module of `dynpss` for R is also under development.

### Table of Contents
 * [Description](#description)
 * [Syntax](#syntax)
 * [Required Options](#opt-req)
 * [Additional Options](#opt-add)
 * [Reference](#reference)
 * [Authors](#authors)
 * [Citations](#citations)
 * [Examples](#examples)
 * [Example Papers](#example-papers)

### Description<a id="description"></a>
`dynpss` is a program to produce dynamic simulations of autoregressive distributed lag models (ARDL) of the sort recommended by Pesaran, Shin, and Smith (2001). See Philips (2016b) for a discussion of this approach and examples of the dynamic simulation strategy in practice.

In short, this modeling strategy involves: 1.) determining the order of integration of the dependent variable (which determines whether the resulting ARDL model is run in error-correction form or lagged-dependent form), 2.) ensuring no variable is higher than I(1) or contains a seasonal unit root 3.) using information criterion to identify the best-fitting `lagdiff( )` structure, which is used to purge the model of residual autocorrelation and ensure it is dynamically stable, 4.) for error-correction models, performing the bounds test of Pesaran, Shin, and Smith (2001) and evaluating whether all regressors are all stationary, mixed I(0), I(1)/I(1) and cointegrating, or all cointegrating. Narayan (2005) provides finite sample critical values for this test. 5.) making changes as needed based off the bounds test, and re-running step 4.

`dynpss` is designed to dynamically simulate the effects of a counterfactual change in one weakly exogenous regressor at a point in time, using stochastic simulation techniques. Since the ARDL procedure can produce models that are complicated to interpret, `dynpss` is designed to ease the burden of substantive interpretations through the creation of predicted (or expected) values of the dependent variable (along with associated confidence intervals), which can be plotted to show how a change in one variable "flows" through the model over time. 

`dynpss` first runs a linear regression. Then it takes 1000 (or however many simulations a user desires) draws of the set of parameters from a multivariate normal distribution with means equal to the estimated parameters from the regression and variance equal to the estimated variance-covariance matrix from the regression. In order to re-introduce stochastic uncertainty back into the model when creating predicted values, `dynpss` simulates sigma squared by taking draws from a scaled inverse chi-squared distribution (scaled by the residual degrees of freedom, n - k, as well as the estimated value of sigma squared from the regression). This ensures that draws of sigma squared are between zero and one. Simulated parameters and sigma squared values are then used to create predicted Y-hat values over time by setting all covariates to certain values (typically means) and introducing stochastic uncertainty by taking a draw from a multivariate normal distribution with mean zero and variance sigma-hat squared. The program then obtains means and percentile confidence intervals of the distribution of predicted values at a particular point in time. The stochastic simulation procedure is similar to the one used in Clarify (King, Tomz, and Wittenberg 2000).


### Syntax <a id="syntax"></a>
`dynpss depvar indepvars [, options]`

## Required options<a id="opt-reg"></a>
 * `lags(numlist)` is a numeric list of the number of lags to include for each variable. The number of desired lags is listed in the order in which the variables in `depvar` and `indepvars` appear. For instance, in a model with two weakly exogenous variables, we lag all variables by specifying: `lags(1 1 1)`. Note that at least one lag for `depvar` must always be specified. To estimate a model without a lag for a particular variable, replace the number with a "."; for instance, `lags(1 . 1)`. If a higher number of lags are specified, `dynpss` will add consecutive lags; i.e. `lags(3 . .)` will introduce lags at t-1, t-2, and t-3 into the model.
 * `shockvar(varname)` is a single independent variable from `indepvars` that is to be shocked. It will experience a counterfactual shock of size `shockval(#)` at time `time(#)`.
 * `shockval(#)` is the amount to shock `shockvar(varname)` by. Most commonly, a +/- one standard deviation shock is specified.

## Additional options<a id="opt-add"></a>
 * `diffs(numlist)` is a numeric list of the number of contemporaneous first differences to include for each variable. Note that the first entry (the dependent variable) will always be empty (denoted by a "."), since the first difference of `depvar` cannot appear on the right-hand side of the equation. `dynpss` will issue an error message if any first differences higher than I(1) are specified.
 * `lagdiffs(numlist)` is a numeric list of the number of lagged first differences to include for each variable. For instance, to include a lag at t-2 for `depvar`, a lag at t-1 for the first weakly exogenous variable, and none for the second, specify `lagdiff(2 1 .)`. The current version does not allow for consecutive lagged differences.
 * `level(numlist)` numeric list of variables to appear in levels. IF both `level( )` and `ec` are specified, `dynpss` will issue a warning message.
 * `ec` if specified, `depvar` will be estimated in first differences (i.e. error correction) form.
 * `range(#)` is the length of scenario to simulate (default is 20). `range( )` must always be larger than `time( )`.
 * `sig(#)` significance level for the percentile confidence intervals (default is 95%).
 * `time(#)` scenario time in which the shock occurs to `shockvar( )` (default is 10).
 * `saving(string)` specifies the name of the output file. If no filename is specified, the program will save the results as "dynpss_results.dta".
 * `forceset(numlist)` by default, `dynpss` estimates the ARDL model in equilibrium; all lagged variables and variables appearing in levels are set to their sample means. The first differences and any lagged first differences are set to zero. This option allows the user to change the setting of the lagged (or unlagged) levels of the variables. This could be useful when estimating a dummy variable--for instance--when we wish to see the effect of a movement from zero to one.
 * `sims(numlist)` number of simulations (default is 1000). If confidence intervals are particularly noisy, it may help to increase this number. Note that you may also need to increase the matsize.
 * `burnin(#)` allows program to iterate out so starting values are stable (rarely used). If using the option `forceset( )`, the predicted values will not be in equilibrium at the start of the simulation, and will take some time to flatten out. To get around this, use the `burnin` option to specify a number of simulations to "throw out" at the start. By default, this is 20. Burnins do not change the simulation range or time; to simulate a range of 25 with a shock time at 10, and a burnin of 30, specify: `burnin(30) range(25) time(10)`.
 * `graph` although `dynpss` saves the means of the predicted values and user-specified confidence intervals in saving, users can automatically plot the dynamic results with a spikeplot by using `graph`. As an alternative, by adding the option `rarea`, the program will automatically create an area plot. Predicted means, 75, 90, and 95% confidence intervals are shown.
 * `expectedval` by default, `dynpss` will calculate predicted values of the dependent variable for a given number of simulations. For each simulation, this will contain a systematic component as well as a single draw of the stochastic component. With the `expectedval` option, the program instead calculates expected values of the dependent variable...i.e. the average of 1000 stochastic draws becomes the estimate of the stochastic component for each of the simulations. Predicted values are more conservative than expected values. Note that `dynpss` takes longer to run if calculating expected values.

### Reference<a id="reference"></a>
If you use `dynpss`, please cite:
Philips, Andrew Q. 2016a. "[dynpss: A program to dynamically simulate autoregressive distributed lag (ARDL) models.](http://andyphilips.github.io/dynpss)."

and

Philips, Andrew Q. 2016b. "[Have your cake and eat it too? Cointegration and dynamic inference from autoregressive distributed lag models](http://dx.doi.org/10.13140/RG.2.1.3812.5684)." Working paper.


### Authors<a id="authors"></a>
[Andrew Q. Philips](http://people.tamu.edu/~aphilips/), Texas A&M University, College Station, TX. aphilips [AT] pols.tamu.edu. @andyphilips


### Citations<a id="citations"></a>
Narayan, Paresh Kumar. 2005. "The Saving and Investment Nexus for China: Evidence from Cointegration Tests." Applied Economics 37(17):1979-1990.

Pesaran, M Hashem, Yongcheol Shin and Richard J Smith. 2001. "Bounds testing approaches to the analysis of level relationships." Journal of Applied Econometrics 16(3):289-326.

Philips, Andrew Q. 2016c. "[pssbounds: Stata module to conduct the Pesaran, Shin, and Smith (2001) bounds test for cointegration](http://andyphilips.github.io/pssbounds)." 

Tomz, Michael, Jason Wittenberg and Gary King. 2003. "CLARIFY: Software for interpreting and presenting statistical results." Journal of Statistical Software 8(1):1-30.


### Examples<a id="examples"></a>
Open up the Ura dataset ("supreme court mood replication.dta") from his article, "Backlash and Legitimation: Macro Political Responses to Supreme Court Decisions", available [HERE](http://doi:10.7910/DVN/24765)
```
use supreme court mood replication.dta, clear
```
Since the program uses Stata's matrix capabilities, it helps to increase the total matrix size:
```
set matsize 5000
```
Run the error-correction model that Ura does, and examine what a 1 standard deviation increase to inflation would to to public mood (the dependent variable). Since we want lags for all variables (including the dependent variable), we specify `lags(1 1 1 1 1)`. Since all of the regressors are also to appear in first differences, we add `diffs(. 1 1 1 1)`. We will specify a range of 30 using the `range` option and we want the shock to occur at time t=10 by using the `time` option. To shock inflation at this time, we add `shockvar(inflation)`, and the magnitude of the shock is given by `shockval(2.9)`. We can automatically see a spikeplot of the dynamic simulation results by using the `graph` option. Last, by adding the `ec` option, the program will turn the dependent variable into first-differences:
```
dynpss mood policy unemployment inflation caselaw, ///
lags(1 1 1 1 1) diffs(. 1 1 1 1)                   ///
shockval(2.9) shockvar(inflation) 	           ///
time(10) range(30) graph ec
```
And the spikeplot that is automatically created:
![graph1](https://raw.githubusercontent.com/andyphilips/miscellaneous/master/img/dynpss_g1.png "+1 Standard Deviation Increase in Inflation")
The spikeplot shows point estimates of the mean of the predicted value, and 75, 90, and 95 percent confidence intervals shown in dark to light blue gradients, respectively. Note that the confidence intervals surrounding the predicted values are somewhat bumpy across time. By increasing the number of simulations with `sim( )`, we can add more simulations and smooth things out. Let's also see what an area plot looks like by adding the `rarea` option:
```
dynpss mood policy unemployment inflation caselaw, ///
lags(1 1 1 1 1) diffs(. 1 1 1 1)                   ///
shockval(2.9) shockvar(inflation) 	           ///
time(10) range(30) graph ec sims(5000) rarea
```
![graph3](https://raw.githubusercontent.com/andyphilips/miscellaneous/master/img/dynpss_g2.png "+1 Standard Deviation Increase in Inflation, Area Plot")
We can add lagged first differences by using the `lagdiffs( )` option. Let's add a first difference at the second lag for inflation, and a first difference at the first lag for caselaw:
```
dynpss mood policy unemployment inflation caselaw, ///
lags(1 1 1 1 1) diffs(. 1 1 1 1)                   ///
shockval(2.9) shockvar(inflation) 	           ///
time(10) range(30) graph ec sims(5000)             ///
lagdiffs(. . . 2 1)
```
![graph3](https://raw.githubusercontent.com/andyphilips/miscellaneous/master/img/dynpss_g3.png "+1 Standard Deviation Increase in Inflation, with Lagged First Differences")

While the plots above depict predicted values, we can use the `expectedval` option to instead calculate expected values. Since this takes much longer to simulate, we will lower the number of simulations to 500:
```
dynpss mood policy unemployment inflation caselaw, ///
lags(1 1 1 1 1) diffs(. 1 1 1 1)                   ///
shockval(2.9) shockvar(inflation) 	           ///
time(10) range(30) graph ec sims(500)             ///
lagdiffs(. . . 2 1) expectedval
```
![graph4](https://raw.githubusercontent.com/andyphilips/miscellaneous/master/img/dynpss_g4.png "+1 Standard Deviation Increase in Inflation, Expected Values")

### Example Papers<a id="example-papers"></a>
Use `dynpss` in one of your papers? Let me know (aphilips [AT] pols.tamu.edu) and I will add it to the list below:
