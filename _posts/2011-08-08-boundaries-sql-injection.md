---
layout: post
title: Boundaries - SQL Injection
tags: PHP, Solution, SQL Injection
---

# Boundaries - SQL Injection

## Details
* __Affected Software:__ My Calendar Wordpress Plugin
* __Fixed in Version:__ > 1.7.2
* __Issue Type:__ SQL Injection
* __Original Code:__ [Found Here](http://spotthevuln.com/2011/08/boundaries/)

## Details

This week's bug was a subtle mistake in the usage of an escaping routine. It seems the developer understood the dangers of SQL injection and therefore used an escaping routine to sanitize user controlled input before using that input to build a SQL statement. Unfortunately, the developer overlooked a crucial characteristic and used the wrong escaping routine. Looking at the vulnerable line, we see the following:

```php
$sql = "SELECT * FROM " . WP_CALENDAR_CATEGORIES_TABLE . " WHERE category_id=".mysql_escape_string($_GET['category_id']);
```

As you can clearly see, the developer chose to utilize the mysql_escape_string() function to escape $_GET['category_id] before using category_id to build a SQL statement. Looking at [the documentation](http://php.net/manual/en/function.mysql-escape-string.php) for `mysql_escape_string()`, we see that the specific characters escaped are: null byte (0), newline (\n), carriage return (\r), backslash (\), single quote ('), double quote (") and substitute (SUB, or \032). In this case, none of these characters are required in order for SQL injection to be successful. The user controlled $_GET['category_id'] is not enclosed in quotes, so there is no need to break out of quotes for SQL injection. For example, the attacker can pass the following:

```
http://path-to-server/calendar.php? category_id=1%20union%20select%201,2,3,4,5,6%20from%20users;
```

This would result in the following SQL statement:

```sql
SELECT * FROM WP_CALENDAR_CATEGORIES_TABLE WHERE category_id=1 union select 1,2,3,4,5,6 from users;
```

As you can see, the attacker can craft a valid SQL injection without using any of the characters escaped by mysql_escape_string(). The developers addressed this issue by casting the $_GET['category_id'] to an int before using it in a SQL statement.

If you look closely... you'll see other, unpatched SQL injections with the same symptom littered throughout the code...

## Developer's Solution

```php
...snip...
</style>
<?php
  // We do some checking to see what we're doing
  if (isset($_POST['mode']) && $_POST['mode'] == 'add')
    {
      // Proceed with the save
      $sql = "INSERT INTO " . WP_CALENDAR_CATEGORIES_TABLE . " SET category_name='".mysql_escape_string($_POST['category_name'])."', category_colour='".mysql_escape_string($_POST['category_colour'])."'";
      $wpdb->get_results($sql);
      echo "<div class=\"updated\"><p><strong>".__('Category added successfully','calendar')."</strong></p></div>";
    }
  else if (isset($_GET['mode']) && isset($_GET['category_id']) && $_GET['mode'] == 'delete')
    {
      $sql = "DELETE FROM " . WP_CALENDAR_CATEGORIES_TABLE . " WHERE category_id=".mysql_escape_string($_GET['category_id']);
      $wpdb->get_results($sql);
      $sql = "UPDATE " . WP_CALENDAR_TABLE . " SET event_category=1 WHERE event_category=".mysql_escape_string($_GET['category_id']);
      $wpdb->get_results($sql);
      echo "<div class=\"updated\"><p><strong>".__('Category deleted successfully','calendar')."</strong></p></div>";
    }
  else if (isset($_GET['mode']) && isset($_GET['category_id']) && $_GET['mode'] == 'edit' && !isset($_POST['mode']))
    {
      $sql = "SELECT * FROM " . WP_CALENDAR_CATEGORIES_TABLE . " WHERE category_id=".mysql_escape_string($_GET['category_id']);
      $cur_cat = $wpdb->get_row($sql);
      ?>
<div class="wrap">
   <h2><?php _e('Edit Category','calendar'); ?></h2>
    <form name="catform" id="catform" class="wrap" method="post" action="<?php echo bloginfo('wpurl'); ?>/wp-admin/admin.php?page=calendar-categories">
                <input type="hidden" name="mode" value="edit" />
                <input type="hidden" name="category_id" value="<?php echo stripslashes($cur_cat->category_id) ?>" />
                <div id="linkadvanceddiv" class="postbox">
                        <div style="float: left; width: 98%; clear: both;" class="inside">
        <table cellpadding="5" cellspacing="5">
                                <tr>
        <td><legend><?php _e('Category Name','calendar'); ?>:</legend></td>
                                <td><input type="text" name="category_name" class="input" size="30" maxlength="30" value="<?php echo stripslashes($cur_cat->category_name) ?>" /></td>
        </tr>
                                <tr>
        <td><legend><?php _e('Category Colour (Hex format)','calendar'); ?>:</legend></td>
                                <td><input type="text" name="category_colour" class="input" size="10" maxlength="7" value="<?php echo stripslashes($cur_cat->category_colour) ?>" /></td>
                                </tr>
                                </table>
                        </div>
                        <div style="clear:both; height:1px;">&nbsp;</div>
                </div>
                <input type="submit" name="save" class="button bold" value="<?php _e('Save','calendar'); ?> &raquo;" />
    </form>
</div>
      <?php
    }
  else if (isset($_POST['mode']) && isset($_POST['category_id']) && isset($_POST['category_name']) && isset($_POST['category_colour']) && $_POST['mode'] == 'edit')
    {
      // Proceed with the save
      $sql = "UPDATE " . WP_CALENDAR_CATEGORIES_TABLE . " SET category_name='".mysql_escape_string($_POST['category_name'])."', category_colour='".mysql_escape_string($_POST['category_colour'])."' WHERE category_id=".mysql_escape_string($_POST['category_id']);
      $wpdb->get_results($sql);
      echo "<div class=\"updated\"><p><strong>".__('Category edited successfully','calendar')."</strong></p></div>";
    }
  $get_mode = 0;
  $post_mode = 0;
  if (isset($_GET['mode'])) {
    if ($_GET['mode'] == 'edit') {
      $get_mode = 1;
    }
  }
  if (isset($_POST['mode'])) {
    if ($_POST['mode'] == 'edit') {
      $post_mode = 1;
    }
  }
  if ($get_mode != 1 || $post_mode == 1)
    {
?>
  <div class="wrap">
    <h2><?php _e('Add Category','calendar'); ?></h2>
    <form name="catform" id="catform" class="wrap" method="post" action="<?php echo bloginfo('wpurl'); ?>/wp-admin/admin.php?page=calendar-categories">
                <input type="hidden" name="mode" value="add" />
                <input type="hidden" name="category_id" value="">
                <div id="linkadvanceddiv" class="postbox">
                        <div style="float: left; width: 98%; clear: both;" class="inside">
              <table cellspacing="5" cellpadding="5">
                                <tr>
                                <td><legend><?php _e('Category Name','calendar'); ?>:</legend></td>
                                <td><input type="text" name="category_name" class="input" size="30" maxlength="30" value="" /></td>
                                </tr>
                                <tr>
                                <td><legend><?php _e('Category Colour (Hex format)','calendar'); ?>:</legend></td>
                                <td><input type="text" name="category_colour" class="input" size="10" maxlength="7" value="" /></td>
                                </tr>
                                </table>
                        </div>
            <div style="clear:both; height:1px;">&nbsp;</div>
                </div>
                <input type="submit" name="save" class="button bold" value="<?php _e('Save','calendar'); ?> &raquo;" />
    </form>
    <h2><?php _e('Manage Categories','calendar'); ?></h2>
```