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
      * Java 17 must be properly installed and configured to build and run Utilities-Recon.
      * Download the JDK from [Adoptium Temurin Java 17](https://adoptium.net/temurin/releases/?package=jdk&version=17&os=any&arch=any).
      *  Install Java 17-Complete the installation with default settings.
      *  After installation, Java will typically be installed in:
      *  **Windows:** `C:\Program Files\Eclipse Adoptium\jdk-17.x.x`
      *  **macOS/Linux:** `/Library/Java/JavaVirtualMachines/` or `/usr/lib/jvm/`
      *  Set JAVA\_HOME and Update Path
           *  1. Open **System Properties** → **Environment Variables**
              2. Under "System variables", click **New**:
              3. * **Variable name:** `JAVA_HOME`
              4. * **Variable value:** `C:\Program Files\Eclipse Adoptium\jdk-17.x.x`
              5. Edit the **Path** variable and add: %JAVA_HOME%\bin
      * Verify Installation
          * ```java -version```
          * ```java version "17.0.8" 2023-07-18 LTS Eclipse Adoptium (Temurin)```
       

   








