---
layout: post
title: Meaningless
tags:
- Code Snippet
status: publish
type: post
published: true
meta:
  _sexybookmarks_permaHash: 8ea88e20a72a22876b642ca00a279bc7
  _sexybookmarks_shortUrl: http://bit.ly/d0r2Ph
  aktt_notify_twitter: 'yes'
  _aktt_hash_meta: ''
  _edit_last: '2'
  bfa_ata_body_title: ''
  bfa_ata_display_body_title: ''
  bfa_ata_body_title_multi: ''
  bfa_ata_meta_title: ''
  bfa_ata_meta_keywords: ''
  bfa_ata_meta_description: ''
  aktt_tweeted: '1'
---
<blockquote><strong>Facts are meaningless. You could use facts to prove anything that's even remotely true.
- Homer Simpson</strong></blockquote>
[ccnLe_java]

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

        String searchFilter = "(&amp;(objectClass=*)(userPrincipalName=" + userPrincipalName + "))";

        NamingEnumeration answer = ldapContext.search(searchBase, searchFilter, searchCtls);

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
     * to role names.  This implementation uses the <a href="mailto:{@link">{@link</a> #groupRolesMap} to map group names to role names.
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

[/ccnLe_java] 
