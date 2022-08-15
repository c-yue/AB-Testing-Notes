
### A/B Testing: Whether to use the new version paywall

#### 1. Notes

Process
- randomly subset the users and show one set the control and one the treatment
    - control group: current paywall
    - treatment group: proposed paywall
- monitor the conversion rate to see which is better

Randomness! 
- isolatee the impact of gthe change made
- reduce the impact of confounding variables

Response variable
- used to meqsure the impact of the change
- should be KPI or directly related to KPI

Factors & variants
- factors: the type of variable changing
    - the paywall color
- variants: particular changes testing
    - red versus blue parwall


#### 2. Choosing experiment unit & response variable
- primary goal: increase revenue
- better metric: conversion rate
- experiment unit: paywall views
    - simple to work with


#### 3. Question: Whether the new version paywall has higher conversion rate

Baseline conversion rate
- conversion rate before running the test
- 0.347
- determine test sensitivity

#### 4. Calculating sample size

**Test Sensitivity**
- what size of impact is meaningful to detect? 1% 20%
- sensitivity: the minimun level of change we want to be able to detect in the test
- exploring in history data, here 10% of the improvement of avg_revenue looks good, 

**Standard Error <=> Standard Deviation**
- SE: eg. the diff between sample mean and the porpulation mean
- SD: the dispersion of the sample
- Standard Error = $\sqrt{\overline{E} * (1 - \overline{E}) / n}$
- where $\overline{E}$ means the observed conversion rate, n means the sample number
    ```
    # Find the number of paywall views 
    n = purchase_data.purchase.count()
    # Calculate the quantitiy "v"
    v = conversion_rate * (1 - conversion_rate) 
    # Calculate the variance and standard error of the estimate
    var = v / n 
    se = var**0.5
    ```

**type 1&2 Error**
type 1 error: Null true but reject - $\alpha$
type 2 error: Null false but accept - $\beta$

**Confidence Level**
- probability not to make type 1 error
- conmmon values: 0.9, 0.95

**Statistical Power**
- probability not to make type 2 error 
- given the null hypothesis false (there is diff between A, B groups), we truely reject it and accpet another hypothesis, the prob to be right 
- $1-\beta$

several values related:
- Confidence Levle, Statistical Power, Standard Error, Test Sensitivity
- sample size up, power up too
- confidence leverl up, power down

**Given confidence level, power, conversion rate of control and test
we can get Sample Size**
```
    given cl, p1, p2 
    => diff sample size 
    => diff power 
    => match the set power 
    => the sample size fit the constraint 
    => the sample size works well
```

to **achieve both higher cl and power** (less possible to make error in both type 1 and type 2), **more samples needed**

```
# Calculate the test power (some details omitted)
def get_power(n, p1, p2, cl): 
    alpha = 1 - cl
    qu = stats.norm.ppf(1 - alpha/2)
    diff = abs(p2 - p1)
    bp = (p1 + p2) / 2
    ...
    power = power_part_one + power_part_two
    return(power)


def get_sample_size(power, p1, p2, cl, max_n=1000000):
    n = 1 
    while n <= max_n:
        tmp_power = get_power(n, p1, p2, cl)

        if tmp_power >= power: 
            return n 
        else: 
            n = n + 100

    return "Increase Max N Value"
```


#### 5. Analyzing the A/B test results

Confirming test results
- Are our groups the same size?
- Do our groups have similar demographics?

Is the result statistically significant?
- **statistically significant**: are the conversion rates diff enough?
- **p-values**: 
    - if the **null hypothesis is true**, the probability to **observe a result as or more extreme** than the the one we observed (observed one is from the test result data).
    - The p-value is the probability that the null hypothesis is true given the data observed.
    - smaller, null hypothesis more not possible
    - accept or reject hypothesis based on the p-values
    - 0.01, very strong evidence against the null hypothesis
    - 0.01 - 0.05, stong
    - 0.05 - 0.1, very weak
    - 0.1, small or no evidence
    ```
    def get_pvalue(con_conv, test_conv, con_size, test_size):  
        lift =  - abs(test_conv - con_conv)
        scale_one = con_conv * (1 - con_conv) * (1 / con_size)
        scale_two = test_conv * (1 - test_conv) * (1 / test_size)
        scale_val = (scale_one + scale_two)**0.5
        p_value = 2 * stats.norm.cdf(lift, loc = 0, scale = scale_val )
        return p_value
    ```
    - confidence intervals
    ```
    def get_ci(value, cl, sd):
    loc = sci.norm.ppf(1 - cl/2)
    rng_val = sci.norm.cdf(loc - value/sd)

    lwr_bnd = value - rng_val
    upr_bnd = value + rng_val 

    return_val = (lwr_bnd, upr_bnd)
    return(return_val)
    ```

Interpreting your test results
- Plotting the distribution
- Plotting the difference distribution