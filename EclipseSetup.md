## Introduction

This is a small document to help developers configure Eclipse for

use with Spacewalk's java code.  

For now, I use Eclipse to create and edit classes and run unit tests.
You will still need to go to the command line to make sure things still
work in the common build environment using Ant.
## Install from your Distro:




       yum install ivy ant-contrib

NOTE: On my Fedora 24 system, I had to install the apache-ivy package instead of ivy to avoid conflicts between apache-commons and jakarta-commons.  I also had to install ant-junit package.  I don't know if these packages are necessary on other OS version.


       dnf install apache-ivy ant-contrib ant-junit
## Install Eclipse from your Distro:




       yum install eclipse
## OR Install Eclipse from GZIP file



Go here to grab the latest rev of eclipse.  Make sure you get the gtk version

  http://www.eclipse.org/downloads/

Download Eclipse SDK 3.X.X. to /tmp, where .X.X translates to the version they display on their downloads page.  Make sure in the below commands that you substitute the proper version.

tar/gunzip the downloaded file:

       tar xvzf eclipse-SDK-3.X.X-linux-gtk.tar.gz
This will create a new subdir with a bunch of files in ./eclipse

Now, as root, move ./eclipse to /opt/eclipse-3.X.X 

      # su root
      # mv ./eclipse /opt/eclipse-3.X.X
      # cd /opt
      # ln -s eclipse-3.X.X eclipse
If you downloaded / extracted this to a directory below your home directory, you will not be able to see it once you become root.  You will need to move it to /tmp, then become root and move it to /opt.

Now skip down below to the Create Project Directory step.
## Setup the Project directory

1) Create a directory called workspace in your home directory


       cd ~/
       mkdir workspace
2) In that workspace directory, create a symlink to your java directory.
   NOTE: assumes your checkout is in ~/spacewalk

       cd ~/workspace
       ln -s ~/spacewalk/java spacewalk
3) cd to ~/spacewalk/java/.  We use ivy to resolve our jar dependencies.  Run the following cmd below to properly setup some of your .classpath entries.


       mkdir -p build/test-lib
       mkdir -p build/build-lib
       ant make-eclipse-project



4) Start Eclipse.

       eclipse
or if you installed manually


       /opt/eclipse/eclipse
5) Once, Eclipse is started setup the project:

   - File -> Import
   - Expand "General"
   - select "Existing projects into workspace"
   - Click next
   - Browse or enter  ~/workspace/spacewalk
   - click "Finish"
## Code generation




If you want to configure Eclipse to generate code to match the coding
standards follow the following steps.

Configure Code Formatter:

    1) Click Window -> Preferences
    2) In the Preferences dialog, Click Java -> Code Style -> Code Formatter
    3) Click the Import... button.
    4) Browse to the ~/spacewalk/java/conf/eclipse/code_formatter_rhn.xml
    5) Click OK
Configure Code Templates:

    1) Click Window -> Preferences
    2) In the Preferences dialog, Click Java -> Code Style -> Code Templates
    3) Select Comments
    4) Click the Import... button.
    5) Browse to the ~/spacewalk/java/conf/eclipse/code_templates_comments.xml
    6) Click OK
    7) Select Code
    8) Click the Import... button.
    9) Browse to the ~/spacewalk/java/conf/eclipse/code_templates_code.xml
    10) Click OK
Configure import statement order:

    1) Click Window -> Preferences
    2) In the Preferences dialog, Click Java -> Code Style -> Organize Imports
    3) Click the Import... button.
    4) Browse to the ~/spacewalk/java/conf/eclipse/rhn.importorder
    5) Click OK
## Performance issues  (might not be relevant)



In order to increase Eclipse's performance, add the following to your
/etc/eclipse.conf file, overriding any other memory settings:

    VM_ARGS="-Xverify:none -Xms192M -Xmx256M"

The key one is the verify:none which keeps the VM from validating
all of the class files during load.

If using the Sun JVM, make the following tweaks:

 VM_ARGS="-Xverify:none -XX:+UseParallelGC -XX:PermSize=20M -XX:MaxNewSize=32M -XX:NewSize=32M -Xms192M -Xmx256M"

-------
The above didn't work for me, so I changed eclipse.ini in the eclipse installation directory, /usr/lib64/eclipse/eclipse.ini in my case. I adjusted the vmargs in there.