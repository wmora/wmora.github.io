--- 
title: Installing Oracle Application Server 10g on Windows Server 2003 SP2
date: "2010-08-22T16:17:00.000-03:00"
author: William Mora
tags: 
- Oracle 10g
- OAS 10g
- ADFBCManager
- Windows Server 2003 SP2
- JVM
- Java Virtual Machine
permalink: /2010/08/installing-oracle-application-server.html
---

If you try to install the Oracle Application Server 10g (9.0.4) on a Windows Server 2003 SP2, you may encounter an error when the installer tries to configure the OC4J instances with the following:

```bash
Deploying application 'ADFBCManager' to OC4J instance 'home'.
FAILED!
ERROR: Caught exception while deploying 'ADFBCManager' to 'home':
java.lang.reflect.InvocationTargetException
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
at java.lang.reflect.Method.invoke(Method.java:324)
at oracle.j2ee.tools.deploy.Oc4jDeploy.doDeploy(Unknown Source)
at oracle.j2ee.tools.deploy.Oc4jDeploy.execute(Unknown Source)
at oracle.j2ee.tools.deploy.Oc4jDeploy.deploy(Unknown Source)
at oracle.j2ee.tools.deploy.Oc4jDeploy.main(Unknown Source)
Caused by: com.evermind.client.orion.AdminCommandException: Deploy error: deploy failed!: ; nested exception is:
oracle.oc4j.admin.internal.DeployerException: Error initializing ejb-module; Exception Error in application ADFBCManager: Error loading package at file:/C:/oraOAS10g/j2ee/home/applications/ADFBCManager/bc4joembean.jar,
Error compiling C:\oraOAS10g\j2ee\home\applications\ADFBCManager\bc4joembean.jar: Syntax error in source
at com.evermind.client.orion.DeployCommand.execute(DeployCommand.java:90)
at com.evermind.client.orion.Oc4jAdminConsole.executeCommand(Oc4jAdminConsole.java:139)
... 8 more
...
```

The reason for this is because one of Microsoft's security updates (MS09-012 or KB956572). When the patch is installed, it prevents any JAVA Virtual Machine to run properly.
If you encounter this issue during the OAS installation (or with any other application running a JVM), do a clean deinstall of the application server. Next, download Microsoft's patch to solve this issue (click [here](http://www.microsoft.com/downloads/details.aspx?FamilyID=972ba7c5-54df-4b0f-819b-4405bbbed291&displaylang=en) to download and remember to download it in the correct language). Restart your server after the patch is installed, run the Oracle Universal Installer again to install the Oracle Application Server and the installation should be successful now.

Cheers!