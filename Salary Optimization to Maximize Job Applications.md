# Building a Salary Optimization Algorithm to Maximize Job Applications

## Introduction
To help employers maximize job applications, Indeed leverages cutting-edge machine learning algorithms to recommend the best salary for each job post. This blog outlines how **Thompson Sampling** and **Bayesian Optimization** were utilized to develop a salary optimization algorithm. Using libraries such as **MABWiser** and **BoTorch**, historical job attributes and application data were analyzed to dynamically suggest optimal salaries. The solution integrates AWS Glue ETL, Redshift, and Spark for data processing, and an API built with MLOps tools like SageMaker, MLFlow, Docker, Kubernetes, and Jenkins ensures scalability and reliability.

---

## Use Case
Employers often struggle to set salaries that attract the highest number of job applications. An ideal salary increases visibility and competitiveness, resulting in better hiring outcomes. This project focuses on:

1. Recommending optimal salaries for job postings.
2. Analyzing historical job attributes (location, industry, role, etc.) and application data.
3. Leveraging reinforcement learning and Bayesian optimization techniques for decision-making.

---

## Data Sources
The primary data sources include:

- **Job Attributes**: Location, industry, job type, company size, and experience level.
- **Application Data**: Historical trends on applications received for different salary ranges.
- **Auxiliary Data**: Market trends, competitor data, and seasonal variations.

The data is extracted, transformed, and loaded using **AWS Glue ETL** pipelines into a **Redshift data warehouse**, where it is processed with **Spark** for feature engineering and model training.

---

## Algorithm Design

### Thompson Sampling
Thompson Sampling is a reinforcement learning algorithm used to balance exploration and exploitation. It is particularly effective in environments with uncertainty, like salary optimization.

```python
from mabwiser.mab import MAB, ThompsonSampling

# Define arms (salary ranges)
arms = ["40k-50k", "50k-60k", "60k-70k"]

# Historical application data (rewards for each arm)
rewards = {
    "40k-50k": [10, 12, 15],
    "50k-60k": [20, 22, 25],
    "60k-70k": [5, 8, 7]
}

# Initialize Thompson Sampling
mab = MAB(arms=arms, learning_policy=ThompsonSampling())

# Fit the model
mab.fit(decisions=list(rewards.keys()), rewards=sum(rewards.values(), []))

# Recommend the best salary range
optimal_salary = mab.predict()
print(f"Recommended Salary: {optimal_salary}")
```

### Bayesian Optimization
Bayesian Optimization is used for global optimization of the salary recommendation system by modeling the relationship between salary and applications.

```python
import torch
from botorch.models import SingleTaskGP
from botorch.acquisition import ExpectedImprovement
from botorch.optim import optimize_acqf
from gpytorch.mlls import ExactMarginalLogLikelihood

# Define training data (salary and applications)
salary_data = torch.tensor([[40], [50], [60], [70]], dtype=torch.float32)
application_data = torch.tensor([[10], [20], [30], [15]], dtype=torch.float32)

# Gaussian Process Model
gp = SingleTaskGP(salary_data, application_data)
mll = ExactMarginalLogLikelihood(gp.likelihood, gp)
mll.train()

# Define acquisition function
EI = ExpectedImprovement(gp, best_f=application_data.max())

# Optimize acquisition function to find the best salary
bounds = torch.tensor([[40], [70]], dtype=torch.float32).T
optimal_salary, _ = optimize_acqf(EI, bounds=bounds, q=1, num_restarts=5, raw_samples=20)
print(f"Optimal Salary: {optimal_salary.item()}")
```

---

## Data Preparation
Data preparation involves:

1. **ETL Pipelines**: Using **AWS Glue ETL** to extract job attributes and application data from multiple sources.
2. **Data Warehouse**: Storing and querying processed data in **Redshift**.
3. **Feature Engineering**: Using **Spark** for preparing features like salary ranges, job categories, and location.

---

## API Design and Deployment
### API for Salary Optimization

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/recommend-salary', methods=['POST'])
def recommend_salary():
    data = request.json
    job_attributes = data['job_attributes']
    # Use pre-trained Thompson Sampling model to predict
    optimal_salary = mab.predict()
    return jsonify({"recommended_salary": optimal_salary})

if __name__ == "__main__":
    app.run()
```

### Deployment with MLOps
1. **Containerization**: Use **Docker** to containerize the API.
2. **Orchestration**: Deploy the container on **Kubernetes** for scalability.
3. **Model Serving**: Use **AWS SageMaker** for hosting the Thompson Sampling and Bayesian Optimization models.
4. **CI/CD**: Automate testing and deployment with **Jenkins**.

---

## Conclusion
By combining **Thompson Sampling** and **Bayesian Optimization**, we developed a powerful algorithm to optimize salaries for job postings. The integration of **MABWiser**, **BoTorch**, and modern MLOps tools ensures scalability and efficiency. Employers can now make data-driven decisions to maximize their job applications, significantly enhancing the hiring process.

