 <h1>Privacy Preserving machine learning for educational data</h1>

<h3>Overview</h3>

<p>This project supports the National Artificial Intelligence Research Resource (NAIRR) initiative and is sponsored under the grant <b>NAIRR240195</b>. 
To learn more about NAIRR, <a href="https://nairrpilot.org/projects/awarded?_requestNumber=NAIRR240195">click here</a><br></p>

<p>We utilize OpenMined’s PySyft as a core component. The purpose of this project is to develop privacy-preserving machine learning models to improve student retention prediction in higher education, while adhering to FERPA (Family Educational Rights and Privacy Act) and GDRP regulations. This framework enables secure, privacy-respecting collaboration across institutions without compromising student data privacy.</p>

<h3>Data Sources</h3>
<ul>
  <li><p><b>Faketucky Dataset:</b> Mock dataset created from the Faketucky dataset.  Used for development and testing on the low-side server.</p></li>
  <li><b>Private Faketucky Dataset:</b> Mocktucky dataset used as the "private" data for testing on the high-side server under strict security protocols.</li>
</ul>
<h3>PySyft Implementation</h3>
<p>This project leverages PySyft, a privacy-preserving machine learning framework. Key PySyft features used include:</p>
<ul>
  <li>Remote Data Science: Enables testing models on private data without access to the data</li>
  <li>Differential Privacy: Adds noise to protect individual privacy while maintaining statistical utility</li>
</ul>   </br>
<p align="center">
  <img src="https://github.com/user-attachments/assets/37bf776b-bc66-49d3-9f66-4dcd81dafd26" width="600" alt="Screenshot 2025-04-22 at 6 17 39 PM" />
</p>

<p>For detailed information on using PySyft in this project, see the documentation in docs/pysyft/.
For general PySyft documentation, visit <a href="https://github.com/OpenMined/PySyft">OpenMined PySyft Documentation</a></p>

<h3>Architecture</h3>
<p>The diagram below outlines the secure data flow and processes between the High Side and Low Side servers used in this project.</br>
<p align = "center">
 <img src="https://github.com/user-attachments/assets/7ad29886-0b06-44c6-b8cd-8c92b0ef0332" 
     alt="Excalidraw Diagram" 
     style="width: 600px; height: 400px;">


</p>
  
<h3>Technical Implementation</h3>
<p>The project uses a dual-server architecture:</p>
<h3>High Side Server (Data-Secure Environment)</h3>

<ul> 
<li>Dell Precision T7610 with encrypted redundant RAID drives</li>
<li>Ubuntu</li>
<li>Python 3.12.3</li>
<li>PySyft 0.9.2</li>
<li>Semi air-gapped with limited connectivity</li>
</ul>

<h3>Low Side Server (Development Environment)</h3>

<ul>
<li>Microsoft Cloud hosted</li>
<li>Ubuntu</li>
<li>Python 3.10.12</li>
<li>PySyft 0.9.2</li>
</ul>

<h3>Citation</h3>
<p>If this work is useful to you in any way, please cite the corresponding paper:
Fields, J., Islam, K. M. S., Thota, R., Chen, V., & Madiraju, P. (under review). 
    A privacy-preserving framework using remote data science for inter-institutional 
    student retention prediction. Journal of Educational Data Mining.</p>
