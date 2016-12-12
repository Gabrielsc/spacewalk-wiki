#### Unused java code



There is a nice tool UCDetector out there and we have used it to create nice report of unused methods and classes.

This report is not accurate since we use many external libraries (like log4j), Struts framework and also there is some dynamic method invoke stuff. But this report is good starting point when you are in doubt if the code you are looking at is unused.

Please do not remove code if you are not absolutely sure it is not called by an external library or dynamically.

The report was created with the default setting with these exceptions:

  * Reduce visibility warnings: all disabled
  * Ignoring classes extending/implementing: `java.lang.Exception,redstone.xmlrpc.XmlRpcInvocationHandler,javax.servlet.jsp.tagext.TagSupport,org.apache.struts.action.Action`

If you run such a detection please attach the report (saved in workspace/ucdetector_reports) to this page.