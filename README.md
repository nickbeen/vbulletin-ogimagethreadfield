# Open Graph Image Threadfield

This product for vBulletin 3.8 adds support for saving and editing an Open Graph image meta tag for each thread. Four file edits in `includes/functions.php` are required to be able to use the data for the meta tag in the `headinclude` template.

Included are also instructions to use the preview text of the first post of the thread for an Open Graph description meta tag and an improved `meta description` tag since vBulletin uses a poorly constructed `meta description` tag by only using the thread title and current page number.

## Product upload

First upload `product-ogimagethreadfield.xml` in the Products & Plugins section.

## File edits

Find near line 1272:
```php
$threadcache["$threadid"] = $vbulletin->db->query_first("
  SELECT IF(visible = 2, 1, 0) AS isdeleted,
```

Replace with:
```php
$threadcache["$threadid"] = $vbulletin->db->query_first("
  SELECT IF(thread.visible = 2, 1, 0) AS isdeleted,
```

Find near line 1278:
```php
. iif($vbulletin->options['threadvoted'] AND $vbulletin->userinfo['userid'], 'threadrate.vote,')
. iif($marking, 'threadread.readtime AS threadread, forumread.readtime AS forumread,') . "
```

Replace with:
```php
. iif($vbulletin->options['threadvoted'] AND $vbulletin->userinfo['userid'], 'threadrate.vote,')
. iif($marking, 'threadread.readtime AS threadread, forumread.readtime AS forumread,')
. iif($vbulletin->options['threadpreview'] > 0 AND THIS_SCRIPT == 'showthread', 'post.pagetext AS preview, ') . "
```

Find near line 1293:
```php
$tachyjoin
$hook_query_joins
WHERE thread.threadid = $threadid
```

Replace with: 
```php
" . iif($vbulletin->options['threadpreview'] > 0 AND THIS_SCRIPT == 'showthread', "LEFT JOIN " . TABLE_PREFIX . "post AS post ON(post.postid = thread.firstpostid)") . "  
$tachyjoin
$hook_query_joins
WHERE thread.threadid = $threadid
```

Find near line 1307:
```php
$thread = $threadcache["$threadid"];
if ($thread['prefixid'])
{
```

Replace with:
```php
if($vbulletin->options['threadpreview'] > 0 AND THIS_SCRIPT == 'showthread') 
{ 
  $threadcache["$threadid"]['preview'] = strip_quotes($threadcache["$threadid"]['preview']);
  $threadcache["$threadid"]['preview'] = htmlspecialchars_uni(
    fetch_censored_text(
      fetch_trimmed_title(
        strip_bbcode($threadcache["$threadid"]['preview'], false, true),
        $vbulletin->options['threadpreview'])
      )
  );
}  

$thread = $threadcache["$threadid"];
if ($thread['prefixid'])
{
```

## Template edits

Go to the `headinclude` template in any of your styles.

Use the following code to add the Open Graph image meta tag to thread pages.

```html
<if condition="$show['threadinfo'] AND !empty($threadinfo[img])">
  <meta property="og:image" content="$threadinfo[img]">
</if>
```

Use the following code to add an Open Graph description meta tag to thread pages.
```html
<if condition="$show['threadinfo']">
  <meta property="og:description" content="$threadinfo[preview]">
</if>
````

Use the following code to use thread specific data in the `meta description` tag

```html
<if condition="$show['threadinfo']">
  <meta name="description" content="$threadinfo[preview]<if condition="$pagenumber > 1"> (<phrase 1="$pagenumber">$vbphrase[page_x]</phrase>)</if>">
</if>
```

## Result

If done properly, your thread pages will produce HTML metadata like this:

```html
<!doctype html>
<html prefix="og: http://ogp.me/ns#">
<head>
  <meta charset="utf-8">
  <meta name="description" content="Lorem ipsum dolor sit amet, consectetur adipiscing elit">
  <meta property="og:description" content="Lorem ipsum dolor sit amet, consectetur adipiscing elit">
  <meta property="og:image" content="https://www.myvbulletinforum.com/attachment.php?attachmentid=1">
  ...
```
