To use `ssl:xxx.yyy..net:1666` to access projects in the **Obsidian Entertainment (OEI) Editor**, which is commonly used for game development (e.g., *Pillars of Eternity*, *Tyranny*, etc.), you're dealing with a **Perforce (Helix Core)** version control system.

Here's how to connect the OEI Editor to a Perforce server using that address:

---

### ✅ **Step-by-Step: Access Projects via `ssl:xxx.yyy..net:1666` in OEI Editor**

#### 1. **Install Perforce Helix P4V Client**

Download and install the **Helix Visual Client (P4V)** from the official Perforce website:
👉 [https://www.perforce.com/downloads/helix-visual-client-p4v](https://www.perforce.com/downloads/helix-visual-client-p4v)

---

#### 2. **Set Up the Server Connection**

* Open **P4V**.
* In the **Connection Setup** window:

  * **Server:** `ssl:xxx.yyy..net:1666`
  * **User:** (your provided Perforce username)
  * **Workspace (Client):** (choose an existing or create a new one)
* Click **OK**.

If it's your first time connecting via SSL, you'll be asked to **trust the server's fingerprint**. Accept it.

---

#### 3. **Create or Sync a Workspace**

* In P4V:

  * Go to **Connection > Edit Workspace**.
  * Set the **Root** to a local folder where your files will be synced.
  * Define the **View Mapping** (or accept the defaults).
  * Save and **sync** the workspace to download files.

---

#### 4. **Open OEI Editor**

* Launch the **OEI Editor** (sometimes called the Eternity Editor).
* Go to **File > Open Project**.
* Navigate to the folder where your Perforce workspace downloaded the game project files.
* Open the project file (usually ends in `.proj` or is within a specific folder like `GameData/` or `ProjectSettings/`).

---

#### 5. **Enable Perforce Integration in OEI Editor** *(if needed)*

If OEI Editor supports native Perforce integration:

* Go to **Tools > Options > Source Control** (or similar menu).
* Enter:

  * Server: `ssl:xxx.yyy..net:1666`
  * User: (your username)
  * Client: (your Perforce workspace/client name)

---

### ❗ Notes & Troubleshooting

* **SSL Issues:** If you have connection errors due to SSL, try testing connectivity with P4V first, and ensure the certificate is trusted.
* **Firewall/Proxy:** Make sure your firewall or company network allows outbound traffic on port `1666`.
* **Credentials:** If you don’t have a Perforce account, you’ll need one provided by the admin of `xxx.yyy..net`.
