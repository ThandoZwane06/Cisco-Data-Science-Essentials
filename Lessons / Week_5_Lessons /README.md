Age vs. Blood Pressure: Linear Modeling and Confounding Variables

A data modeling mini project comparing two datasets, a US population sample and the Yanomami (an isolated Indigenous population in the Amazon), to investigate the relationship between age and blood pressure, and to reason through whether that relationship can be attributed to a single cause.

Built as part of Cisco Networking Academy's Data Science Essentials with Python course, Week 5, which focused on data modeling: using mathematical equations to describe relationships in data.

Background

This week's material covered building linear models to investigate real relationships in data, including the influence of moonlight on lion hunting activity and the impact of poaching on elephant tusk sizes, as well as a quadratic model exploring the relationship between speed and stopping distance for electric bicycles. Data modeling uses mathematical equations to describe relationships between variables, and a model's goodness of fit tells you how well that equation actually explains the pattern in the data.

Using that same approach, I looked at a blood pressure dataset containing two populations, the US and the Yanomami, to check whether age and blood pressure are correlated, and whether that correlation looks the same across both groups.

The models

US Dataset

LinearModel(US_Dataset):
Parameters: slope = 0.74, intercept = 93.44
Equation: y = 0.74x + 93.44
Goodness of Fit (R²): 0.470

The linear model reveals a moderate positive relationship between age and blood pressure (R² = 0.470). As age increases, blood pressure trends upward.

Yanomami Dataset

LinearModel(Yanomami_Dataset):
Parameters: slope = -0.00, intercept = 95.47
Equation: y = -0.00x + 95.47
Goodness of Fit (R²): 0.000

The linear model reveals no relationship between age and blood pressure in the Yanomami dataset. The slope is essentially flat, blood pressure stays roughly constant across all ages.

Show Image

The chart makes the contrast visible at a glance: the US dataset's fitted line rises steadily with age, while the Yanomami dataset's fitted line stays almost perfectly flat around 95 mmHg regardless of age.

Confounding variables

One piece of context given alongside this dataset: the Yanomami people participated in the INTERSALT study, which examined 10,000 individuals across 52 populations in 32 countries, looking at the link between salt consumption and systolic blood pressure.

My reasoning on this:

This data alone does not prove that salt intake causes the blood pressure difference. Comparing the US to the Yanomami introduces too many confounding variables. The Yanomami lifestyle lacks the processed sugars, alcohol, high obesity rates, sedentary habits, and chronic psychological stress common in industrialized Western societies. They also consume far more potassium, which naturally lowers blood pressure. Therefore, while low salt consumption is highly influential, their flatline blood pressure is caused by a combination of these healthy lifestyle factors, not just sodium restriction alone.

What I learned this week
Data modeling as a concept: using mathematical equations (linear, quadratic) to describe and quantify relationships between variables, rather than just eyeballing a chart.
Reading a fitted linear equation: the slope tells you the direction and strength of the relationship (0.74 means blood pressure rises by roughly 0.74 mmHg per year of age in the US dataset), and the intercept is the model's predicted value when the input is zero.
R² (goodness of fit): a measure of how well the model's equation actually explains the variation in the data. The US model's R² of 0.470 shows a real but moderate relationship, while the Yanomami model's R² of 0.000 confirms there's essentially no linear relationship to explain at all, age isn't predicting blood pressure in that population.
Confounding variables: a striking difference between two groups (like the US vs. Yanomami blood pressure trends) doesn't isolate a single cause on its own. Comparing two populations that differ in many ways at once (diet, activity levels, stress, obesity rates) makes it hard to attribute an outcome to any one factor, even one as well-studied as sodium intake.
Tools

Python, pandas, matplotlib
