---
title: "禁用 WordPress 的 Emoji 功能"
date: 2019-07-15T19:31:55+08:00
draft: false
categories:
    - WordPress
tags:
    - emoji
coverColor: "#f2c800"
coverColor: "#f4cc04f7"
coverImage: https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410224644-wordpress-remove-emoji.png
---

wordpress 自 4.2 起自带了 emoji 功能，默认使用 http://s.w.org/images/core/emoji/72x72/ 下的图片来渲染 emoji，可以使用下面的方式替换为 twitter 扁平化风格的 emoji

```php
// 替换 WordPress 默认 Emoji 资源地址
function acme_change_wp_emoji_baseurl ($url)
{
	return set_url_scheme('//twemoji.maxcdn.com/2/72x72/');
}
add_filter('emoji_url', 'acme_change_wp_emoji_baseurl');

// WordPress 4.6 新增 SVG 格式资源
function acme_change_wp_emoji_svgurl ($url)
{
	return set_url_scheme('//twemoji.maxcdn.com/svg/');
}
add_filter('emoji_svg_url', 'acme_change_wp_emoji_svgurl');
```

这种替换为图片的方式一直影响着我的 markdown 编译器，一旦插入 emoji 编辑器就无法获取焦点，而且随着 win10 的普及，系统自带的 emoji 也很漂亮，所以觉得有必要禁掉 wordpress 的 emoji 功能了，只需要在主题文件 `functions.php` 中加入如下代码即可：

```php
remove_action('admin_print_scripts', 'print_emoji_detection_script');
remove_action('admin_print_styles', 'print_emoji_styles');
remove_action('wp_head', 'print_emoji_detection_script', 7);
remove_action('wp_print_styles', 'print_emoji_styles');
remove_action('embed_head', 'print_emoji_detection_script');
remove_filter('the_content_feed', 'wp_staticize_emoji');
remove_filter('comment_text_rss', 'wp_staticize_emoji');
remove_filter('wp_mail', 'wp_staticize_emoji_for_email');
```