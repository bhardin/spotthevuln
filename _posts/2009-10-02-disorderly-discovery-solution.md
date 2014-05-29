---
layout: solution
tags: Cross-Site Scripting (XSS)
---

# Solution: Disorderly Discovery

## Details

* __Vulnerability Type__: Cross Site Scripting (XSS)
* __Affected Software:__ JSPWiki
* __Fixed in Version:__ 2.5.164

## Description
This is a classic Cross-Site Scripting vulnerability which affected JSPWiki.

The `setId()` method assigns a non-sanitized value to the `m_tabID` variable. The `m_tabID` variable is then later used in the `doWikiStartTag()` method as part of a string buffer which is used to build dynamic HTML. The m_tabID variable is never encoded or sanitized before being sent back to the user.

In addition, to the `setId()` method, the `setTitle()`, `setAccessKey()`, and `setUrl()` methods also expose a potential for Cross-Site Scripting. The JSPWiki team used `TextUtil.replaceEntities()` in each of the vulnerable methods to sanitize the values being assigned to the respective variables.

__If you had to write the `TextUtil.replaceEntities()` method, what would it look like?__

## Developers Solution

```diff
public class TabTag extends WikiTagBase {
  private static final long serialVersionUID = -8534125226484616489L;
  private String m_accesskey;
  private String m_tabID;
  private String m_tabTitle;
  private String m_url;

    public void setId(String aTabID) {
-     m_tabID = aTabID;
+     m_tabID = TextUtil.replaceEntities( aTabID );
    }

    public void setTitle(String aTabTitle) {
-     m_tabTitle = aTabTitle;
+     m_tabTitle = TextUtil.replaceEntities( aTabTitle );
    }

    public void setAccesskey(String anAccesskey) {
-     m_accesskey = anAccesskey; //take only the first char
+     m_accesskey = TextUtil.replaceEntities( anAccesskey ); //take only the first char
    }

    public void setUrl( String url ) {
-     m_url = url;
+     m_url = TextUtil.replaceEntities( url );
    }

  public int doWikiStartTag() throws JspTagException {
    TabbedSectionTag parent = (TabbedSectionTag) findAncestorWithClass(this, TabbedSectionTag.class);

    if (m_tabID == null) {
      throw new JspTagException("Tab Tag without \"id\" attribute");
    }
    if (m_tabTitle == null) {
      throw new JspTagException("Tab Tag without \"tabTitle\" attribute");
    }
    if (parent == null) {
      throw new JspTagException("Tab Tag without parent \"TabbedSection\" Tag");
    }

    if (!parent.isStateGenerateTabBody()) return SKIP_BODY;

    StringBuffer sb = new StringBuffer(32);

    sb.append("&lt;div id=\"" + m_tabID + "\"");

    if (!parent.validateDefaultTab(m_tabID)) {
      sb.append(" class=\"hidetab\"");
    }
    sb.append(" &gt;\n");

    try {
      pageContext.getOut().write(sb.toString());
    } catch (java.io.IOException e) {
      throw new JspTagException("IO Error: " + e.getMessage());
    }

    return EVAL_BODY_INCLUDE;
  }
}
```