## Navigation

### Main navigation (top bar and left side bar)


Spacewalk has three navigation menu bars: topnav, left nav, and dialog nav. The top nav is the one you see at the top of every page. It is used to navigate to the main pages of the application.


![Alt](images/topnav.png?raw=True)

The left nav is located, you guessed it, on the left and offers a more detailed level of navigation under the main tab.

![Alt](images/leftnav.png?raw=True)

These navigation bars are handled by two files:

 * sitenav-authenticated.xml  (for authenticated sessions)
 * sitenav.xml (for sessions that have not logged in)

Each of these files exist in java. There used to be independent perl versions in `/eng/web/html/nav/`.

Looking at /eng/java/code/webapp/WEB-INF/nav/sitenav-authenticated.xml, it is pretty self explanatory.  The following tags are valid:

||rhn-tab ||Defines a tab within the navigation bars.
||rhn-tab-url || Used when specifying more than one url that corresponds to a tab. (When a user is browsing to that url, the tab will be shaded to indicate it is active).
||rhn-tab-directory ||Used to specify a directory from which to match urls.

Each tag has a set of attributes:

|  *Tag*      |  *Attribute*  |  *Description*  |
| --- | --- | --- |
|  *rhn-navi-tree*  |  label  |  internal label used by parser  |
|                 |  title-depth  |  how deep in the tree to parse to generate the title of the page  |
|                 |  invisible  |  1 if you want to hide the top level of the navigation used by sub-navigation files  |
|                 |  formvar  |  sets the default formvar to be added to all urls  |
|                 |  acl-mixins  |  a comma separated list of fully qualified ACLHandler classnames  |
|                 |              |   |  |
|  *rhn-tab*  |  name  |  i18n key used to lookup the label shown in the UI for this tab. Maybe plain English i.e. Errata, or channel.nav.all, as long as it maps to a StringResource value.  |
|           |  url  |  the url, usually relative, where the tab is to link  |
|           |  acl  |  the [ACL]() guarding the tab, used to conditionally display the tab  |
|           |  active-image  |  image filename used to highlight the tab when active (selected)   |
|           |  inactive-image  |  image filename used to highlight the tab when inactive (unselected)   |
|                 |              |   |  |
|  *rhn-tab-url*  |  NONE   |  this takes in a value of url no attributes `<rhn-tab-url>/rhn/account/EditAddress.do</rhn-tab-url>`  |
|                 |              |   |  |
|  *rhn-tab-directory*  |  NONE   |  this takes in a value of url no attributes `<rhn-tab-directory>/rhn/users</rhn-tab-directory>`  |

Below is a sample sitenav file.

![Alt](images/sitenav.png?raw=True)
### Sub-navigation

On many pages (System details page, Channel detail page, etc..) there is a tab bar within the page.  These are defined by files within separate files from the sitenav.xml and sitenav-authenticated.xml.  Within the same directories listed above are files called:


 * action_detail.xml
 * channel_detail.xml
 * system_detail.xml
 * and others like them

These files are exactly like the sitenav navigation files, but contain information only for these sub-navigation bars.
### ACLs


### Java side acls

To control whether a tab is displayed at a certain time or not, you can use the built in ACLs (or make your own).  Within an rhn-tab tag, simply specify acl="name(arg)".  For example within sitenav-authenticated.xml:



     <rhn-tab name="Users" url="/rhn/users/ActiveList.do" acl="org_entitlement(sw_mgr_enterprise); user_role(org_admin)" 
      active-image="tab-users-selected.gif" inactive-image="tab-users.gif">

Here this Users tab would not be shown if either `org_entitlement(sw_mgr_enterprise)` or `user_role(org_admin)` returns false.  Both of these are defined in `com.redhat.rhn.common.security.acl.Access`.  The name `org_entitlement` is translated into `aclOrgEntitlement()` when calling the acl, so `Access.aclOrgEntitlement()` is invoked.

Additional ACL classes exist for the sub-tab xml files.  Within [com.redhat.rhn.common.security.acl](http://spacewalk.redhat.com/documentation/javadoc/com/redhat/rhn/common/security/acl/package-summary.html) exists several handlers such as ChannelAclHandler, ErrataAclHandler.  These are used with their corresponding X_detail.xml file.  For example, the ChannelAclHandler is for use with channel_detail.xml.
### Behind the scenes

I'm sure you want to know all of the magic that goes on behind the scenes. The driving code is the [NavDigester](https://github.com/spacewalkproject/spacewalk/blob/master/java/code/src/com/redhat/rhn/frontend/nav/NavDigester.java) class. It uses `commons-digester` to parse the navigation files and create the appropriate NavTree.



    #!java
    /**
     * Copyright (c) 2009 Red Hat, Inc.
     *
     * This software is licensed to you under the GNU General Public License,
     * version 2 (GPLv2). There is NO WARRANTY for this software, express or
     * implied, including the implied warranties of MERCHANTABILITY or FITNESS
     * FOR A PARTICULAR PURPOSE. You should have received a copy of GPLv2
     * along with this software; if not, see
     * http://www.gnu.org/licenses/old-licenses/gpl-2.0.txt.
     *
     * Red Hat trademarks are not licensed under GPLv2. No permission is
     * granted to use or replicate Red Hat trademarks that are incorporated
     * in this software or its documentation.
     */
    
    package com.redhat.rhn.frontend.nav;
    
    import org.apache.commons.digester.Digester;
    
    import java.net.URL;
    
    /**
     * Helper class to parse a sitenav.xml file, returning the tree
     * @version $Rev$
     */
    public class NavDigester {
        // no contruction, please
        private NavDigester() { }
    
        /**
         * buildTree, method to take a url and parse the contents
         * into a NavTree
         * @param url the file to parse
         * @return NavTree the tree represented by the file
         * @throws Exception if something breaks. XXX: fix to be tighter
         */
        public static NavTree buildTree(URL url) throws Exception {
            if (url == null) {
                throw new IllegalArgumentException("URL is null, your definition tag " +
                        "probably points to a non existing file.");
            }
            Digester digester = new Digester();
            digester.setValidating(false);
    
            digester.addObjectCreate("rhn-navi-tree", NavTree.class);
            digester.addSetProperties("rhn-navi-tree");
            digester.addSetProperties("rhn-navi-tree",
                                      "acl_mixins",
                                      "aclMixins");
    
            digester.addObjectCreate("*/rhn-tab", NavNode.class);
            digester.addSetProperties("*/rhn-tab",
                                      "active-image",
                                      "activeImage");
            digester.addSetProperties("*/rhn-tab",
                                      "inactive-image",
                                      "inactiveImage");
    
            digester.addCallMethod("*/rhn-tab",
                                   "addPrimaryURL",
                                   1);
            digester.addCallParam("*/rhn-tab",
                                  0,
                                  "url");
    
            digester.addCallMethod("*/rhn-tab/rhn-tab-url",
                                   "addURL",
                                   0);
            digester.addCallMethod("*/rhn-tab/rhn-tab-directory",
                                   "addDirectory",
                                   0);
    
            digester.addSetNext("*/rhn-tab", "addNode");
            return (NavTree)digester.parse(url.openStream());
        }
    }

[Digester](http://commons.apache.org/digester/) offers an easy way to define how to build objects from an XML file. Given a sitenav.xml file, it will create a [NavTree](https://github.com/spacewalkproject/spacewalk/blob/master/java/code/src/com/redhat/rhn/frontend/nav/NavTree.java) object which contains a `NavNode` for each of the rhn-tab tags found. Once the `NavTree` is created, we then pass it to the [NavTreeIndex](https://github.com/spacewalkproject/spacewalk/blob/master/java/code/src/com/redhat/rhn/frontend/nav/NavTreeIndex.java) to index the tree by urls so that it makes it easier to do matching while rendering the UI. This allows the appropriate tab to be highlighted as well as the page title to be set properly.


The algorithm used by `NavTreeIndex` is pretty simple, we first harvest the primary urls, and any subsequent urls and directory definitions into separate maps. The real work happens when we browse a url and need to determine which is the currently active node. That's where `computeActiveNodes` method comes into play. We split the given url into an array making it more general. For example, given /rhn/users/ActiveList.do we create an array with 3 entries: "/rhn/users/ActiveList.do", "/rhn/users", "/rhn". We then try to find the `NavNode` that best matches those URLs. This gives us our active node and we know which `NavNode` to render and thus highlight in the UI.


    #!java
    
    // NavTreeIndex excerpt
        public String computeActiveNodes(String url, String lastActive) {
            String[] prefixes = splitUrlPrefixes(url);
    
            // If we have a lastActive URL we 
            // will add it to the end of the list of URLs to 
            // use it as a last resort.
            if (lastActive != null) {
                String[] urls = new String[prefixes.length + 1];
    
                // Add the lastActive to the end
                for (int i = 0; i < prefixes.length; i++) {
                    urls[i] = prefixes[i];
                }
                urls[prefixes.length] = lastActive;
                prefixes = urls;
            }
    
            return computeActiveNodes(prefixes);
        }
        private String computeActiveNodes(String[] urls) {
            bestNode = findBestNode(urls);
            if (bestNode == null) {
                // can't find an best node. assume topmost leftmost node is best
                ArrayList depthZero = (ArrayList)nodeLevels.get(0);
                bestNode = (NavNode)depthZero.get(0);
            }
    
            NavNode walker = bestNode;
    
            activeNodes = new HashSet();
            while (walker != null) {
                activeNodes.add(walker);
                walker = (NavNode)childToParentMap.get(walker);
            }
    
            if (log.isDebugEnabled()) {
                log.debug("returning [" + bestNode.getPrimaryURL() +
                          "] as the url of the active node");
            }
            return bestNode.getPrimaryURL();
        }
    
        private NavNode findBestNode(String[] urls) {
    
            for (int i = 0; i < urls.length; i++) {
    
                if (log.isDebugEnabled()) {
                    log.debug("Url being searched [" + urls[i] + "]");
                }
                // first match by the primary url which is the
                // first rhn-tab-url definition in the sitenav.xml.
                if (primaryURLMap.get(urls[i]) != null) {
                    if (log.isDebugEnabled()) {
                        log.debug("Primary node for [" + urls[i] + "] is [" +
                                primaryURLMap.get(urls[i]) + "]");
                    }
    
                    // we found a match, now let's make sure it is accessible
                    // we need to do this because sometimes there are multiple
                    // nodes with the same url.  At that point they are 
                    // distinguishable only by acls.
    
                    if (canViewUrl((NavNode)primaryURLMap.get(urls[i]), 0)) {
                        return (NavNode)primaryURLMap.get(urls[i]);
                    }
                }
    
                // either we couldn't find a primary url match OR it isn't
                // accessible.  Let's go through the other url mappings (if any)
                // looking for an accessible url.
    
                List nodesByUrl = (List) nodeURLMap.get(urls[i]);
                if (nodesByUrl != null) {
                    Iterator nodeItr = nodesByUrl.iterator();
                    while (nodeItr.hasNext()) {
                        NavNode next = (NavNode)nodeItr.next();
                        if (canViewUrl(next, 1)) {
                            if (log.isDebugEnabled()) {
                                log.debug("Best node for [" + urls[i] + "] is [" +
                                        primaryURLMap.get(urls[i]) + "]");
                            }
                            return next;
                        }
                    }
                }
    
                // finally, we couldn't find a match by primary url, nor by
                // any of the other mappings.  At this point we will attempt
                // to match by directory if there was an rhn-tab-directory
                // definition.  Otherwise, we're just going to bail and return
                // null.
    
                if (nodeDirMap.get(urls[i]) != null) {
                    List nodes = (List)nodeDirMap.get(urls[i]);
                    // what do we do with a list that contains
                    // more than one.
                    if (log.isDebugEnabled()) {
                        log.debug("Best node for [" + urls[i] + "] is [" +
                                nodes.get(0) + "]");
                    }
                    return (NavNode)nodes.get(0);
                }
            }
    
            return null;
        }
    
       // ...
#### Rendering

Navi has a [RenderEngine](https://github.com/spacewalkproject/spacewalk/blob/master/java/code/src/com/redhat/rhn/frontend/nav/RenderEngine.java) that is used to display the menus of varying levels. The `RenderEngine` takes in a `NavTreeIndex` and a URL. As part of the rendering process we had a need to render the menus in different ways and also turn off some based on [ACLs]().


To accomplish the displaying of the menus, Navi has a [Renderable](https://github.com/spacewalkproject/spacewalk/blob/master/java/code/src/com/redhat/rhn/frontend/nav/Renderable.java) abstract class used to implement different `Renderers`. Out of the box Navi has five (5) renderers:
|| DialogNavRenderer || ![Alt](images/dialognav.png?raw=True)||
|| SidenavRenderer ||  ![Alt](images/leftnav.png?raw=True)||
|| TextRenderer || ![Alt](images/textnav.png?raw=True)||
|| TitleRenderer || ![Alt](images/title.png?raw=True) ![Alt](images/title-html.png?raw=True)

|  TopnavRenderer  |  ![Alt](images/topnav.png?raw=True) |
| --- | --- |

Lastly, the [NavMenuTag](https://github.com/spacewalkproject/spacewalk/blob/master/java/code/src/com/redhat/rhn/frontend/taglibs/NavMenuTag.java) is a jsp tag used to actually render the menu to the screen. The NavMenuTag determines the url from the Request object and calls into the NavTreeIndex to determine what to output.

For example, to render the TopNav the following tag call is used:

       <rhn:menu mindepth="0" maxdepth="0"
                 definition="/WEB-INF/nav/sitenav-authenticated.xml"
                 renderer="com.redhat.rhn.frontend.nav.TopnavRenderer" />

The `definition` attribute specifies the xml file used to define this menu, while the `renderer` defines how we want to display the menu. In this example we are limiting the depth to the top level.
#### Guarding

Guarding is the ability to enable/disable the rendering of a given node based on any boolean expression as defined by the [ACL system](), be it a configuration entry, dynamic application data, or even assigned user roles. Navi takes the same approach to guarding as it did with the rendering, there is a [RenderGuard](https://github.com/spacewalkproject/spacewalk/blob/master/java/code/src/com/redhat/rhn/frontend/nav/RenderGuard.java) interface which simply returns `true}} or {{{false` on whether the `NavNode` can be rendered.


Navi contains two (2) guards: [AclGuard](https://github.com/spacewalkproject/spacewalk/blob/master/java/code/src/com/redhat/rhn/frontend/nav/AclGuard.java), which returns the result of evaluating the [ACL](), and [DepthGuard](https://github.com/spacewalkproject/spacewalk/blob/master/java/code/src/com/redhat/rhn/frontend/nav/DepthGuard.java), which evaluates the depth of the menu. 

There is also a need to group RenderGuards, hence the creation of [RenderGuardComposite](https://github.com/spacewalkproject/spacewalk/blob/master/java/code/src/com/redhat/rhn/frontend/nav/RenderGuardComposite.java). RenderGuardComposite takes a list of RenderGuards and evaluates them in order as an *and* operation. First RenderGuard to fail causes the entire operation to return `false`.