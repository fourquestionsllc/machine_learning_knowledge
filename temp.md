**Design Document: Arize AI Organizational Structure for Bread Financial**

---

### **1. Introduction**

This document outlines the recommended organizational structure, security configurations, and deployment considerations for Bread Financial's Arize AI implementation. It addresses concerns about pre-deployment inference runs, access control, and integration strategies to ensure a secure and efficient machine learning monitoring environment.

---

### **2. Organizational Structure**

Arize AI's hierarchy consists of **Accounts, Organizations, and Spaces**, which can be leveraged to align with Bread Financial's operational and security requirements.

- **Account**: Represents the top-level entity encompassing all organizations within Bread Financial.
- **Organizations**: Represent teams or business units, ensuring separation of responsibilities.
- **Spaces**: Subcategories within organizations, structured based on environment (e.g., Dev, Prod) or project-specific needs.

#### **Proposed Structure**

**Account**: Bread Financial

- **Organization: AWS**

  - Space: Dev
  - Space: Prod
  - Space: Project-Specific Spaces (if applicable)

- **Organization: Azure**

  - Space: Dev
  - Space: Prod
  - Space: Project-Specific Spaces (if applicable)

This structure ensures **consistency across teams and environments**, reducing confusion and improving operational efficiency.

---

### **3. Security and Access Control**

To maintain controlled access while preventing unauthorized changes, we recommend the following **Role-Based Access Control (RBAC) policies**:

#### **Role Assignments:**

- **MLOps Team**: Assigned as 'Admin' at the **Organization level** (AWS and Azure), granting full access to all spaces within each organization.
- **Team Members**:
  - Assigned as 'Admin' or 'Member' **only at the Space level**, depending on their responsibilities.
  - **Admin**: Full access within the assigned space.
  - **Member**: Limited access based on specific role requirements.

#### **Managing Change & Preventing Accidental Updates:**

To prevent accidental modifications, we recommend the following controls:

- **Approval Workflow for Key Changes**: Implement governance requiring at least one approval for major changes (e.g., adding/removing inference results, modifying access settings).
- **Versioning & Audit Logs**: Track changes to models and inference results to allow rollback if necessary.
- **Scoped Admin Permissions**: Restrict admin permissions to only required users, minimizing accidental updates.

#### **Integration with Active Directory (AD) Groups:**

Utilizing AD groups ensures that only authorized personnel can access their respective spaces:

- **Res-azure\_prd\_dstk\_AlCOEUser** → AlCOE space
- **Res-azure\_prd\_dstk\_CreditRiskUser** → Credit Risk space
- **Res-azure\_prd\_dstk\_MarketingUser** → Marketing space
- **Res-azure\_prd\_dstk\_FinmodelCOEUser** → Finance Model COE space

This **automates access control** and prevents manual permission management errors.

---

### **4. Pre-Deployment Pipelines & Inference Runs**

To separate **pre-deployment inference runs** from production runs and avoid contamination of production metrics, we propose the following:

#### **Isolation of Pre-Deployment Runs:**

- **Separate Spaces for Pre-Deployment Testing**: Use dedicated **Dev spaces** for pre-deployment inference, ensuring they don’t interfere with production metrics.
- **Tagging & Metadata Management**: Clearly tag inference runs with `"pre-deployment"` vs. `"production"` labels to facilitate filtering.
- **Data Segregation in Arize**: Configure separate ingestion pipelines for pre-deployment vs. production runs, ensuring that test runs do not skew actual performance tracking.

#### **Pre-Deployment Considerations:**

- **Early Issue Detection**: Identify potential model issues before production deployment.
- **Performance Benchmarking**: Establish baseline metrics to compare against post-deployment performance.
- **Compliance & Validation**: Ensure models meet regulatory and internal standards before release.

---

### **5. CI/CD & Automation Considerations**

For automated model deployment and monitoring setup, we recommend:

- **GraphQL API for Automation**: Best suited for object creation and pipeline management, mirroring UI functionalities.
- **Python SDK for Initial Testing**: Useful for integration but requires API keys for authentication.
- **Evaluation of Delta Sharing for Data Integration**: If SDK-based ingestion has performance limitations, **Delta Sharing** will be assessed as an alternative.

#### **Integration with Databricks & Delta Sharing:**

- **Credit risk models monitored via EDH in Databricks** can be integrated with Arize AI.
- **Delta Sharing** can be leveraged to share relevant tables with Arize for efficient data access.
- **Creating a view with only necessary fields** ensures optimized performance and security.

---

### **6. Implementation Plan & Next Steps**

**Decisions Made in Meeting:**
✅ Start with **Python SDK** for initial testing.\
✅ Evaluate **Delta Sharing** if SDK-based ingestion proves inefficient.\
✅ Maintain **consistent organizational structure** within Arize AI.\
✅ Implement **incremental testing** before full production deployment.

**Action Items:**

1. **Test Python SDK** for data ingestion efficiency.
2. **Establish pre-deployment isolation strategies** (dedicated spaces, tagging, data segregation).
3. **Implement RBAC governance** (approval workflows, audit logs, scoped permissions).
4. **Assess Delta Sharing feasibility** for Databricks integration if SDK performance is inadequate.
