To install Poetry on Windows, follow these steps:

---

### **Option 1: Install Using the Official Installer Script**
1. **Open PowerShell (as Administrator) or Command Prompt**  
   Open a terminal with administrative privileges to avoid permission issues.

2. **Run the Installation Command**  
   Execute the following command in your terminal:
   ```powershell
   (Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | python -
   ```

3. **Verify Installation**  
   After installation, close and reopen your terminal to reload the environment variables. Then verify that Poetry is installed by running:
   ```bash
   poetry --version
   ```

---

### **Option 2: Install Using pip**
If you already have Python installed, you can install Poetry using pip.

1. **Install Poetry**  
   Run the following command in your terminal:
   ```bash
   pip install poetry
   ```

2. **Add Poetry to PATH (if needed)**  
   Ensure the Poetry executable is added to your system PATH. By default, it will be located in the `Scripts` directory of your Python installation.

3. **Verify Installation**  
   Check that Poetry is installed by running:
   ```bash
   poetry --version
   ```

---

### **Common Post-Installation Configuration**
1. **Add Poetry to PATH (If Not Automatically Added)**  
   If Poetry is not recognized, manually add its installation path to your environment variables. The default location is:
   - `%APPDATA%\Python\Scripts` (for pip installs)
   - `%USERPROFILE%\.poetry\bin` (for script installs)

2. **Configure Poetry to Use the Correct Python Version**  
   To set your preferred Python version, run:
   ```bash
   poetry env use <python-executable>
   ```
   Replace `<python-executable>` with the path to your desired Python version, e.g., `python3.10`.

---

### **Additional Notes**
- Ensure Python (>=3.7) is installed on your system before installing Poetry.
- To upgrade Poetry later, run:
  ```bash
  poetry self update
  ```
