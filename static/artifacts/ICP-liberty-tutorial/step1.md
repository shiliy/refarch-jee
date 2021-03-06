# Step 1. Modernise application to run on WebSphere Liberty profile (Optional)

In this step, we are going to make the modifications needed both at the application level and the server configuration level to migrate our WebSphere Application Server 7 application to run in WebSphere Liberty.

1.  [Install WebSphere Application Server Liberty locally](#install-websphere-application-server-liberty-locally)
2.  [Get the code](#get-the-code)
3.  [Set up your development environment](#set-up-your-development-environment)
4.  [Source Code Migration](#source-code-migration)
    - [Software Analyzer Configuration](#software-analyzer-configuration)
    - [Run the Software Analyzer](#run-the-software-analyzer)
5. [Configure WebSphere Liberty Server](#configure-websphere-liberty-server)
6. [Run the application](#run-the-application)


## Install WebSphere Application Server Liberty locally

The IBM WebSphere Application Server Liberty can be installed from eclipse. Since we will also need to configure eclipse to use IBM WebSphere Liberty Server as the targeted runtime, we will then do both from eclipse at once.

1. Open eclipse

```
cd ~/eclipse
./eclipse
```

select the default workspace by clicking OK

![Source migration 48](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source48.png)

2. Close the welcome page

3. At the bottom, select the servers tab and click on the _"No servers available..."_ message.

![Source migration 49](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source49.png)

4. On the new server dialog that pops up, type in _liberty_ so that available servers get filtered. Then, select WebSphere Application Server Liberty and leave the rest as is. Click Next.

![Source migration 50](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source50.png)

5. Set the path for the existing installation to `/home/vagrant/wlp`. Click Next.

![Source migration 51](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source51.png)

6. On the next window, leave all the options as they are (see picture below) and click on Finish.

![Source migration 52](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source52.png)

7. You should now see your WebSphere Application Server Liberty at localhost created in the Servers tab at the bootom and a project that stores the configuration for your server on the left in the Enterprise Explorer tab.

![Source migration 53](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source53.png)

8. Finally, double click on the newly created WebSphere Application Server Liberty at localhost in the Servers view and set the start timeout value for the Liberty server to something around 180 or over (if the timeout is too small you might see an error saying that the Server WebSphere Application Server Liberty at localhost was unable to start within XX seconds)

![Source migration 61](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source61.png)

## Get the code

The migration toolkit is eclipse based. Therefore, these are the steps to be taken to get the code into eclipse to run the migration toolkit on it:

1. Open a terminal window

2. Download Customer Order Services application's source code from GitHub

```
cd ~/git
git clone https://github.com/ibm-cloud-architecture/refarch-jee-customerorder.git
```
![Source migration 35](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source35.png)

3. Switch to the WAS 7 development branch

```
cd refarch-jee-customerorder
git checkout was70-dev
```
![Source migration 36](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source36.png)

4. Go back to eclipse

5. Import projects

- Click on File --> Import...
- On the dialog that pops up, select Existing Projects into Workspace in the General folder and click Next.

![Source migration 37](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source37.png)

- In the next dialog, browse to ```/home/vagrant/git/refarch-jee-customerorder``` for the root directory and click Browse. 

![Source migration 38](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source38.png)

- Eclipse will directly detect all the projects within that fodler and select them all. Click Finish.

A migration dialog will pop up after importing the projects into the eclipse workspace since eclipse will detect such projects are configured to run on a runtime that it is not aware of. As a result, it pops up with a workspace migration dialog.

![Source migration 39](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source39.png)

Disregard this piece of advice by clicking on Cancel. In the migration cancel window, click OK. The runtime migration will be done in the following sections.

## Set up your development environment

When we use/create a new development environment, it is most likely that we will need to tidy it up a bit since installation paths, development tools versions, libraries and things like that might very well be different from the original development environment.

As you can see when you import the projects into eclipse, we get those projects with red error marks.

![Source migration 40](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source40.png)

You can open the problems view (Window --> Show View --> Other... --> Problems) in order to see what the problems in your workspace are.

![Source migration 54](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source54.png)

If you do so, you should be able to find errors for each of the projects regarding the build path among many other problems. That is, the references to the Java Runtime Environment and WebSphere libraries we have in our projects need to be updated for our new development environment. This should be the first step of all, since a project must have a correct build path to be built (and potentially uncover new issues).

In order to update your JRE and WebSphere libraries references, right click on a project and click on Build Path --> Configure Build Path... Once the properties dialog opens up, click on the Libraries tab. You should now see something similar to the following:

![Source migration 41](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source41.png)

We then need to fix the paths for the two **unbound** libaries which are the JRE System Library and the Server Library. As you can see, both are yet pointing to the WebSphere Application Server traditional V7.0 libraries from the old/original development environment. To update those libraries to point to the appropriate path in your environment, you need to select each of the libraries and click on the edit button on the right hand side of the properties dialog.

For the JRE System Library, select the last option which says Workspace default JRE and click Finish.

![Source migration 42](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source42.png)

For the Server Library, repeat the steps but select the only WebSphere Application Server Liberty option and click Finish.

![Source migration 43](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source43.png)

Click OK to close the properties window.

**IMPORTANT: Make sure you have fixed the build path for CustomerOrderServices, CustomerOrderServicesWeb and CustomerOrderServicesTest projects**

After updating the references to our actual WebSphere Server and JRE System libraries, we should clean and rebuild the entire workspace. For doing so, click on Project --> Clean... Verify Clean all projects is selected and click OK.

![Source migration 45](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source45.png)

If we now check the Problems view out, we might very well see many more problems. The reason for this is that projects are now getting built as a result of a correct build path:

![Source migration 55](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source55.png)

However, if we dobule click on a couple of those different errors we see in the Problems view, we realise that these errors happen as a result of missing third party libraries:

![Source migration 56](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source56.png)

![Source migration 57](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source57.png)

In our case, we are getting errors because we are missing the Java EE 6 rest services technology library needed for the compilation of our code. This technology comes as a library called JAX-RS and the Java EE 6 version of it is 1.1. Neither the JRE version we have installed on our system nor the WebSphere Application Server Liberty server come with the needed JAX-RS library out of the box.

However, WebSphere Application Server Liberty is a composable lightweight server which you need to add features to in order to get the Liberty server to support certain funcionality and technologies. Therefore, if we want to run an application that uses JAX-RS 1.1 technology we need to add/install the JAX-RS 1.1 feature/liibrary to our WebSphere Application Server Liberty server. This will also make the JAX-RS 1.1 library avaiable for compilation.

To install the JAX-RS 1.1 functionality to our WebSphere Application Server Liberty server execute:

```
~/wlp/bin/installUtility install jaxrs-1.1
```

![Source migration 58](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source58.png)

More about what technologies WebSphere Application Server Liberty supports and comes with out of the box as well as how to get these installed/added to your WebSphere Application Server Liberty on the [Configure WebSphere Liberty Server - features](#1-features) section below.

After installing any new feature/JEE feature to our WebSphere Application Server Liberty, we must clean and build the workspace. For doing so, click on Project --> Clean... Verify Clean all projects is selected and click OK.

If we look now to the Problems view, we should see many less problems:

![Source migration 44](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source44.png)

However, we still see problems. In this case, we want to sort out the Xpath is invalid error. To sort it out, click on CustomerOrderServicesWeb project, which is the one the errors are found on, and select Properties. On the properties dialog, select Validation on the left hand side, scroll down to the last validator which is XSL Validator and right click on it. Deselect both Manual and Build options.

![Source migration 46](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source46.png)

Click Apply and OK. Finally, clean and build the workspace. You should now see Target runtime errors and one JPA related error (library provider) which we will get fixed in the next section.

**Conclusion:** As we could see in the above walk through, when we move to a different/new development environment we will need to plan ahead and bear in mind that some time will be needed for setting up your new development environment. Even more if this new environment will be used for a WebSphere Application Server migration.

## Source Code Migration

In order to migrate the code to get our WebSphere Application Server 7 application working on WebSphere Liberty, we are going to use the [WebSphere Application Server Migration Toolkit (WAMT)](https://developer.ibm.com/wasdev/downloads/#asset/tools-WebSphere_Application_Server_Migration_Toolkit). The migration toolkit provides a rich set of tools that help you migrate applications from third-party application servers, between versions of WebSphere Application Server, to Liberty, and to cloud platforms such as Liberty for Java on IBM Bluemix, IBM WebSphere on Cloud and Docker.

### Software Analyzer Configuration

More precisely, we are going to use the Software Analyzer that the WAMT comes with. For doing so, we first need to have in our development environment eclipse opened with the WAMT installed on it and our WAS 7 application projects imported into the workspace. Once we have the above in place, click on **Run -> Analysis...** This will open the **Software Analyzer**.

![Source migration 1](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source1.png)

Now, **right-click** on _Sofwtare Analyzer_ and select **New**. Give a relevant and appropriate name to the new configuration and click on the **Rules** tab for this configuration. Select the **WebSphere Application Server Version Migration** option for the _Rule Sets_ dropdown menu and click **Set...**  If this step is already completed for you, just select the Rule set and examine the details.

![Source migration 2](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source2.png)

The Rule set configuration panel should be displayed. This panel must be configured so that the appropriate set of rules based on our migration requirements are applied during the software analysis of our applications. Click OK when done.

![Source migration 3](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source3.png)

### Run the Software Analyzer

Click Analyze. After running the _Software Analyzer_ you should see a _Software Analyzer Results_ tab at the bottom. The Software Analyzer rules are categorised, and so are the errors and warnings produced in its report, in four categories: **Java Code Review, XML File Review, JSP Code Review and File Review**. We must go through each of these tabs/categories and review the errors and warnings as code/configuration changes might be needed.

![Source migration 4](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source4.png)

We will start off with the **File Review** analysis tab. As you can see in the image below, one warning that will always appear when you migrate your apps to a newer WebSphere Application Server version is the need to configure the appropriate target runtime for your applications. This is the first and foremost step:

![Source migration 5](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source5.png)

Along with the warning and errors reported, the WebSphere Application Migration Toolkit also comes with information about the rule that flagged the each error/warning and the possible solutions for them. In order to see this info and help, click on **Help -> Show Contextual Help**. This will open the help portlet in eclipse.

![Source migration 6](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source6.png)

If you scroll down to the bottom and click Detailed help, it will show you additional ideas on how to resolve that problem.

![Source migration 7](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source7.png)

If we follow the instructions and go to Targeted Runtimes for any of the project we might find out that WebSphere Liberty does not seem to appear as an option.

![Source migration 8](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source8.png)

However, if we click on the Show all runtimes option, we see the other runtimes availabe in our environment. They now appear although they are greyed out. The reason is, as we can read at the bottom of the panel, that we might need to uninstall one or more of the currently installed project facets.

![Source migration 9](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source9.png)

In our case, the problem resides in the current WebSphere Application Server version 7 specific facets we have installed in our projects for them to properly run on that WebSphere version. As a result, we now need to uninstall them. For doing so, click the Uninstall Facets... hyperlink presented on this panel and then deselect all WebSphere specific facets you might find active.

![Source migration 10](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source10.png)

Once you have deselected all active WebSphere specific facets installed for the project, click on Finish. You will get back to the targeted runtimes panel but you will not see the WebSphere Application Server Liberty option available to select just yet until you click on Apply. Hence, click on Apply, deselect the existing WebSphere Application Server traditional V7.0 option, select the WebSphere Application Server Liberty option and click on Apply and OK. **Repeat the same process for all four projects**

![Source migration 11](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source11.png)

If we ran the Software Analyzer again, we should see the File Review tab empty.

Moving on to the **Java Code Review** category tab, these are the aspects the WebSphere Application Migration Toolkit warns us about:

![Source migration 12](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source12.png)

The first warning we see in there is about using the default initalContext JNDI properties. In order to understand more about the problem, we click on it and read what the Help portlet says about it. If we still want or need more information, we can always click on the Detailed help link displayed within the portlet for a deeper and longer explanation:

![Source migration 13](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source13.png)

Now that we have understood what the problem is, we can double click on the file pointed out by the Software Analyzer to inspect the actual code and determine whether this warning affects our application or not:

![Source migration 14](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source14.png)

As you can see by looking at the code, we are not using any of the two default initialContext JNDI properties this warning is about so we do not need to care about their default values. As a result, we can ignore this warning and move on to the next one.

Next aspect in this **Java Code Review** section is about the use of sytem-provided third-party APIs:

![Source migration 18](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source18.png)

If we click on the detailed help within the help portlet we are presented with the following information:

![Source migration 19](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source19.png)

However, that does not seem like enough information in order to figure out what the problem is. Therefore, we click on the link at the bottom which get us to the following IBM Knowledge Center page for WebSphere:

![Source migration 20](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source20.png)

With the information gathered along the links we have visited above, we now realise we need to configure the Liberty server to be able to allow the application access to third-party libraries. We do so by adding the following to the server.xml configuration file (this will be done in the next [Configure the Liberty Server](#configure-the-liberty-server) section):

```
<application id="customerOrderServicesApp" name="CustomerOrderServicesApp.ear" type="ear" location="${shared.app.dir}/CustomerOrderServicesApp.ear">
     <classloader apiTypeVisibility="spec, ibm-api, third-party" />
</application>
```

The above will allow the classloader to have access to the third-party libraries included with Liberty. Some of those third-party libraries we need the classloader to have access to so that our application work fine are Jackson and Apache Wink libraries.

Last aspect in this **Java Code Review** section has to do with a change in the JPA cascade strategy and its detailed information says the following:

![Source migration 15](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source15.png)

As we can read in the detailed info above, the change in the JPA cascade strategy is not expected to affect most applications. Also, this new cascade strategy can be mitigated by simply reverting to the previous behaviour by setting the _openjpa.Compatibility_ peroperty in the _persistence.xml_ file. Anyway, newer WebSphere Application Server versions can always be configured to run on previous or older version for most of the JEE technologies. JPA is one of them and you can see in the server.xml file that we are using tje jpa-2.0 feature so that the warning above does not affect our app at all.

Finally, moving on to the last **XML File Review** section in the Software Analyzer results, we see a problem due to a behaviour change on lookups for Enterprise JavaBeans:

![Source migration 16](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source16.png)

And this is what the detailed help says about it:

![Source migration 17](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source17.png)

If we click on the file that is being raised the error on, we realise we are using the WebSphere Application Server traditional namespaces for the EJB binding:

![Source migration 34](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source34.png)

Therefore, we need to change it to

```
java:app/CustomerOrderServices/ProductSearchServiceImpl!org.pwte.example.service.ProductSearchService
```

Save and close the file.

## Configure WebSphere Liberty Server 

The IBM WebSphere Application Server Liberty Profile is a composable, dynamic application server environment that supports development and testing of Java EE Full Platform web applications.

The Liberty profile is a simplified, lightweight development and application runtime environment that has the following characteristics:

* Simple to configure. Configuration is read from an XML file with text-editor-friendly syntax.
* Dynamic and flexible. The run time loads only what your application needs and recomposes the run time in response to configuration changes.
* Fast. The server starts in under 5 seconds with a basic web application.
* Extensible. The Liberty profile provides support for user and product extensions, which can use System Programming Interfaces (SPIs) to extend the run time.

[Here](https://www.ibm.com/support/knowledgecenter/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_feat.html) you can see the technologies that the WebSphere Application Server Liberty support on its different flavors. 

The application server configuration is described in a series of elements in the server.xml configuration file. We are now going to see what we need to describe in that server.xml configuration file to get our Liberty server prepared to successfully run our Customer Order Services application.

In this section, we are going to see the different configuration pieces for the Liberty server to run the Customer Order Services application. As said above, this is done by editing the server.xml file which lives in `/home/vagrant/wlp/usr/servers/defaultServer`. 

You can manually edit this server.xml file yourself using your prefered editor or you can also do so in eclipse:

![Source migration 47](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source47.png)

<sup>*</sup>Also, you can directly replace your existing server.xml file with an already configured server.xml version that can be found [here](tutorialConfigFiles/step1/server.xml) and just read through the below explanation of it for your information.

#### 1. Features

First of all, we need to identify the Java EE features we want our Liberty server to provide us with in order to run our application. In our Customer Order Services application, the main Java EE features we use are:

* JPA
* JAX-RS
* SERVLET
* EJB
* JDBC
* JSON
* JAAS

We need to define all these features in the _featureManager_ section within the server.xml configuration file so that the WebSphere Application Server Liberty server loads them during its startup and make them available to your application. As mentioned earlier, WebSphere Application Server Liberty is dynamic and flexible. That is, the run time loads only what your application needs and recomposes the run time in response to configuration changes.

```
<!-- Enable features -->
<featureManager>
    <feature>jpa-2.0</feature>
    <feature>jaxrs-1.1</feature>
    <feature>jsonp-1.0</feature>
    <feature>servlet-3.1</feature>
    <feature>jdbc-4.1</feature>
    <feature>ejbLite-3.1</feature>
    <feature>appSecurity-2.0</feature>
    <feature>localConnector-1.0</feature>
</featureManager>
```

However, we need to install such features in order for the Liberty server to load them and make them available to the app for it to run correctly. **We will explain how to install all the features above in the next section** [Run the application](#run-the-application).

Extra info:

If you want to check all the features your WebSphere Application Server Liberty server has installed at any point you can run

`~/wlp/bin/featureManager featureList feature_report.xml`

and a *feature_report.xml* file will be created with a list of all the features your Liberty server comprises of. For an quicker and easier first look you can then execute

`cat ~/wlp/bin/feature_report.xml | grep "feature name="`

and you will see an output similar to the following which lists the features WebSphere Application Server Liberty server comes with out of the box

![Source migration 59](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source59.png)

<sup>*</sup>*We can see in the list above that the jaxrs-1.1 feature does not come installed out of the box and that is the reason for the errors we saw in previous [Set up your development environment](#set-up-your-development-environment) section. If you need to manually install features to your WebSphere Application Server Liberty server check the [installUtility command](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/rwlp_command_installutility.html)*

#### 2. Application

Add the following lines to your server.xml file to tell the Liberty server what application to run and their class loaders:

```
<!-- Define application and its classloaders -->
<application id="customerOrderServicesApp" location="${shared.app.dir}/CustomerOrderServicesApp.ear" name="CustomerOrderServicesApp.ear" type="ear">
    <classloader apiTypeVisibility="spec, ibm-api, third-party"/>
</application>
```

#### 3. Http Endpoint

Add the following lines to your server.xml file to tell the Liberty server what is going to be the http endpoint for the Customer Order Services web application:

```
<!-- To access this server from a remote client add a host attribute to the following element, e.g. host="*" -->
<httpEndpoint host="*" httpPort="9080" httpsPort="9443" id="defaultHttpEndpoint"/>
```

#### 4. Application security

Add the following lines to your server.xml file to set you application security so that only secure users can access the application:

```
<!-- User and group security definitions -->
<basicRegistry id="basic" realm="customRealm">
    <user name="rbarcia" password="bl0wfish"/>
    <group name="SecureShopper">
        <member name="rbarcia"/>
    </group>
</basicRegistry>
```

#### 5. Data sources

Add the following lines to your server.xml file to define your application data sources as well as what jdbc drivers and properties the Liberty server needs to use in order to access the application's data:

```
<!-- DB2 library definition -->
<library apiTypeVisibility="spec, ibm-api, third-party" id="DB2JCC4Lib">
    <fileset dir="/home/vagrant/db2lib" includes="db2jcc4.jar db2jcc_license_cu.jar"/>
</library>
    
<!-- Data source definition -->
<dataSource id="OrderDS" jndiName="jdbc/orderds" type="javax.sql.XADataSource">
    <jdbcDriver libraryRef="DB2JCC4Lib"/>
    <properties.db2.jcc databaseName="ORDERDB" password="db2user01" portNumber="50000" serverName="localhost" user="db2inst1"/>
</dataSource>
```
#### 6. Expand WAR and EAR files

Add the following lines to your server.xml file to configure your Liberty server to expand WAR and EAR files at startup so that you dont need to it manually:

```
<!-- Automatically expand WAR files and EAR files -->
<applicationManager autoExpand="true"/>
```

**IMPORTANT:** At this point, you should already have the server.xml with the needed configuration to successfully run the Customer Order Services application. However, **you still need to install the Liberty server features** described earlier in this section. In order to install them, you need to execute:

`~/wlp/bin/installUtility install <server_name>`

where *<server_name>* is the name you have given to your Liberty server when you created it at the beginning of this tutorial. If you have followed the instructions as they are, the Liberty server name should be *defaultServer*.

![Source migration 60](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source60.png)

## Run the application

In order to locally run our application now that we seem to have the appropriate source code of the application and the server configuration migrated to WebSphere Liberty, we right-click on the CustomerOrderServicesApp project and select Export --> EAR file.

We are presented with a dialog to export our project as an EAR file. In this dialog, we must give our EAR project the appropriate name **CustomerOrderServicesApp** and the proper destination **/home/skytap/PurpleCompute/wlp/usr/shared/apps/CustomerOrderServicesApp.ear**. Finally, we optimize the application to run on WebSphere Application Server Liberty and select the Overwrite existing file option in case there was an existing application with the same name already. Click Finish.

![Source migration 21](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source21.png)

Once the EAR project has been exported as an EAR file into the shared applications folder for WebSphere Liberty (and the application itself as a result), we go into the Servers tab in eclipse, right click on WebSphere Application Server Liberty at localhost and click on start. We should get moved to the Console tab in eclipse right away in order to see the WebSphere Liberty's output. We should then see something similar to:

![Source migration 22](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source22.png)

where we can find the links for the two web applications deployed into WebSphere Liberty. One is a test project we will ignore for the time being. The other is the Customer Order Services Web Application and should be accessible at http://localhost:9080/CustomerOrderServicesWeb/. Go ahead and click on that link in the Console tab in eclipse or open a web browser yourself and point it to that url.

You will first be presented with a log in dialog since we have security for our application enabled as you can find out in the server.xml file.

![Source migration 23](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source23.png)

Introduce **rbarcia** as the username and **bl0wfish** as the password for the credentials. Once you have done that, you should be able to see the Customer Order Services Web Application displayed in your web browser. However you will most likely be presented with an error on screen:

![Source migration 24](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source24.png)

If we go into the Console tab for WebSphere Liberty Server in eclipse we will see the following trace:

![Source migration 25](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source25.png)

This means that _CustomerOrderResource.java_ couldn't find _ejblocal:org.pwte.example.service.CustomerOrderServices_ during the lookup of it. The reason for this is what we have already seen during the Software Anaylisis section above in regards to the EJB lookups in WebSphere Liberty. More precisely, you can find the explanation in the **XML File Review** section above. However, the Software Analyzer did not raised these lookups since they were not done through bindings or the @EJB annotation but through initial context lookups:

![Source migration 26](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source26.png)

As a result, we need to change the three web resources within the Web project so that the inital context lookups succeed. Hence, change the initial context lookups located in the following files:

![Source migration 27](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source27.png)

Those lookups should look like the following respectively:

```
java:app/CustomerOrderServices/ProductSearchServiceImpl!org.pwte.example.service.ProductSearchService
java:app/CustomerOrderServices/CustomerOrderServicesImpl!org.pwte.example.service.CustomerOrderServices
java:app/CustomerOrderServices/ProductSearchServiceImpl!org.pwte.example.service.ProductSearchService
```

Once we have corrected the way EJB lookups are specified in WebSphere Liberty, we export the EAR project again and run the application. After loging into the application we should now see the Customer Order Services Application:

![Source migration 28](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source28.png)

However, if we look into the Console tab for WebSphere Liberty in eclipse, we still see errors:

![Source migration 29](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source29.png)

Looking carefuly at them, we see there is a problem with the data type problem with the data that is returned from the database and that this happens in the loadCustomer method in CustomerOrderServicesImpl.java. So is we look into that method we soon realise this is only trying to return an AbstractCustomer from the database:

![Source migration 30](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source30.png)

Therefore, the problem must reside in the AbstractCustomer class. However, as the name suggests, this is an abstract class and thus it will not be instantiated. Instead, we need to look for the classes that extends such abstract class. These are BusinessCustomer and ResidentialCustomer. If we remember the SQL error we have in the WebSphere Liberty Console log, it is about a value 'Y' being returned as an integer. If we then look at the Java classes we realise that some boolean attributes, which will get values of 'Y' and 'N', are being returned as Integer causing the SQL exception.

The reason for this is that the OpenJPA driver treats booleans differently based on its version. In this case, the OpenJPA driver version we are using in WebSphere Liberty does not convert 'Y' or 'N' database values into booleans automatically. As a result, we need to store them as Strings and check those Strings to return a boolean value:

![Source migration 31](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source31.png)
![Source migration 32](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source32.png)

Again, save all the changes, export the EAR project to the WebSphere Liberty folder and start the Server up. You should now see the Customer Order Services web application with no errors at all either on the browser or in the Console tab for WebSphere Liberty in eclipse:

![Source migration 33](https://github.com/ibm-cloud-architecture/refarch-jee/blob/master/static/imgs/toLiberty/Source33.png)

When completed, stop the Liberty server.
