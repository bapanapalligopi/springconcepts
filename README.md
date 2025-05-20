<div align="center">
  <h2>Utilities-Recon</h2>
</div>

<div align="center">
  <h3>1. Introduction</h3>
</div>

* ### 1.1 What is Utilities-Recon?
  * Utilities-Recon is a ``Java 17-based`` project designed to support QR-on-POS reconsettlement jobs. It provides a comprehensive set of POJOs (Plain Old Java Objects), Value Objects (VOs), and utility classes that streamline and standardize the reconciliation and settlement processes. This library serves as a foundational component, enabling efficient and maintainable development across related systems.
<div align="center">
  <h3>2. Application Requirements</h3>
</div>

* ### 2.1 Required Applications and Tools
  *  <table>
    <tr>
      <td>1</td>
      <td>Java</td>
      <td><img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/java/java-original.svg" width="40"/></td>
      <td>JDK 21 - LTS Temurin 21.0.7+6-LTS</td>
      <td>  <a href="https://adoptium.net/temurin/releases/?package=jdk&version=21&os=any&arch=any" target="_blank" style="text-decoration: none;">
     Download
      </a> </td>
    </tr>
    <tr>
        <td>2</td>
       <td>Maven</td>
      <td><img src="https://github.com/user-attachments/assets/bd4ac478-2084-4e59-8464-227c0bf1f482" width="40"/></td>
      <td>Maven 3.9.9</td>
       <td>  <a href="https://maven.apache.org/download.cgi" target="_blank" style="text-decoration: none;">
     Download
      </a> </td>
    </tr>
    <tr>
      <td>3</td>
      <td>Trivy</td>
      <td><img src="https://github.com/user-attachments/assets/629f802b-9e8b-4b20-bdd9-8dd9fe27efc9" width="40"/></td>
      <td>Trivy V0.62</td>
       <td>  <a href="https://trivy.dev/v0.62/getting-started/installation/" target="_blank" style="text-decoration: none;">
     Download
      </a> </td>
    </tr>
    <tr>
      <td>4</td>
      <td>STS/EClipse/Vscode</td>
      <td><img src="https://github.com/user-attachments/assets/59ccd534-16dc-4d06-9406-77bc4dee3352" width="40"/></td>
       <td>STS</td>
       <td>  <a href="https://spring.io/tools" target="_blank" style="text-decoration: none;">
     Download
      </a> </td>
    </tr>
    <tr>
      <td>5</td>
      <td>EclEmma</td>
      <td><img src="https://github.com/user-attachments/assets/473ddeea-054c-4e3b-894c-87326e8d3a2f" width="40"/></td>
      <td>EclEmma 3.9.5 Pulgin in IDLE</td>
       <td>  <a href="https://www.eclemma.org/installation.html#marketplace" target="_blank" style="text-decoration: none;">
       Plugin 
      </a> </td>
    </tr>
    <tr>
      <td>6</td>
       <td>Git</td>
      <td><img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/git/git-original.svg" width="40"/></td>
      <td>Git 2.x.x</td>
       <td>  <a href="https://git-scm.com/downloads" target="_blank" style="text-decoration: none;">
     Download
      </a> </td>
    </tr>
  </table>






* ### 2.2  Applications Setup
  * #### 2.2.1 Java 17 Setup Guide
    * 
      <details>
        <summary>Click to expand for detailed setup</summary>
      
        * Download the JDK from [Adoptium Temurin Java 17](https://adoptium.net/temurin/releases/?package=jdk&version=17&os=any&arch=any).
        * Install Java 17 - Complete the installation with default settings.
        * After installation, Java will typically be installed in:
          * **Windows:** `C:\Program Files\Eclipse Adoptium\jdk-17.x.x`
          * **macOS/Linux:** `/Library/Java/JavaVirtualMachines/` or `/usr/lib/jvm/`
        * Set `JAVA_HOME` and Update Path:
          1. Open **System Properties** → **Environment Variables**
          2. Under "System variables", click **New**:
            3. **Variable name:** `JAVA_HOME`
            4. **Variable value:** `C:\Program Files\Eclipse Adoptium\jdk-17.x.x`
            5. Edit the **Path** variable and add: `%JAVA_HOME%\bin`
        * Verify Installation:
          ```bash
          java -version
          ```
          Expected output:
          ```bash
          java version "17.0.8" 2023-07-18 LTS Eclipse Adoptium (Temurin)
          ```
      
      </details>
    
   * #### 2.2.2 Maven 3.9.9 Setup Guide
      * 
        <details>
          <summary>Click to expand for detailed setup</summary>
          
          * Download the binary zip archive from the [Apache Maven 3.9.9](https://maven.apache.org/download.cgi) website.
          * Extract the archive to a desired location, such as:
            * **Windows:** `C:\Program Files\Apache\Maven\apache-maven-3.9.9`
            * **macOS/Linux:** `/opt/apache-maven-3.9.9` or `~/apache-maven-3.9.9`
          * Set `MAVEN_HOME` and Update Path:
            1. Open **System Properties** → **Environment Variables**
            2. Under "System variables", click **New**:
              3. **Variable name:** `MAVEN_HOME`
              4. **Variable value:** `C:\Program Files\Apache\Maven\apache-maven-3.9.9`
            3. Edit the **Path** variable and add: `%MAVEN_HOME%\bin`
               *(For macOS/Linux: add `export PATH=$MAVEN_HOME/bin:$PATH` to `.bashrc`, `.zshrc`, or `.bash_profile`)*
          * Verify Installation:
            ```bash
            mvn -version
            ```
            Expected output:
            ```bash
            Apache Maven 3.9.9
            Maven home: C:\Program Files\Apache\Maven\apache-maven-3.9.9
            Java version: 17.0.8, vendor: Eclipse Adoptium
            ```
        
        </details>
    * #### 2.2.3 Git Setup Guide
      * <details>
        <summary>Click to expand for detailed setup</summary>
        
        * Download the installer from the [Official Git Website](https://git-scm.com/downloads).
        * Install Git — complete the installation using default settings unless specific customization is required.
        * After installation, Git will typically be installed in:
          * **Windows:** `C:\Program Files\Git`
          * **macOS (via Homebrew):** `/usr/local/git`
          * **Linux (via package manager):** `/usr/bin/git` or similar
        * Add Git to System PATH (usually done automatically by the installer):
          1. Open **System Properties** → **Environment Variables**
          2. Edit the **Path** system variable
          3. Add the path to Git’s `bin` and `cmd` directories:
             * **Windows example:**
               ```
               C:\Program Files\Git\bin  
               C:\Program Files\Git\cmd  
               ```
             * *(On macOS/Linux, ensure Git is in the terminal path using `which git`)*
        * Verify Installation:
          * Open a terminal or command prompt and run:
            ```shell
            git --version
            ```
          * Expected output:
            ```shell
            git version 2.43.0
            ```
      
      </details>
      
  * #### 2.2.4 Spring Tool Suite (STS) Setup Guide
      * <details>
          <summary>Click to expand for detailed setup</summary>
          
          * Download the latest STS version from the [Spring Tools official website](https://spring.io/tools).
          * Choose the distribution appropriate for your operating system and extract or install it:
            * **Windows:** Installer or ZIP archive
            * **macOS/Linux:** tar.gz archive
          * Suggested installation locations:
            * **Windows:** `C:\Program Files\SpringToolSuite4`
            * **macOS/Linux:** `/Applications/SpringToolSuite4.app` or `~/sts-4.x.x.RELEASE`
          * Launch STS and configure workspace:
            1. Start STS by launching `SpringToolSuite4.exe` or the `.app`/shell script
            2. When prompted, set the workspace directory (e.g., `C:\workspace\utilities-recon`)
          * Configure JDK and Maven in STS:
            * **Set JDK 17:**
              1. Go to **Window** → **Preferences** → **Java** → **Installed JREs**
              2. Click **Add**, select **Standard VM**, and set JDK 17 path (e.g., `C:\Program Files\Eclipse Adoptium\jdk-17.x.x`)
              3. Check the box to make it the default
            * **Verify Maven settings (optional):**
              1. Go to **Window** → **Preferences** → **Maven** → **Installations**
              2. Ensure Maven 3.9.9 is either detected or manually added
          * Install recommended STS plugins (if needed):
            * **Spring Boot Tools**
            * **Buildship Gradle Integration** (if using Gradle modules)
          * Verify Setup:
            * Create or import a sample Maven project
            * Build and run to ensure STS recognizes the Java 17 and Maven 3.9.9 setup correctly
        
        </details>
    
  * #### 2.2.5 Trivy Setup Guide  
     * <details>
          <summary>Click to expand for detailed setup</summary>
        
          * Download Trivy from the [official GitHub releases page](https://github.com/aquasecurity/trivy/releases) or install it via a package manager.
          * Choose the appropriate method based on your operating system:
            * **Windows (using .exe file):**
              1. Download the `trivy_*.zip` file for Windows
              2. Extract it to a directory, e.g.:
                 ```
                 C:\Tools\Trivy
                 ```
              3. Add this directory to the system PATH:
                 * Open **System Properties** → **Environment Variables**
                 * Edit the **Path** system variable
                 * Add: `C:\Tools\Trivy`
          * Verify Installation:
            * Open a terminal or command prompt and run:
              ```bash
              trivy --version
              ```
            * Expected output:
              ```text
              Version: 0.50.1
              Vulnerability DB: 2024-05-19
              ```
        
        </details>

  * #### 2.2.6 EclEmma Plugin Setup Guide (Code Coverage in STS)
    *  <details>
        <summary>Click to expand for detailed setup</summary>
      
        * Steps to Install EclEmma in STS:
          1. Open **Spring Tool Suite (STS)**
          2. Navigate to **Help** → **Eclipse Marketplace**
          3. In the **"Find"** field, type:
             ```
             EclEmma Java Code Coverage
             ```
          4. Click **Go**, find the plugin in the results, and click **Install**
          5. Follow the installation prompts and restart STS when prompted
        * Verify Installation:
          * After restarting, right-click any Java class or test class →  
            You should see options like:
            * **Coverage As** → **JUnit Test**
            * **Coverage As** → **Java Application**
        * Optional: Enable Code Coverage View:
          1. Go to **Window** → **Show View** → **Other**
          2. Search for and select **Coverage** under the **Java** category
          3. This will show coverage results and statistics after running tests with coverage
      
      </details>
        


    

  

  

       

   








