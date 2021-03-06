---
layout: post
title: Meaningless - LDAP Injection
tags:
- Apache
- Java
- LDAP Injection
- Shiro
- Solution
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '1'
  bfa_ata_body_title: Meaningless - LDAP Injection
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: Meaningless - LDAP Injection
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
  _sexybookmarks_permaHash: 32fa27aa747b0c81b3b07348af143c75
  _sexybookmarks_shortUrl: http://bit.ly/cxO204
---
## Details
__Affected Software:__ Apache Shiro Project

__Fixed in Version:__  Revision 887987

__Issue Type:__ LDAP Injection

Original Code: <a title="Meaningless" href="http://spotthevuln.com/2010/04/meaningless/" target="_blank">Found Here</a>
## Description
This patch fixes a LDAP injection vulnerability in the Apache Shiro Project. A quick glance of the vulnerable code shows several references to LDAP and LDAP queries scattered throughout the sample code. In this example, we see that the Shiro developers originally used the username to dynamically build a LDAP query. The symptoms are very similar to a typical SQL injection. The username is used in a string building technique to build the LDAP query which eventually gets passed to an LDAP server. If an attacker crafts the proper username, they could have the ability to modify the original LDAP query and execute an arbitrary LDAP query of their choosing.

Just as the symptoms are very similar to SQL injection, the fix also looks very similar to code that would be used to fix a SQL injection vulnerability. The developers helped the application make a distinction between code and data by removing the string built LDAP query
## Developer's Solution
```diff
<div id="_mcePaste">

protected AuthorizationInfo buildAuthorizationInfo(Set&lt;String&gt; roleNames) {

return new SimpleAuthorizationInfo(roleNames);

}

private Set&lt;String&gt; getRoleNamesForUser(String username, LdapContext ldapContext) throws NamingException {

Set&lt;String&gt; roleNames;

roleNames = new LinkedHashSet&lt;String&gt;();

SearchControls searchCtls = new SearchControls();

searchCtls.setSearchScope(SearchControls.SUBTREE_SCOPE);

String userPrincipalName = username;

if (principalSuffix != null) {

userPrincipalName += principalSuffix;

}

-       String searchFilter = "(&amp;(objectClass=*)(userPrincipalName=" + userPrincipalName + "))";

+       //SHIRO-115 - prevent potential code injection:

+       String searchFilter = "(&amp;(objectClass=*)(userPrincipalName={0}))";

+       Object[] searchArguments = new Object[]{userPrincipalName};

-       NamingEnumeration answer = ldapContext.search(searchBase, searchFilter, searchCtls);

+      NamingEnumeration answer = ldapContext.search(searchBase, searchFilter, searchArguments, searchCtls);

while (answer.hasMoreElements()) {

SearchResult sr = (SearchResult) answer.next();

if (log.isDebugEnabled()) {

log.debug("Retrieving group names for user [" + sr.getName() + "]");

}

Attributes attrs = sr.getAttributes();

if (attrs != null) {

NamingEnumeration ae = attrs.getAll();

while (ae.hasMore()) {

Attribute attr = (Attribute) ae.next();

if (attr.getID().equals("memberOf")) {

Collection&lt;String&gt; groupNames = LdapUtils.getAllAttributeValues(attr);

if (log.isDebugEnabled()) {

log.debug("Groups found for user [" + username + "]: " + groupNames);

}

Collection&lt;String&gt; rolesForGroups = getRoleNamesForGroups(groupNames);

roleNames.addAll(rolesForGroups);

}

}

}

}

return roleNames;

}

/**

* This method is called by the default implementation to translate Active Directory group names

* to role names. This implementation uses the {@link #groupRolesMap} to map group names to role names.

*

* @param groupNames the group names that apply to the current user.

* @return a collection of roles that are implied by the given role names.

*/

protected Collection&lt;String&gt; getRoleNamesForGroups(Collection&lt;String&gt; groupNames) {

Set&lt;String&gt; roleNames = new HashSet&lt;String&gt;(groupNames.size());

if (groupRolesMap != null) {

for (String groupName : groupNames) {

String strRoleNames = groupRolesMap.get(groupName);

if (strRoleNames != null) {

for (String roleName : strRoleNames.split(ROLE_NAMES_DELIMETER)) {

if (log.isDebugEnabled()) {

log.debug("User is member of group [" + groupName + "] so adding role [" + roleName + "]");

}

roleNames.add(roleName);

}

}

}

}

return roleNames;

}

}

</div>
```
