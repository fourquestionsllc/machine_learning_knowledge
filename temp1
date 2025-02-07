### **Summary of Discussion on Deploying Arize AI for ML Model Monitoring at Bread Financial**  

#### **Key Discussion Points:**  
- **Organizational Structure in Arize AI:**  
  - Teams will generally have their own **organizations**, while **spaces** will be environment-specific (e.g., dev, prod) or project-specific.  
  - Consistency in structure is important, either keeping teams at the org level and environments at the space level or vice versa.  

- **CI/CD & Automation Considerations:**  
  - GraphQL API is likely the best option for automation, allowing creation of objects and pipelines similar to UI actions.  
  - Python SDK can be used, but it is separate from Arize's main API, requiring API keys for authentication.  

- **Integration with Databricks & Delta Sharing:**  
  - Credit risk models currently monitored via **EDH in Databricks** can be integrated with Arize AI.  
  - Options include **extracting data manually** or using **Delta Sharing** to share relevant data tables with Arize.  
  - Creating a **view with only necessary fields** for Delta Sharing is being considered.  

- **Implementation Plan:**  
  1. **Get the Python SDK working first** and test if it handles the required data volumes efficiently.  
  2. If SDK-based integration faces performance issues (e.g., timeouts), evaluate **Delta Sharing** as an alternative.  
  3. Iterate on the most robust approach to move towards a **production-ready deployment**.  

#### **Decisions Made:**  
✅ **Start with Python SDK implementation** for initial testing.  
✅ **Evaluate Delta Sharing** if SDK-based ingestion proves inefficient.  
✅ **Ensure consistency** in organizational structure within Arize AI.  
✅ **Proceed incrementally** to test feasibility before full production deployment.
