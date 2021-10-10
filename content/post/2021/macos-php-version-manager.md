---

title: "Mac OS PHP 版本管理"
date: 2021-10-10T15:24:17+08:00
draft: false
## 格式：aside、chat、status、quote、link、gallery、image、audio、video，为空则代表标准格式
# format: 
## 主题色
# themeColor: 
## 封面颜色
coverColor: "rgba(119,70,27,0.97)"
## 封面图片
coverImage: https://repository-images.githubusercontent.com/229187949/f140f880-4c25-11eb-8105-aefec9dc7c66
## 文章页封面图分割：0-默认小图模式 1-大图模式
# coverStyle: 0
---

## 准备

- 安装 homebrew

    ```bash
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
    ```

- 更新到最新

    ```bash
    brew update
    ```

- 搜索 PHP

    ```bas
    brew search php
    ```

- 查看本机安装的情况

    ```bash
    brew list
    ```

## 安装 PHP

- 安装 `PHP7.4`

    ```bash
    brew install shivammathur/php/php@7.4
    ```

    安装后会提示

    ```bash
    If you need to have php@7.4 first in your PATH, run:
      echo 'export PATH="/usr/local/opt/php@7.4/bin:$PATH"' >> ~/.zshrc
      echo 'export PATH="/usr/local/opt/php@7.4/sbin:$PATH"' >> ~/.zshrc
    
    For compilers to find php@7.4 you may need to set:
      export LDFLAGS="-L/usr/local/opt/php@7.4/lib"
      export CPPFLAGS="-I/usr/local/opt/php@7.4/include"
    ```

    

- 安装后，链接它

    ```bash
    brew link --overwrite --force php@7.4
    ```

- 重新启动终端并测试您的 PHP 版本：

    ```bash
    php -v
    ```

<warning>

> **常见错误**
>
> 如果遇到类似错误：`dyld: Library not loaded: /usr/local/opt/tidy-html5/lib/libtidy.58.dylib`，则可能是你的 `tidy-html5` 版本太低了，可以尝试执行 `brew reinstall tidy-html5` 重新安装

</warning>

## 升级 PHP 版本

可以将 PHP 版本升级到最新的修补程序版本。

```bash
brew upgrade shivammathur/php/php@7.4
```

## PHP 多版本之间切换

- 切换到 `PHP7.4`

    ```bash
    brew link --overwrite --force php@7.4
    ```

- 如果您得到如下警告，请按建议操作：

    ```bash
    Warning: Already linked: <Cellar Path>
    To relink:
      brew unlink <formula> && brew link <formula>
    ```

    ```bash
    brew unlink php@7.4
    brew link --overwrite --force php@7.4
    ```

    <info>

    > 如果切换失败，可以尝试编辑 `~/.zshrc` 中的 PHP 版本直接修改环境变量

    </info>

## 参考

- [shivammathur/homebrew-php: Homebrew tap for PHP 5.6 to 8.2. PHP 8.1 and 8.2 are nightly builds. (github.com)](https://github.com/shivammathur/homebrew-php)

