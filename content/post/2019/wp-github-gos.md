---
title: "wordpress 插件 – 使用 Github 作为免费图床"
date: 2019-07-23T21:36:03+08:00
draft: false
categories:
    - WordPress
tags:
    - github
    - 图床
coverColor: "#2574abf7"
coverImage: https://static.acme.top/wp-content/uploads/2019/07/wp_github_gos.png
---

## 前言

插件名称 `wp-github-gos`， 本插件核心功能使用了 GitHub API

设置页面和核心业务逻辑主要参考插件 [wordpress-qcloud-cos](https://github.com/sy-records/wordpress-qcloud-cos) 的实现，替换了其中 `腾讯云 COS 官方 SDK` 为 `GitHub API`

## 插件特色

- 使用 GitHub 仓库存储 WordPress 站点图片等多媒体文件

- 可配置是否上传缩略图和是否保留本地备份

- 本地删除可同步删除腾讯云上面的文件

- 支持替换数据库中旧的资源链接地址

- 支持在图片链接地址后面自定义拼接图片`宽度`、`高度`、`大小`三个参数

- 建议使用 jsdelivr 免费加速 github 下的图片，速度还是很可以的

## 插件缺点

- 使用 Github API 同步图片等附件的时候速度相较于国内的免费图床比较慢 😅

- 未来不知道会不会被屏蔽 🙂

## 安装

从 Github 下载源码，通过 WordPress 后台上传安装，或者直接将源码上传到 WordPress 插件目录 `wp-content/plugins`，然后在后台启用

Github 项目地址: [https://github.com/niqingyang/wp-github-gos](https://github.com/niqingyang/wp-github-gos "https://github.com/niqingyang/wp-github-gos")

修改配置

方法一：在 WordPress 插件管理页面有设置按钮，进行设置

方法二：在 WordPress 后台管理左侧导航栏设置下 `Github 存储`，点击进入设置页面

特别说明

本插件仅支持PHP 5.4+ 版本

## 设置

因为本插件使用了 Github API，所以需要使用者进行一下操作：

1. 注册一个 Github 账户
2. 并创建一个用于存放附件的公开的仓库
3. 创建一个 access token

![点击 settings 菜单](https://static.acme.top/wp-content/uploads/2019/07/paste-3fcd46b9dd1495dae823a40edfa37fbb-1.png?w=212&h=452)

![进入 Developer Settings 页面创建 Token](https://static.acme.top/wp-content/uploads/2019/07/paste-05312293588326deb1dbad23b28f41de-1.png?w=1012&h=385)

![勾选必要的权限](https://static.acme.top/wp-content/uploads/2019/07/paste-905e412f7c8cd8018957961c14f9c315-1.png?w=737&h=149)

4. 将 `Github 账户名`、`仓库名`、`access token` 粘帖到该插件设置页面中相应的位置

![填写相关设置](https://static.acme.top/wp-content/uploads/2019/07/paste-7fa86ef9fd73858f6491d23919e5ac3c-1.png?w=614&h=315)

5. URL前缀的格式为 `https://raw.githubusercontent.com/{用户名}/{仓库名}/master/`，或者 `https://raw.githubusercontent.com/{用户名}/{仓库名}/master/{本地文件夹}`，“本地文件夹” 务必与上面设置的 “本地文件夹” 保持一致（结尾无/）

6. 设置自己的域名：可以使用 `Github Page 的域名`，也可以用 CNAME 将自己的域名解析到 `Github Page 的域名` 上, 然后将`https://raw.githubusercontent.com/{用户名}/{仓库名}/master/` 替换成自己的域名就可以了

7. 💥💥 国内访问 github 速度不是很理想，可以使用 jsdelivr 免费加速 github，只需要将 URL 前缀设置格式为 `https://cdn.jsdelivr.net/gh/{用户名}/{仓库名}/`即可

## 插件预览

![设置页预览](https://static.acme.top/wp-content/uploads/2019/07/paste-ac1ecbb94dfeb65dbe5fd75f49171446-1.png?w=1706&h=1266)

## 常见问题

1、怎么替换文章中之前的旧资源地址链接

这个插件已经加上了替换数据库中之前的旧资源地址链接功能，只需要填好对应的链接即可，如图所示

![](https://static.acme.top/wp-content/uploads/2019/07/paste-be19add5c14b2d259e53a8d1b67ee8e6-1.png?w=743&h=236)

## WP-Editor.md

本站使用的 Markdown 编辑器是 [WP-Editor.md](https://github.com/JaxsonWang/WP-Editor.md "Editor.md")，非常好用，此处推荐一下，如果你也使用的是这个插件，那么可以通过在主题模板下的 `functions.php` 文件中加入下面的代码，即可实现在编辑器中粘帖截图上传的功能：

```php
/**
 * 文章编辑页面从媒体选择器中插入图片
 *
 * @param string $html
 * @param integer $id
 * @param string $caption
 * @param string $title
 * @param string $align
 * @param string $url
 * @param string $size
 * @param string $alt
 * @return mixed
 */
function acme_image_send_to_editor ($html, $id, $caption, $title, $align, $url, $size, $alt)
{
    $attrs = wp_kses_hair($html, wp_allowed_protocols());

    $attrs = array_column($attrs, 'value', 'name');

    $src = $url . '?w=' . $attrs['width'] . '&h=' . $attrs['height'] . '&id=' . $id;

    $html = str_replace($url, $src, $html);

    return $html;
}

add_filter('image_send_to_editor', 'acme_image_send_to_editor', 10, 8);

/**
 * 粘帖图片上传，替换EditorMD插件的粘帖上传
 */
function acme_imagepaste_action_callback ()
{
    $result = array(
        'error' => ''
    );
    $upload = wp_upload_dir();
    $uploadUrl = $upload['url'];
    $uploadDir = $upload['path'];
    $extension = '';
    list($data, $image) = explode(';', $_REQUEST['dataurl']);
    list($field, $type) = explode(':', $data);
    list($encoding, $content) = explode(',', $image);
    if($type == 'image/png')
    {
        $extension = 'png';
    }
    $name = md5($_REQUEST['dataurl']);
    if(! $extension)
    {
        $result['error'] = "Could not determine image extension type";
    }
    else
    {
        $file = $uploadDir . '/paste-' . $name . '.' . $extension;
        file_put_contents($file, base64_decode($content));
        // 获取图片尺寸
        $size = @getimagesize($file);

        if($size == false)
        {
            $result['error'] = "Could not get image size";
        }
        else
        {
            $_FILES['file'] = [
                'name' => '/paste-' . $name . '.' . $extension,
                'type' => 'image/' . $extension,
                'tmp_name' => $file,
                'error' => 0,
                'size' => filesize($file)
            ];

            $time = current_time('mysql');

            $overrides = array(
                'test_form' => false,
                'action' => 'acme_imagepaste_action'
            );

            $file = wp_handle_upload($_FILES['file'], $overrides, $time);

            if(isset($file['error']))
            {
                $result['error'] = $file['error'];
            }
            else
            {
                if(strpos($file['url'], '?'))
                {
                    $url = $file['url'];
                }
                else
                {
                    $url = $file['url'] . '?w=' . $size[0] . '&h=' . $size[1];
                }

                $result['w3tc'] = 0;
                $result['url'] = $url;
                $result['elementid'] = $_REQUEST['elementid'];
            }
        }
    }

    echo json_encode($result);
    die();
}

add_action('wp_ajax_imagepaste_action', 'acme_imagepaste_action_callback', 1);
```

## 更新记录

**V1.0 - 2019/07/23**

- 使用 GitHub 仓库存储 WordPress 站点图片等多媒体文件
- 可配置是否上传缩略图和是否保留本地备份
- 本地删除可同步删除腾讯云上面的文件
- 支持替换数据库中旧的资源链接地址
- 支持在图片链接地址后面自定义拼接图片宽度、高度、大小三个参数

**V1.1 - 2020/06/28**

- 根据 Github API 的变更，将 access_token 参数淘汰，请求时将其放入请求头中
- 优化上传，偶发性的会发生 cURL error 35 错误，这种错误会重新尝试最多三次上传
- 优化上传，发生未知的错误会将其捕获并反馈到前端显示上传失败
- 为便于调试上传，支持将上传及错误信息记录日志，默认是关闭的

## 最后

感谢 Github 为开发者提供的服务，也希望大家能够善待 Github，毕竟存储和访问都是在消耗 Github 白花花的银子，切勿恶意滥用 ~