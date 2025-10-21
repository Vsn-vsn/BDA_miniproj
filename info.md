- Z-score of a data point measures how many standard deviations it is away from the mean:
    - z=(x−μ)/σ, where μ is the mean and σ is the standard deviation.

- The Z-score, which converts data to a common scale, tells us how unusual a value is relative to a normal distribution

### 3-sigma rule for anomaly separation 
- The 3-sigma rule (also called the empirical rule or 68–95–99.7 rule) is a fundamental concept in statistics that describes how data is distributed in a normal (Gaussian) distribution.
- It says that for a normal distribution, about:
        - 68% of data lies within ±1σ
        - 95% within ±2σ
        - 99.7% within ±3σ
- So, only 0.3% (roughly 1 in 333 observations) will naturally fall outside that range — meaning such points are statistically exceptional.

### Statistical outlier
- A statistical outlier is a data point that deviates significantly from the expected distribution of a variable. 
- In our clickstream dataset, this typically refers to navigation pairs (prev &rarr; curr) whose click counts are unusually high or low compared to the majority of navigation events.