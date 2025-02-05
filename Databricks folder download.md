To download a folder from **DBFS (Databricks File System)** to your local machine using the Databricks CLI, follow these steps:

### **Step 1: Ensure Databricks CLI is Installed & Configured**
If you haven’t set up the Databricks CLI yet, install and configure it:

1. **Install Databricks CLI** (if not installed):
   ```sh
   pip install databricks-cli
   ```
   
2. **Authenticate Databricks CLI** (if not configured):
   ```sh
   databricks configure --host https://<databricks-instance> --token
   ```
   - Replace `<databricks-instance>` with your Databricks workspace URL (e.g., `https://adb-123456789.0.azuredatabricks.net`).
   - Enter your Databricks **personal access token** when prompted.

---

### **Step 2: Download Folder from DBFS**
Use the following **Databricks CLI command** to copy the folder from DBFS to your local machine:

```sh
databricks fs cp -r dbfs:/path/to/folder /local/path/to/download
```

- Replace `/path/to/folder` with the actual folder path in DBFS.
- Replace `/local/path/to/download` with the local directory where you want to save the files.
- The `-r` flag ensures recursive copying (for directories).

---

### **Example**
If you have a folder in DBFS at **`dbfs:/mnt/data/my_folder`** and you want to download it to your local machine under **`~/Downloads/my_folder`**, run:

```sh
databricks fs cp -r dbfs:/mnt/data/my_folder ~/Downloads/my_folder
```

---

### **Step 3: Verify the Downloaded Files**
After the command runs, check the **`~/Downloads/my_folder`** directory on your local machine to confirm that the files were successfully copied.
