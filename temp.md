Here’s a Python program that uses Plotly to visualize the given DataFrame. Assuming your DataFrame is named `df`, the script creates a scatter plot with the `date` column on the x-axis, `interest_income_domestic_loan` on the y-axis, and colors the data points by `requestId`.

```python
import pandas as pd
import plotly.express as px

# Example DataFrame
data = {
    "requestId": ["C-USA", "C-USA", "C-USA", "BNPQY-USA", "BNPQY-USA"],
    "interest_income_domestic_loan": [7606.0, 6781.0, 6374.0, None, None],
    "date": ["2020-03-31", "2020-06-30", "2020-09-30", "2024-03-29", "2024-06-28"]
}
df = pd.DataFrame(data)

# Ensure the 'date' column is in datetime format
df['date'] = pd.to_datetime(df['date'], errors='coerce')

# Create a Plotly scatter plot
fig = px.scatter(
    df,
    x="date",
    y="interest_income_domestic_loan",
    color="requestId",
    title="Interest Income on Domestic Loans Over Time",
    labels={"date": "Date", "interest_income_domestic_loan": "Interest Income"},
)

# Update layout for better visualization
fig.update_layout(
    xaxis_title="Date",
    yaxis_title="Interest Income (Domestic Loan)",
    legend_title="Request ID",
)

# Show the plot
fig.show()
```

### Steps:
1. Ensure that your `date` column is correctly converted to a `datetime` format using `pd.to_datetime()`.
2. Use `plotly.express.scatter` to plot the data with appropriate arguments for `x`, `y`, and `color`.
3. Display the interactive plot using `fig.show()`.

If your data differs, replace the example DataFrame with the actual one from your dataset.
