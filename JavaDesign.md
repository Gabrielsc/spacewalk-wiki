# Java Design

The Java version of RHN employs a standard three tier web application model


        UI   ----    Business logic    ----    Persistence layer

Each of these layers has a defined type of entry point.  For the UI layer, we
use Struts, so our defined entry point into the Java code is an extension
of the Struts Action class.  The business logic layer is implemented using
the Session bean facade model, although we don't use session beans.  All
entries into the business logic layer happens through our Manager classes, 
which are implemented as a set of static methods.  The Persistence layer
is implemented using [Hibernate](http://hibernate.org/), a replacement for entity beans, and all 
entry points to the Persistence layer extend the `HibernateFactory` class.
# The UI

Our UI is implemented using Struts as the controller and JSPs as the view.

All web requests should be directed to the Struts servlet, and Struts is 
used to direct the request to the correct Java code for that request.  There 
is no business logic done in the UI.  All of our UI logic wraps the values
retrieved from the form into a Data Transfer Object (DTO) and forwards that
DTO to the appropriate Manager class in the Business tier.  After the work is
done in the Business tier, the appropriate DTOs are returned to the UI layer
and are attached to the request for use within the JSP.

Just as there is no business logic allowed within our Struts actions, there is
no business logic allowed in our JSP code or our taglibs.  Our JSPs are for
display only, and they take the objects associated with the request and 
format that information correctly for the UI.  Our taglibs are likewise
restricted to display logic only.  We use taglibs to centralize common display
tasks.  For example, displaying a list of data is done through a taglib.

Our web services UI is implemented using a combination of [Redstone XML-RPC](http://xmlrpc.sourceforge.net/).  Both of these tools are abstracted out to the point that if the
web services implementer extends the `BaseHandler` class the APIs will be available through XML-RPC.

# Business logic

Our business logic tier is implemented as a set of static methods, which perform all security checks, and then make calls to the correct business
objects to perform the work.  This tier is slightly muddied today, because
it is much heavier weight than it should be.  The problem is the amount of
logic that we have embedded in stored procedures.  Because so much of our
logic is in the database, we have the Manager classes make calls directly
to the DB instead of utilizing our business objects to do that work for us.

The business logic tier is implemented so that it is the common place for the
web UI and the web services APIs to plug into the actual implementation.  This
allows a single implementation for all of the business logic that we expose.

Manager classes are only allowed to call directly into their related Factory
class.  For example, the `UserManager` can call the `UserFactory`, but it can't
call the `ServerFactory`.  If the `UserManager` wishes to retrieve Server 
information, it must call through the `ServerManager` to do so.  This ensures
that no matter what is happening, security checks are performed, because the
Managers take care of all security checks.

# Business object/Web form validation
A common problem for web applications is web form and business object 
validation.  While Struts provides a mechanism for validating web forms, it
isn't available for use in the business tier.  To resolve this problem, we have
implemented our own validation service that can be used by both Struts and our
business layer.

# Persistence

We use Hibernate for our persistence layer, which allows us to persist standard
Java objects.  We have created a standard `HibernateFactory` class, which 
implements all of our persistence logic.  Each major business object class has
its own factory which extends the `HibernateFactory`.  For example, the User
object has a `UserFactory` class which knows how to persist `Users`, `Addresses`,
and `EmailAddresses`.


    TODO: add note about DataSource

# Internationalization/Localization

Java is already very strong in i18n/l10n, but we had some requirements that
Java didn't handle out of the box.  Our resolution for this was to replace
the standard `ResourceBundle` concept from Java with an `XmlResourceBundle`.  This
leverages Java's strengths in the i18n/l10n arena, but allows us to store our
localized strings in an XML format.  The main reason for doing this, was so 
that we could use standard English phrases as our keys, which `ResourceBundles`
didn't allow.

# Security
Security in the Java code is done completely differently from how it is done
in the perl code.  Instead of doing security based on formvar name or XML-RPC
parameter name, all of the security checks are done in code.  This is not done
by adding if checks throughout the Java code.  Rather, those if statements are
added in very few, very strategic locations, such as loading a System or
loading a User from the DB.  After that, the DB itself is used to provide
security.  For example, when loading a scheduled Action from the DB, the code
should not look like:



         action = ActionFactory.lookupById(actId);
         if (action.getOrg() != loggedInUser.getOrg()) {
             throw PermissionException();
         }


Instead, the logic is:



         action = ActionManager.lookupByIdAndOrgId(actId,
                                                   loggedInUser.getOrg().getId());


The SQL query used to lookup the Action ensures that the Org id's are the
same.  This limits the security to relationships within the DB.  For more fine
grained security, we have a couple of options.

1)  Add a writable flag to all of our domain objects, and have the lookup
method be responsible for setting that flag.  If writable is false, then the
commit code (1 method) can throw a permission exception.

2)  Extend the SQL queries for lookup something up and add a flag that
indicates why you are looking up an object.  For example, if you are looking
up an object for display purposes, you call lookupByIdReadOnly, which just
checks for the read-only security level and returns a read-only object.

For security checks against large groups of objects, we will need to change
the methodology a bit.  First, the ideal is that if you can access the group,
then you can take any action on any server in the group.  This means that
instead of checking security on every server in the group, you can check it
once, on the group itself.  If that doesn't work, then we will need to modify
the DB schema a bit.  The idea will be the same, that all security checks
happen on the group, but the group's security will be the union of the 
security for the group members.
# Server integration

Our code base runs on Tomcat and integrates with Apache through mod_jk.  While

we could have used mod_proxy to integrate with Apache, it removed a lot of
flexibility that we had with mod_jk,  For example, the ability to have multiple
Tomcat instances attached to a single Apache box.  The complete explanation
for why we chose mod_jk can be read in:   rhn-svn/docs/mod_jk_explanation.txt.
