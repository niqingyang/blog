---
title: "为 Editor.md 编辑器插件增加预览和发布按钮"
date: 2019-06-23T17:19:50+08:00
draft: false
categories:
    - WordPress
tags:
    - blog
    - feature
themeColor: "#2196f3"
coverColor: rgba(33, 150, 243, 0.91)
coverImage: https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410224611-wp-editor.md_.png
---

## 前言

一直在使用 Editor.md 插件作为博客的编辑器，用着挺好，但是在全屏下编辑时，每次想预览或者保存又必须切换到非全屏状态下才可以点击按钮，用着不舒服，所以花了一点时间在工具栏上增加了预览、保存、发布三个按钮，在此记录一下修改的过程。

## 开发

根据 Editor.md 设置中的 "编辑器设置 > 编辑器静态资源地址" 找到静态资源文件，我的博客是放在了 wordpress 根目录下的 `assets` 文件夹下，修改文件 `/assets/Config/editormd.js`，增加内容如下：

```diff
var toolBar;
switch (textareaID) {
    case 'wp-content-editor-container' :
        toolBar = fullToolBar;
        break;
    case 'comment' :
        toolBar = simpleToolBar;
        break;
    case 'wp-replycontent-editor-container' :
        toolBar = miniToolBar;
        break;
}

+var postSaveText = $("#publish").val();
+var toolbarHandlers = {};

+if($("body").hasClass("wp-admin") && ($("body").hasClass("post-type-post") || $("body").hasClass("post-type-page")){
+    // 预览更改
+    toolBar.push('||', 'postPreview');
+    // 保存草稿
+    if($("#save-post").size() == 1){
+        toolBar.push('postSaveDraft');
+    }
+    // 发布/更新
+    toolBar.push('postSave');
+
+    toolbarHandlers = {
+        /**
+         * @param {Object}	cm			CodeMirror对象
+         * @param {Object}	icon		图标按钮jQuery元素对象
+         * @param {Object}	cursor		CodeMirror的光标对象，可获取光标所在行和位置
+         * @param {String}	selection	编辑器选中的文本
+         */
+        postPreview: function(cm, icon, cursor, selection){
+            $("#post-preview").click();
+        },
+        postSaveDraft: function(cm, icon, cursor, selection){
+            $("#save-post").click();
+        },
+        postSave: function(cm, icon, cursor, selection){
+            if($("#publish").attr("name") == "save"){
+                $("#publish").click();
+            }else if(confirm("您确定要" + postSaveText + "文章吗？")){
+                $("#publish").click();
+            }
+        },
+    };
+}

var wpEditormd = editormd({
    id: textareaID,
    path: editor.editormdUrl + '/assets/Editormd/lib/',
    width: '100%', //编辑器宽度
    height: textareaID === 'wp-content-editor-container' ? 640 : 320,    //编辑器高度
    syncScrolling: editor.syncScrolling !== 'off', //即是否开启同步滚动预览
	// ........
	// 其他代码
	// .......
    onload: function () {
        //加载完成执行
        if ( textareaID === 'comment' ) {
            //修改评论表单name
            $('textarea.editormd-markdown-textarea').attr('name', 'comment');
        }

        if ( textareaID === 'wp-replycontent-editor-container' ) {
            $('.reply').click(function () {
                setTimeout(function () {
                    $('.edit-comments-php .CodeMirror.cm-s-default.CodeMirror-wrap').css('margin-top',$('.editormd-toolbar').height());
                },100);
            })
        }

        if (getWidth() === 1600) {
            // 1600分辨率删除编辑器编辑空白外边距
            var codeMirror = $('.editormd .CodeMirror.CodeMirror-wrap');
            var codeMirrorMarginTop = codeMirror.css('margin-top');
            codeMirror.css('margin-top',parseInt(codeMirrorMarginTop) - 32 + "px");
        }
    },
+    toolbarIconsClass: {
+        // 指定一个FontAawsome的图标类
+        postPreview: "fa-chrome",
+        postSaveDraft: "fa-floppy-o",
+        postSave: "fa-paper-plane",
+    },
+    lang: {
+        toolbar: {
+            postPreview: "预览更改",
+            postSaveDraft: "保存草稿",
+            postSave: postSaveText,
+        }
+    },
+    // 自定义工具栏按钮的事件处理
+    toolbarHandlers: toolbarHandlers
});
```

## 压缩

将编写好后的代码压缩，然后替换掉同级目录下的 `editormd.min.js`，就大功告成了！

## 效果

部署好后，工具栏右上角就会多出三个按钮，然后就可以愉快的在全屏下写文章了~

![](https://cdn.jsdelivr.net/gh/niqingyang/blog-static@main/images/2021/04/20210410205436-paste-e6e62536cd9cd221d6f1d4cfbf602b83-1.png)

## 参考资料

[editor.md 官网](https://pandao.github.io/editor.md/ "editor.md 官网")

[editor.md 自定义工具栏](https://pandao.github.io/editor.md/examples/custom-toolbar.html "editor.md 自定义工具栏")