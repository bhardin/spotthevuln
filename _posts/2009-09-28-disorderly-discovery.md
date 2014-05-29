---
layout: vulnerability
tags: Code Snippet, Java
---

# Disorderly Discovery

> One of the advantages of being disorderly is that one is constantly making exciting discoveries.

> - A. A. Milne

## Vulnerable Code

```java
public class TabTag extends WikiTagBase {
  private static final long serialVersionUID = -8534125226484616489L;
  private String m_accesskey;
  private String m_tabID;
  private String m_tabTitle;
  private String m_url;

  public void setId(String aTabID) {
    m_tabID = aTabID;
  }

  public void setTitle(String aTabTitle) {
    m_tabTitle = aTabTitle;
  }

  public void setAccesskey(String anAccesskey) {
    m_accesskey = anAccesskey; // take only the first char
  }

  public void setUrl(String url) {
    m_url = url;
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
