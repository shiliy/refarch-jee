**IMPORTANT:** This tutorial shows how to 'lift & shift' a traditional on-premise WebSphere Application Server workload to run on the WebSphere Application Server as a Service offering in IBM Bluemix.
This tutorial is part of a series of tutorials which were created for the Cloud & Cognitive Activation workshops in 2017 to educate technical sellers on the Cloud & Cognitive patterns. Tutorials are also publicly available [here](https://www.ibm.com/devops/method/content/engagement/sellers).

# Customer Order Services - JavaEE Enterprise Application

## Application Overview

The application is a simple store-front shopping application, built during the early days of the Web 2.0 movement.  As such, it is in major need of upgrades from both the technology and business point of view.  Users interact directly with a browser-based interface and manage their cart to submit orders.  This application is built using the traditional [3-Tier Architecture](http://www.tonymarston.net/php-mysql/3-tier-architecture.html) model, with an HTTP server, an application server, and a supporting database.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/imgs/apparch-pc-phase0-customerorderservices.png">
</p>

There are several components of the overall application architecture:
- Starting with the database, the application leverages two SQL-based databases running on [IBM DB2](https://www.ibm.com/analytics/us/en/technology/db2/).
- The application exposes its data model through an [Enterprise JavaBean](https://en.wikipedia.org/wiki/Enterprise_JavaBeans) layer, named **CustomerOrderServices**.  This components leverages the [Java Persistence API](https://en.wikibooks.org/wiki/Java_Persistence/What_is_JPA%3F) to exposed the backend data model to calling services with minimal coding effort.
  - As of the [WebSphere Application Server](http://www-03.ibm.com/software/products/en/appserv-was) Version 9 build, the application is using **EJB 3.0** and **JPA 2.0** versions of the respective capabilities.
- The next tier of the application, named **CustomerOrderServicesWeb**, exposes the necessary business APIs via REST-based web services.  This component leverages the [JAX-RS](https://en.wikipedia.org/wiki/Java_API_for_RESTful_Web_Services) libraries for creating Java-based REST services with minimal coding effort.
  - As of the [WebSphere Application Server](http://www-03.ibm.com/software/products/en/appserv-was) Version 9 build, the application is using **JAX-RS 1.1** version of the respective capability.
- The application's user interface is exposed through the **CustomerOrderServicesWeb** component as well, in the form of a [Dojo Toolkit](#tbd)-based JavaScript application.  Delivering the user interface and business APIs in the same component is one major inhibitor our migration strategy will help to alleviate in the long-term.
- Finally, there is an additional integration testing component, named **CustomerOrderServicesTest** that is built to quickly validate an application's build and deployment to a given application server.  This test component contains both **JPA** and **JAX-RS**-based tests.  

### Step 0: Prerequisites

The following are prerequisites for completing this tutorial:
- Bluemix Services:
  - [WebSphere Application Server Version 9](https://console.bluemix.net/catalog/services/websphere-application-server) - Referred to as _WASaaS_ throughout the rest of the tutorial
  - [DB2 on Cloud SQL DB](https://console.bluemix.net/catalog/services/db2-on-cloud-sql-db-formerly-dashdb-tx)
    - NOTE: Your Bluemix account may not be able to provision the desired Db2 on Cloud SQL DB (formerly dashDB TX) service instances as requested below. If that is the case, your lab proctor has been provided with shared credentials for you to use for this lab.
- Command line tools:
  - [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
  - VPN Client for connectivity to WASaaS private network
    - [Windows 64-Bit (OpenVPN)](https://swupdate.openvpn.org/community/releases/openvpn-install-2.3.11-I001-x86_64.exe)
    - [Windows 32-Bit (OpenVPN)](https://swupdate.openvpn.org/community/releases/openvpn-install-2.3.11-I001-i686.exe)
    - [Linux (OpenVPN)](https://openvpn.net/index.php/access-server/download-openvpn-as-sw.html)
    - [Mac (Tunnelblick)](https://tunnelblick.net/)
  - SSH capability
    - Windows users will need [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) or [OpenSSH](https://www.openssh.com/)
  - [WebSphere Application Server Migration Toolkit for Application Binaries](https://developer.ibm.com/wasdev/downloads/#asset/tools-Migration_Toolkit_for_Application_Binaries)
- File downloads:
  - [Eclipse for Windows](http://ibm.biz/BdiTNM)  
  - [Eclipse for Mac](ibm.biz/BdiTNv)  
  - [Eclipse Workspace - Cross-platform](http://ibm.biz/BdiTNy)

### Step 1: Getting the project repository

You can clone the repository from its main GitHub repository page and checkout the appropriate branch for this version of the application.

1. `git clone https://github.com/ibm-cloud-architecture/refarch-jee-customerorder.git`  
2. `cd refarch-jee-customerorder`  
3. `git checkout was90-prod`  

## Application Modernization Assessment

### Step 2: Perform assessment walkthrough

#### Step 2.1 Assess CustomerOrderServicesApp's readiness for migration

In this section you will use the Migration Toolkit for Application Binaries to generate evaluation reports for the EAR file **[CustomerOrderServicesApp-0.1.0-WAS7.ear](https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/artifacts/WASaaS-tWAS-tutorial/WAS7/CustomerOrderServicesApp-0.1.0-WAS7.ear)**. It is the original app that runs on WAS V7.

##### 2.1.1: Use the Migration Toolkit for Application Binaries to evaluate the applications

In this section you will generate and review the Application Evaluation Report for [**CustomerOrderServicesApp-0.1.0-WAS7.ear**](https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/artifacts/WASaaS-tWAS-tutorial/WAS7/CustomerOrderServicesApp-0.1.0-WAS7.ear)

1. In a browser, download the [WebSphere Application Server Migration Toolkit for Application Binaries](https://developer.ibm.com/wasdev/downloads/#asset/tools-Migration_Toolkit_for_Application_Binaries) by clicking the **Download** button on the opened page.

2. In a terminal, navigate to the location of the download (generally **Downloads**) and then install the toolkit:

`cd ~/Downloads`  
`java -jar binaryAppScannerInstaller.jar`  

After accepting the license agreement, and specifying the target directory for the installation, the tool is installed in the `<target directory>/wamt` directory.

3. Navigate to the location of **{binaryAppScanner.jar}** by issuing the command:

`cd wamt`

4. Run the Migration Toolkit for Application Binaries passing the binaryInputPath to CustomerOrderServicesApp and the `–evaluate` action as shown below:

`java -jar binaryAppScanner.jar ~/Downloads/CustomerOrderServicesApp-0.1.0-WAS7.ear --evaluate`

The Application Evaluation Report will be displayed in the browser.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/MigToolKit/AppEvalReport.png">
</p>

This report gives a quick evaluation of the Java EE technologies that an application uses and whether each of the WebSphere runtimes support the given technologies. **This report can be used to determine whether a WebSphere Application Server runtime supports the technologies required by the application.** For example, this CustomerOrderServicesApp **is a perfect match for any of the WAS runtimes.**

5. **Review** the report and then **close** the tab.

*Customer Decision: At this point the customer has decided to move the application to tWAS V9.  It is important to remember that CustomerOrderServicesApp would have been completely supported not only by tWAS V9, but also Liberty for Java, Liberty Core, Liberty, and so forth.*

##### 2.1.2: Use the Migration Toolkit for Application Binaries to inventory CustomerOrderServicesApp

In this section, you will generate and review the Application Inventory Report for CustomerOrderServicesApp which will document the application structure as well as list any possible deployment problems that may be encountered.

1. Run the Migration Toolkit for Application Binaries passing the binaryInputPath to CustomerOrderServicesApp and the `--inventory` action as shown below.

`java -jar binaryAppScanner.jar ~/Downloads/CustomerOrderServicesApp-0.1.0-WAS7.ear --inventory`

2. The Application Inventory Report will be displayed in the web browser.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/MigToolKit/Inventory.png">
</p>

3. **Scroll** down through the report and **review** the information provided. The **Inventory** section of the report provides information about the type and quantity of the different components contained in the application. The **Potential Deployment Problems** section details potential issues that might be encountered if the application was deployed to WAS V9.

- **Archives not referenced in the application.** You can ignore this potential problem.

- **Archives missing dependencies in the application.** You will need to find missing dependencies. Most of the time you can find the missing dependencies in a shared library. Click the **Show details** button to see the missing dependencies.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/MigToolKit/DeploymentProb.png">
</p>

4. The Missing Dependencies report describes the problem and recommendation for how to deal with this problem, then provides a table with additional information to help you solve the problem. The table shows Missing Dependencies (classes) and the Archive from the CustomerOrderServicesApp that is dependent upon them.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/MigToolKit/MissDependencies.png">
</p>

5. **Close** the tab.

#### Step 2.2: Use Eclipse IDE Tools to analyze the application code

In this section you will use the WebSphere Application Migration Toolkit (WAMT) to analyze the CustomerOrderServicesApp application code for readiness to run on traditional WebSphere Application ServerV9 (tWAS V9). An Eclipse Workspace has been provided with the CustomerOrderServicesApp Application Source Code already imported.

First, you will need an Eclipse IDE client application on your workstation.

If you have already installed Eclipse Neon.3, ensure that you have the following plugins installed:

[WebSphere Developer Tools for Eclipse Neon](https://developer.ibm.com/wasdev/downloads/#asset/tools-WebSphere_Developer_Tools_for_Eclipse_Neon)

Note: Select WebSphere Application Server Migration Toolkit as part of the download.

If you have already installed Eclipse Oxygen, ensure that you have the Websphere Developer Tools for Eclipse Oxygen by going to the Help menu, selecting Eclipse Marketplace, searching for **Websphere Developer Tools**, and clicking the Install button.

Note: Select WebSphere Application Server Migration Toolkit as part of the download.

If you do NOT have an Eclipse IDE installed on your workstation, or have a version prior to Neon.3, you can download the Eclipse Neon.3 image containing all the necessary tools already installed at the following links. (You may have previously downloaded these in the prerequisites section.)

[Eclipse Neon.3 for Windows w/ WAMT](http://ibm.biz/BdiTNM)

[Eclipse Neon.3 for Mac w/ WAMT](http://ibm.biz/BdiTNv)

Alternatively, you can click here to install Eclipse Neon 3 [Eclipse IDE for Java EE Developers Neon.3](http://www.eclipse.org/downloads/packages/eclipse-ide-java-ee-developers/neon3). You will then have to install the Developer Tools and Migration Toolkit plugins as described above.

Secondly, all users should download the required [Eclipse Workspace - Cross-platform](http://ibm.biz/BdiTNy) containing the CustomerOrderServicesApp Application Source Code.

##### 2.2.1: Analyze the CustomerOrderServicesApp application code

1. Open Eclipse with a clean workspace, from your application menu or using the command line:

cd {ECLIPSE_NEON_HOME}
eclipse {-clean}

2. In Eclipse, go to the File menu and choose Import....

3. Expand General option and select Existing Projects into Workspace.

4. Choose the option Select Archive File and then browse the workspace refarch-jee-customerorder-was70-dev (Export-Archive-File).zip.

5. Ensure all four of the CustomerOrderServices projects are selected and click Finish to complete the import.

- If you want to use your own Eclipse environment, make sure you install all the necessary packages referenced at the beginning of [Step 2.2](#step-22-analyze-the-customerorderservicesapp-application-code).

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/EclipseWorkspace.png">
</p>

3. The Java EE Perspective will be displayed with the projects that make up the CustomerOrderServicesApp-0.1.0-SNAPSHOT.ear application already imported. Take a moment to familiarize yourself with the projects.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/EnterpriseExplorer.png">
</p>

- **CustomerOrderServices-0.1.0-SNAPSHOT** is the EJB Module for the application
- **CustomerOrderServicesApp-0.1.0-SNAPSHOT** is the EAR for the application
- **CustomerOrderServicesTest-SNAPSHOT** is the Web Module for the application
- **CustomerOrderServicesWeb-SNAPSHOT** is the Web Module for integration tests

##### 2.2.2. Execute the WAMT rules for traditional WAS V9 on-prem

In this section,you will use WAMT to analyze the CustomerOrderServicesApp application source code for readiness for migration to run on WebSphere Application Server V9 in a non-Cloud environment.

1. In Eclipse, click **Run -> Analysis... (move your mouse pointer to the menu bar at the top of the Eclipse Neon window to make the menu names appear as they are hidden by default)**

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/Analysis.png">
</p>

2. When the Software Analyzer Configurations dialog is displayed, **right-click** on **Software Analyzer** and select **New** to create a new configuration.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/SoftwareAnalyzer.png">
</p>

3. Enter **“WAS V9 on-prem”** in the **Name** box and click on the **Rules** tab.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/Rules.png">
</p>

4. Validate that the correct number of rules are shown in the **Java Code Review** section (the value should be **598**). If the correct number of rules are not shown, exit and start eclipse with the –clean parameter (eclipse –clean) and repeat steps 1-3 in this section)

<p align="center">
  <!-- <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/RulesV9.png"> -->
</p>

5. Select **WebSphere Application Server Version Migration** from the **Rule Sets** drop down then click **Set...**

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/WASVersionMig.png">
</p>

6. When the Rule set configuration panel is displayed select the following values and click **OK**

Source application server: **WebSphere Application Server V7.0**

Target application server: **WebSphere Application Server traditional V9.0**

Target cloud runtime: **None**

**NOTE:** It is important to note the six options that exist in the Java EE 7 technologies section. Traditional WAS V9 (tWAS V9) is a Java EE 7 runtime which by default runs newer levels of the CDI, EL, JAX-RS, JMS, JPA and Servlet specifications. These options allow you to state whether you intend to upgrade your application code to the latest specification during migration or not. In this case, you do **not** plan to upgrade the application from JPA 2.0 to JPA 2.1 (JPA 2.1 is now the default implementation for tWAS V9) or from JAX-RS 1.1 to JAX-RS 2.0(JAX-RS 2 is now the default for tWAS V9) so leave those boxes unchecked.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/RuleSet.png">
</p>

7. Note that rules have been selected for you based on the configuration parameters you provided in the previous step. Take a moment to expand the selected sections and to review the selected rules.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/SelectedRules.png">
</p>

8. Click **Apply** and then click **Analyze**.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/Apply%26Analyze.png">
</p>

9. When the analysis is complete, the **Software Analyzer Results** panel will be displayed with four tabs shown below: XML Code Review, XML File Review, JSP Code Review, and File Review.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/SAResults.png">
</p>

10. In the Eclipse menu bar, click **Help -> Show Contextual Help** to display the Eclipse Help on the right side of the workspace.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/Help.png">
</p>

11. In the **Java Code Review** section, expand the **Java EE 6** result tree and click on the **Java API for RESTful Web Services (JAX-RS)** result to display the related help in the Eclipse Help view.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/JavaCodeReview.png">
</p>

12. WAMT has detected that JAX-RS is part of this application and given that you stated in step #6 that you do not intend to upgrade to JAX-RS 2.0, WAMT is drawing your attention to the fact that JAX-RS 1.1 will run on tWAS V9 but only when explicitly configured. Click **Detailed help**.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/DetailedHelp.png">
</p>

13. Review the information provided in the detailed help which discusses why this rule was executed and provides a link to the Information Center describing how the server can be configured to use JAX-RS 1.1. **Note that had you specified that you wanted to upgrade to JAX-RS 2.0 when configuring the WAMT rules, there are quick fixes and additional rules to help with the code migration.**

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/JAXRS.png">
</p>

14. In the **Java Code Review** section, double-click on CustomerOrderRESTTest.java

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/RestTest.png">
</p>

15. CustomerOrderRESTTest.java is displayed with the import for **javax.ws.rs.core.MediaType** highlighted. Note that WAMT only reports the first instance of a result for this rule per Java file.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/RestTestCode.png">
</p>

16. Close **CustomerOrderRESTTest.java**

17. In the **Java Code Review** section, click on the **Java Persistence API (JPA)** result to display the related help in the Eclipse Help view.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/JPA_API.png">
</p>

18. WAMT has detected that JPA is part of this application and given that you stated in Step #6 that you do not intend to upgrade to JPA 2.1, WAMT is drawing your attention to the fact that JPA 2.0 will run on tWAS V9 but only when explicitly configured. Click **Detailed help**.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/tWASDetailedHelp.png">
</p>

19. Review the information provided in the detailed help which discusses why this rule was executed and provides a link to the Information Center describing how to identify and modify the JPA level on a tWAS V9 server. **Note that had you specified that you wanted to upgrade to JPA 2.1 when configuring the WAMT rules, there are quick fixes and additional rules to help with the code migration.**

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/JPARules.png">
</p>

20. In the **Java Code Review** section, double-click on the AbstractCustomer.java result.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/AbstractCust.png">
</p>

21. AbstracCustomer.java is displayed with the import for **javax.persisence.CascadeType** highlighted. Note that WAMT only reports the first instance of a result for this rule per Java source.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/javax.png">
</p>

22. Close **AbstractCustomer.java**.

23. In the **Java Code Review** section, expand the **Java EE 7 -> CDI** section and click on **CDI recognizes implicit bean archives** to display the related help in the Eclipse Help view.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/CDI.png">
</p>

24. WAMT detected two Java files with bean-defining annotations without corresponding beans.xml file. This means that their WAR or JAR modules would not be scanned for implicit beans in WAS 7.0, but they would be scanned in WAS 9.  Click **Detailed help**.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/CDIHelp.png">
</p>

25. Review the information provided in the detailed help which discusses why this rule was executed and provides a solution to disable this CDI 1.2 behavior for the web modules and EJB modules.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/CDI_Bean_Archives.png">
</p>

26. In the **Java Code Review** section,double-click on **CustomerOrderServicesImpl.java**

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/CDIImplicitBean.png">
</p>

27. CustomerOrderServiceImpl.java is displayed which defines the EJB Stateless Session Bean with the @Stateless annotation highlighted.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/Stateless.png">
</p>

28. Close **CustomerOrderServicesImpl.java**.

29. In the **Java Code Review** section, open the **Java EE 7 -> WebSphere version migration** section and click on **Check for a behavior change in JPA cascade strategy** to display the related help.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/BehaviorChange.png">
</p>

30. This rule flags the use of the JPA entity relationships that use cascade types PERSIST, MERGE, or ALL as their behavior has changed in WAS V8.5 and later. Click **Detailed help**.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/BehaviorChange_HElp.png">
</p>

31. The detailed help describes how the JPA cascade strategy has changed in WAS V8.5 and later. This behavior does not impact most applications, but if so the detailed help shows you how to go back to the earlier behavior.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/JPACascadeStrategy.png">
</p>

32. In the **Java Code Review** section, double-click on **AbstractCustomer.java**.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/AbsCust_BehaviorChange.png">
</p>

33. The **AbstractCustomer.java** file is displayed with the line that triggered the rule highlighted. That line defines JPA OneToOne relationship with CascadeType of MERGE.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/CascadeType.png">
</p>

34. Close **AbstractCustomer.java**.

35. In the **File Review** section, expand the **Project Review** section and click on the **CDI scans for implicit beans when there is no beans.xml** file result to display the related help in the Eclipse Help view.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/BeansXML.png">
</p>

36. WAMT has again detected that there are projects that don’t contain a beans.xml file which means that the project will be scanned for implicit beans on application startup which can degrade performance.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/ApplicationAnalysis/CDI_BeansXML.png">
</p>

37. The table below summarizes the results and the recommended actions.

| **WAMT Result** | **Recommended Action** |
| ------- | ------- |
| Java API for RESTful Web Services (JAX-RS) | WebSphere Application Server traditional version 9 supports both Java™ API for RESTful Web Services (JAX-RS) 2.0 and JAX-RS 1.1 features. In version 9 and its later versions, you can switch among different JAX-RS providers.The default JAX-RS provider on WebSphere Application Server traditional version 9 is Apache CXF-based JAX-RS 2.0. You can also switch to Apache Wink-based JAX-RS 1.1 either by using command prompt or administrative console. A migration approach to migration that focuses on avoiding code changes would continue to use the JAX-RS 1.1 provider while a migration approach to eliminate technical debt would migrate to JAX-RS 2.0. |
| Java Persistence API (JPA) | Starting with WebSphere Application Server Version 9, Eclipselink is the default JPA 2.1 persistence provider when JPA 2.1 is the active specification level. Before version 9, WSJPA, the IBM JPA provider based on Apache OpenJPA, which supports the JPA 2.0 specification, was the default provider. Because JPA providers differ in both behavior and vendor-specific APIs, and OpenJPA is not available as a JPA 2.1 specification implementation, the product provides a JPA 2.0 specification compatibility mode. WAS V9 bundles the following persistence providers: Eclipselink 2.6.x_WAS (default for JPA 2.1), WSJPA 2.2.x (default for JPA 2.0). You can also use third-party JPA persistence providers such as Hibernate. The third-party provider must match the enabled JPA specification level. A migration approach that focuses on avoiding code changes would enable the JPA 2.0 specification and use a JPA 2.0 provider while a migration approach to eliminate technical debt would migrate to JPA 2.1.  |
| CDI recognizes implicit bean archives | The change in CDI 1.1 means that WAR and JAR files are scanned for implicit beans even if they don’t have a beans.xml file. The concern here is two-fold: first, the application startup time is increased as every WAR and JAR without a beans.xml file is scanned; second, the beans that weren’t detected in CDI 1.0 will now be detected and started which might change the application functionality. The recommendation here is to add a beans.xml file to all WAR and JAR files that either states that no beans are present or correctly identifies the beans to ensure that scanning is kept to a minimum and only the expected beans are started. |
| Check for a behavior change in JPA cascade strategy | Code should be reviewed based on the information provided in the WAMT detailed help to ensure that that the new exception is handled correctly. |
| Target runtime not set | The recommendation is to specify the correct target runtime which requires installation of the WebSphere Developer Tools |

**Summary: Although a few sections of code should be reviewed by a developer, the application will run on traditional WebSphere Application Server V9 on-prem without changes. Strategically it would be wise to consider upgrading the JAX-RS to 2.0 and upgrading JPA to 2.1.**

## WebSphere On The Cloud

You will now switch over to deploying a migrated version of the EAR to a managed WebSphere instance on Bluemix.

### Step 3: Create DB2 service instance for ORDERDB

NOTE:  Your Bluemix account may not be able to provision the desired `Db2 on Cloud SQL DB (formerly dashDB TX)` service instances as requested below.  If that is the case, your lab proctor has been provided with shared credentials for you to use for this lab.

1. Go to your [Bluemix console](https://new-console.ng.bluemix.net/) and create an instance of [`Db2 on Cloud SQL DB (formerly dashDB TX)`](https://console.bluemix.net/catalog/services/db2-on-cloud-sql-db-formerly-dashdb-tx) and name it `DB2 on Cloud - ORDERDB`
2. When you are redirected back to your Services dashboard, click on the new database service instance.
2. Click on `Service Credentials` and then click on `New credential`.
3. Click `Add` and then click on `View credentials`.
4. Make note of the `password` field for your instance.
5. Click on `Manage` and then click on `Open`.
6. Click on `Run SQL`
7. Click on `Open Script` and browse to `createOrderDB.sql` inside the 'Common' sub-directory of the project directory.
8. Click `Run All`
9. You should see some successes and some failures.  This is due to the scripts cleaning up previous data, but none exists yet.  You should see 28 successful SQL statements and 30 failures.
10. Go to the dropdown in the upper right and click on `Connection Info`
11. Select `Without SSL` and copy the following information for later:
- Host name _(most likely in the form of dashdb-txn-flex-yp-dalXX-YY.services.dal.bluemix.net)_
- Port number _(most likely 50000)_
- Database name _(most likely BLUDB)_
- User ID _(most likely bluadmin)_
- Password _(previous password from the `View credentials` tab)_

### Step 4: Create DB2 service instance for INVENTORYDB

NOTE:  Your Bluemix account may not be able to provision the desired `Db2 on Cloud SQL DB (formerly dashDB TX)` service instances as requested below.  If that is the case, your lab proctor has been provided with shared credentials for you to use for this lab.

1. Go to your [Bluemix console](https://new-console.ng.bluemix.net/) and create an instance of [`Db2 on Cloud SQL DB (formerly dashDB TX)`](https://console.bluemix.net/catalog/services/db2-on-cloud-sql-db-formerly-dashdb-tx) and name it `DB2 on Cloud - INVENTORYDB`
2. When you are redirected back to your Services dashboard, click on the new database service instance.
3. Click on `Service Credentials` and then click on `New credential`.
4. Click `Add` and then click on `View credentials`.
5. Make note of the `password` field for your instance.
6. Click on `Manage` and then click on `Open`.
7. Click on `Run SQL`
8. Click on `Open Script` and browse to `InventoryDdl.sql` inside the 'Common' sub-directory of the project directory.
9. Click `Run All`
10. You should see some successes and some failures.  This is due to the scripts cleaning up previous data, but none exists yet.  You should see 5 successful SQL statements and 4 failures.
11. Click on `Open Script` and browse to `InventoryData.sql` inside the 'Common' sub-directory of the project directory.  Confirm the prompt that you would like to open this new file and replace the previous content of the SQL Editor.
12. Click `Run All`
13.  You should now see 12 successes.
14. Go to the dropdown in the upper right and click on `Connection Info`
15. Select `Without SSL` and copy the following information for later:
- Host name _(most likely in the form of dashdb-txn-flex-yp-dalXX-YY.services.dal.bluemix.net)_
- Port number _(most likely 50000)_
- Database name _(most likely BLUDB)_
- User ID _(most likely bluadmin)_
- Password _(previous password from the `View credentials` tab)_

### Step 5: Create WebSphere Application Server service instance

1. Go to your [Bluemix console](https://new-console.ng.bluemix.net/) and create a [**WebSphere Application Server** instance](https://console-regional.ng.bluemix.net/catalog/services/websphere-application-server)

2. Name your service, choose the **WAS Base Plan**, and create the instance.

3. Once the service instance is created, provision a **WebSphere Version 9.0.0.0** server of size **Medium**.  This should be a server deployment taking up 2 of your 2 trial credits.  This step can take up to 30 minutes to complete.

4. Now you will access the WASaaS Admin Console to reconfigure your WAS cloud server. But first, you need to install and configure a VPN connection (if you have not already done so as a prerequisite):

A. Install the VPN Client

For Windows users, use the links provided on your WASaaS dashboard to download the installation executables for the latest versions of OpenVPN. Once downloaded, these files can be opened in that location. Follow the installation instructions. Install the VPN client using the following directory:

C:\Program Files\OpenVPN

For Linux users, use the link provided on your WASaaS dashboard and download the application binary for your version of Linux to install OpenVPN.

For Mac users, use the link provided on your WASaaS dashboard and follow the instructions to download and install OpenVPN.

Note: Please use the links provided in your Bluemix WASaaS dashboard.

B. Download and Extract the VPN Config Files

For Windows users, download the VPN configuration archive file using the **Download VPN Configuration** button on your WASaaS dashboard. From the download location, extract the files to the following required location:

`C:\Program Files\OpenVPN\Config`

For Linux and Mac users, download the VPN configuration archive file using the **Download VPN Configuration** button on your WASaaS dashboard. From the download location, extract the files to the following location:

`{OpenVPN home}\config`

C. Establish Your VPN Connection

For Windows users:

1. Right-click on the OpenVPN GUI icon on your desktop, select **Run as administrator**
2. Right-click on the OpenVPN GUI icon in your Windows system tray, select **Connect**

For Linux Users:

1. Go to **Network Manager** tray.
2. From the pop up menu, click on **VPN connections** and select the **Open VPN** connection.

For Mac users:

1. Click on the OpenVPN GUI icon in the Menu bar.
2. Click on the **connect wasaas-uss** to connect to OpenVPN.

Your VPN connection should now be established.

D. You should now be able to access the WASaaS Admin Console by selecting "Open the Admin Console" on your WASaaS dashboard.

5. Get a public IP address. This can be done using the [**Manage Public IP Access**](https://console.bluemix.net/docs/services/ApplicationServeronCloud/networkEnvironment.html#networkEnvironment) option.

6. You can **ssh** into the WebSphere Application Server instance using the Admin Username **root** and Password provided in your bluemix instance.

   In **Application Hosts/Nodes**, by expanding the Traditional **WebSphere Base option**, you can find the details of your OS distribution, Admin username, Admin password and Key Store password.

### Step 6: Perform WebSphere configuration

1. You will need to add a new firewall rule to the WebSphere instance to communicate with both the DB2 service and the remote LDAP server.  While **ssh**'ed into the WebSphere instance, run the following commands to allow traffic between WAS, DB2, and LDAP via our Secure Gateway connection.

`cd /opt/IBM/WebSphere/AppServer/virtual/bin`  
`sudo ./openFirewallPorts.sh -ports 50000:tcp,17830:tcp -persist true`  

2. Log into the Admin Console via the address accessible from your service instance page.

3. In the Global security section, check **Enable application security** and click **Save**.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/imgs/Customer_README/Readme1.png">
</p>

##### LDAP Configuration

In order to manually set WebSphere up to use a standalone LDAP registry for Authentication and Authorization of application users, follow these instructions:

1. Open WebSphere Admin Console and go to **Security --> Global Security**.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/LDAP_Images/LDAP_home.png">
</p>

2. In the *User account repository section* at the bottom of the page, select **Standalone LDAP registry** from the *Available realm definition* dropdown menu.

3. Click on Configure...

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/LDAP_Images/LDAP_configure.png">
</p>

4. Set the Primary administrative user name to **uid=wasadmin,ou=caseinc,o=sample**.

5. Select **IBM Tivoli Directory Server** from the Type of LDAP server dropdown menu.

6. Set the Host and Port to **cap-sg-prd-4.integration.ibmcloud.com** and **17830** respectively.

7. Set the LDAP admin credentials using the Bind distinguished name and password below:
   * Bind distinguished name (DN)     : **uid=casebind,ou=caseinc,o=sample**
   * Bind password                    : **caseBindUser!**

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/LDAP_Images/LDAP_TestConn.png">
</p>

8. Click **OK** to confirm the settings.

9. Click on the **Test connection** button at the top to make sure your standalone LDAP server is reachable by your WebSphere Application Server.

10. Go to the Additional Properties at the bottom of the page and click on **Advanced Lightweight Directory Access Protocol (LDAP) user registry settings**

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/LDAP_Images/LDAP_Adv.png">
</p>

11. Set the User filter and group filter.

    **User filter: (&(uid=%v)(objectclass=inetorgperson))**

    **Group filter: (&(cn=%v)(objectclass=groupOfUniqueNames))**

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/LDAP_Images/LDAP_Adv_Settings.png">
</p>

12. Apply the changes and save the settings.

13. Click **OK** to go back to the Global Security page.

14. Set **Standalone LDAP registry** as the current realm definition by clicking **Set as Current**.

15. You will now need to restart the server.  For this, you will need to return to your **ssh** session on the WebSphere Application Server instance.  Perform the following commands to restart the server:

`cd /opt/IBM/WebSphere/Profiles/DefaultAppSrv01/bin`

`./stopServer.sh server1 -username wsadmin -password <YOUR PASSWORD>` where _<YOUR PASSWORD>_ is found on the service instance details page in Bluemix.  

`./startServer.sh server1`  

16. Now you will log back into the Admin Console but use a different set of credentials, now that we're connected to the remote LDAP as our user registry.  Use the credentials below to login to the Admin Console:

**Username:** `uid=wasadmin,ou=caseinc,o=sample`  
**Password:** `websphereUser!` 

17.  You will now need to add an SSL Certificate into the WebSphere SSL Key Store for cross-server communication
   -  Log back into the administrative console:
   -  Expand **Global Security** and click **SSL certificate and key management**. Under _Configuration settings_, click **Manage endpoint security configurations**. 
   -  Select the appropriate outbound configuration to get to the **(cell):CloudBurstCell_123XYZ:(node):CloudBurstNode_123XYZ** management scope. 
   -  Under _Related Items_, click **Key stores and certificates** and click the **NodeDefaultTrustStore** key store. 
   -  Under _Additional Properties_, click **Signer certificates** and  **Retrieve From Port**.  
   -  In the _Host_ field, enter **hooks.slack.com** in the host name field, enter **443** in the _Port_ field, and **hooks.slack.com_cert** in the _Alias_ field. 
   -  Click **Retrieve Signer Information**.  
   -  Verify that the certificate information is for a certificate that you can trust. 
   -  Click **Apply** and **Save**.

##### Configuring JDBC Resources

Under **Global Security**, expand **Java Authentication and Autorization Service** and choose **J2C authentication data**.

Create a new user named **DBUser-ORDERDB** using your DB2 on Cloud instance and password.  The user will be `bluadmin` and the password will be specific to the instance you named **DB2 on Cloud - ORDERDB**.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/imgs/Customer_README/Readme4.png">
</p>

On the same page, create another new user named **DBUsuer-INVENTORYDB** using your DB2 on Cloud instance and password.  The user will again be `bluadmin`, but the password will be different as you are connecting to a different DB2 database on a different Bluemix service instance.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/imgs/Customer_README/Readme4_1.png">
</p>

1. Go to the **Resources > JDBC > JDBC Providers** section and ensure that you are at the **Cell** scope.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/imgs/Customer_README/Readme5.png">
</p>

2. Click the New Button to create a new JDBC provider.
    -  Database type : **DB2**
    -  Provider type : **DB2 Using IBM JCC Driver**
    -  Implementation type : **XA data source**

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/imgs/Customer_README/Readme6.png">
</p>

3. You need to enter the database classpath information for the DB2 driver JARs.  This value should be `/opt/IBM/WebSphere/AppServer/deploytool/itp/plugins/com.ibm.datatools.db2_2.2.200.v20150728_2354/driver` for your new WebSphere Application Server instance.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/imgs/Customer_README/Readme7.png">
</p>

4. Press **Next** and then **Finish**. Save the Configuration.

<p align="center">
  <img src="https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/imgs/Customer_README/Readme8.png">
</p>

5. Go to the **Resources > JDBC > Data sources** section to create a new data source.
   1. Make sure that the scope is at **Cell** level and click **New**
   2. OrderDB - Step 1
      -  Data source name: **OrderDS**
      -  JNDI name: **jdbc/orderds**
      <p align="center">
      <img src="https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/imgs/Customer_README/Readme9.png">
      </p>    
   3. OrderDB - Step 2
      - Select an existing JDBC provider --> **DB2 Using IBM JCC Driver (XA)**
      <p align="center">
      <img src="https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/imgs/Customer_README/Readme10.png">
      </p>
   4. ORDERDB - Step 3
      - Driver Type: **4**
      - Database name: **BLUDB**
      - Server name: **The DB2 host from your "DB2 on Cloud - ORDERDB" service instance**
      - Port number: **The DB2 port from your "DB2 on Cloud - ORDERDB" service instance***
      <p align="center">
      <img src="https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/imgs/Customer_README/Readme11.png">
      </p>
   5. OrderDB - Step 4
      - Authentication alias for XA recovery: **DB2User-ORDERDB**
      - Component-managed authentication alias: **DB2User-ORDERDB**
      - Mapping-configuration alias: **DefaultPrincipalMapping**
      - Container-managed authentication alias: **DB2User-ORDERDB**
       <p align="center">
       <img src="https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/imgs/Customer_README/Readme12.png">
       </p>

6. Once this is done, under Preferences, there will be a new resource called **OrderDS**. Make sure that the resources got connected using **Test Connection** option. You will see a success message if the connection is established successfully.

<p align="center">
<img src="https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/imgs/Customer_README/Readme13.png">
</p>

7. Check the Data source and select Test Connection to ensure you created the database correctly.  If the connection fails, a few things to check are
      - Your database is started as we did in the beginning.  
      - Your host and port number are correct.
      - The classpath for the Driver is set properly.  
      - Check the WebSphere Variables.  You may want to change them to point to your local DB2 install.

8. Create the INVENTORYDB data source using the same process as before.  Click **New**.
   1. InventoryDB - Step 1
      -  Data source name: **INDS**
      -  JNDI name: **jdbc/inds**
   2. InventoryDB - Step 2
      - Select an existing JDBC provider --> **DB2 Using IBM JCC Driver (XA)**
   3. InventoryDB - Step 3
      - Driver Type: **4**
      - Database name: **BLUDB**
      - Server name: **The DB2 host from your "DB2 on Cloud - INVENTORYDB" service instance**
      - Port number: **The DB2 port from your "DB2 on Cloud - INVENTORYDB" service instance***
   4. InventoryDB - Step 4
      - Authentication alias for XA recovery: **DB2User-INVENTORYDB**
      - Component-managed authentication alias: **DB2User-INVENTORYDB**
      - Mapping-configuration alias: **DefaultPrincipalMapping**
      - Container-managed authentication alias: **DB2User-INVENTORYDB**

9. Remember to save and test the connection again.

##### Configuring JPA Specification

1. In the admin console, go to **Servers** and expand **Server Types**.

2. Select **WebSphere application servers**.

<p align="center">
<img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/JPA_Specification/JPA1.png">
</p>

3. Click on **server1** from the Application Servers list.

<p align="center">
<img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/JPA_Specification/JPA2.png">
</p>

4. In the **Configuration** tab, expand **Container Services**

<p align="center">
<img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/JPA_Specification/JPA3.png">
</p>

5. Choose **Default Java Persistence API settings**

<p align="center">
<img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/JPA_Specification/JPA4.png">
</p>

6. Go to **JPA Specification** option and choose **2.0**

<p align="center">
<img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/JPA_Specification/JPA5.png">
</p>

<p align="center">
<img src="https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/JPA_Specification/JPA6.png">
</p>

7. Click **Apply** and **Save** the configuration.


**Note**: Whenever you make any changes to the WebSphere Configuration Settings, it prompts you with a warning message. Please review and save the modifications.

### Step 7: Install Customer Order Services application

1.  We have provided a built EAR that has had the previously discussed changes for installation on WAS V9.0.  It is available at [https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/artifacts/WASaaS-tWAS-tutorial/WAS9/CustomerOrderServicesApp-0.2.0-WAS9.ear](https://github.com/ibm-cloud-architecture/refarch-jee/raw/master/static/artifacts/WASaaS-tWAS-tutorial/WAS9/CustomerOrderServicesApp-0.2.0-WAS9.ear) for download.

2.  Install the EAR to the Admin console.
   -  Login to the Administrative Console.
   -  Select **Applications > Application Types > WebSphere enterprise applications**
   -  Choose **Install > Browse the EAR > Next > Choose Detailed**
   -  Click on **Step 12**.
   -  Customize the environment variables for your **DB2 on Cloud - ORDERDB** instance:
      - **DEPLOYER_EMAIL:** Enter your IBM email address in the form of _name@xx.ibm.com_.  **NOTE:**  This is important as this is how you will get credit for completing the lab.
      - **DBUNIT_CONNECTION_URL:** `jdbc:db2://{YOUR HOST}:50000/BLUDB` where {YOUR HOST} is the `host` value from your credentials
      - **DBUNIT_SCHEMA:** `BLUADMIN` - **NOTE:** This must be upper-case
      - **DBUNIT_USERNAME:** `bluadmin`
      - **DBUNIT_PASSWORD:** _(Value acquired in Step 3.4)_
   -  Click on **Step 13** to map the necessary users to the application.
      - Click on the checkbox for the **SecureShopper** Role.
      - Then click on **Map Groups**.  
      - Click **Search** and select the `SecureShopper` group.
      - Click the right-facing arrow to move those selected users into the **Selected** box on the right.
      - Click **OK**
   -  Click on **Summary** (Step 18) and click **Finish**.
   -  Once you see Application **CustomerOrderServicesApp** installed successfully, click **Save** and now your application is ready.

3.  Go back to the Enterprise Applications list, select the application, and click **Start**.

4.  Prime the database with the **JPA** tests available at **https://<i>your-host</i>:9443/CustomerOrderServicesTest** , where **your_host** is the **Host** address of your WAS instance displayed in **Application Hosts/Nodes** section.

5.  Login as the user `rbarcia` with the password of `bl0wfish`.  

6.  Select the checkboxes for the **org.pwte.example.jpa.test.CustomerOrderServicesTest** and **org.pwte.example.jpa.test.ProductSearchServiceTest** tests.  Then click **Run**.  

7.  You should see a screen with 10 successful tests.  If you do not, you should verify the connectivity information you updated during application installation.  This is available in the Admin Console via the `Applications > WebSphere enterprise applications > CustomerOrderServicesApp > Environment entries for Web modules` page.

8.  Access the application at **https://<i>your-host</i>:9443/CustomerOrderServicesWeb/#shopPage** , where **your_host** is the **Host** address of your WAS instance displayed in **Application Hosts/Nodes** section.

9.  Login as the user `rbarcia` with the password of `bl0wfish`.  

10.  Add an item to the cart by clicking on an available item.  Drag and drop the item to the cart.

11.  You should see the contents of your cart updated in the top right of the screen.
