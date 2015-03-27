---
layout: post
title: sublime安装
description: "sublime"
modified: 2015-03-22
category: articles
tags: [sample]
image:
  feature: texture-feature-02.jpg
---

###installation
1.  安装key
    ```
    —– BEGIN LICENSE —–
    Andrew Weber
    Single User License
    EA7E-855605
    813A03DD 5E4AD9E6 6C0EEB94 BC99798F
    942194A6 02396E98 E62C9979 4BB979FE
    91424C9D A45400BF F6747D88 2FB88078
    90F5CC94 1CDC92DC 8457107A F151657B
    1D22E383 A997F016 42397640 33F41CFC
    E1D0AE85 A0BBD039 0E9C8D55 E1B89D5D
    5CDB7036 E56DE1C0 EFCC0840 650CD3A6
    B98FC99C 8FAC73EE D2B95564 DF450523
    —— END LICENSE ——
    ```
2.  命令行工具
    ```shell
    ln -s /Applications/Sublime\ Text.app/Contents/SharedSupport/bin/subl /usr/local/bin/
    ```

###安装Package Control
打开组合键Ctrl+\`，出现控制台，输入下面内容，安装[sublime-package](https://packagecontrol.io/installation)
```python
import urllib.request,os,hashlib; h = 'eb2297e1a458f27d836c04bb0cbaf282' + 'd0e7a3098092775ccb37ca9d6b2e4b7d'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

###Markdown
安装Markdown Preview

1.	打开组合键`cmd+shift+p`
2.	输入`install`，等待显示package列表后输入`markdown preview`，选择安装
3.	绑定快捷键，`Preference` > `Key Bindings - User`，输入
    ```json
    { "keys": ["alt+m"], "command": "markdown_preview", "args": {"target": "browser", "parser":"markdown"} }
    ```
4.	设置preview语法高亮和mathjax支持，在`Preferences` > `Package Settings` > `Markdown Preview` > `Setting Default`下输入
    ```json
    {
      "enable_mathjax": true,
      "enable_highlight": true,
    }
    ```

###Ruby
1.  通过Package install安装soda theme和railcast color theme
2.  修改`Perference` > `Settings User`
```json
{
  "auto_complete_commit_on_tab": true,
  "caret_style": "solid",
  "color_scheme": "Packages/Color Scheme - Default/Cobalt.tmTheme",
  "draw_white_space": "all",
  "enable_tab_scrolling": false,
  "ensure_newline_at_eof_on_save": true,
  "file_exclude_patterns":
  [
    "*.pyc",
    "*.pyo",
    "*.exe",
    "*.dll",
    "*.obj",
    "*.o",
    "*.a",
    "*.lib",
    "*.so",
    "*.dylib",
    "*.ncb",
    "*.sdf",
    "*.suo",
    "*.pdb",
    "*.idb",
    "*.class",
    "*.psd",
    "*.db",
    "*.beam",
    ".DS_Store",
    ".tags"
  ],
  "folder_exclude_patterns":
  [
    ".svn",
    ".git",
    ".hg",
    "CVS",
    ".sass-cache"
  ],
  "font_face": "Menlo",
  "font_options":
  [
    "no_round"
  ],
  "font_size": 13,
  "highlight_line": true,
  "highlight_modified_tabs": true,
  "ignored_packages":
  [
    "Vintage"
  ],
  "rulers":
  [
    80,
    120
  ],
  "scroll_past_end": false,
  "tab_size": 2,
  "theme": "Soda Light 3.sublime-theme",
  "translate_tabs_to_spaces": true,
  "trim_trailing_white_space_on_save": true,
  "wide_caret": true,
  "word_separators": "./\\()\"'-:,.;<>~@#$%^&*|+=[]{}`~",
  "word_wrap": false
}
```
3.  安装其他Packages

    - All Autocomplete 自动补全
    - SideBarEnhancements 文件菜单增强
    - GitGutter 需修改`GitGutter` > `Settings-User`
      ```json
      "git_binary": "/usr/local/bin/git"
      ```
    - Alignment ⌘ + Ctrl + A 基于等号对齐
    - emmet(Zen Coding)
    - Ctags
      ```shell
      brew install ctags
      ```
      在项目中新建一个ctags_for_ruby的文件并运行，代码如下
      ```ruby
      #!/usr/bin/env ruby
      system "find . -name '*.rb' | ctags -f .tags -L -"

      if File.exist? './Gemfile'
        require 'bundler'
        paths = Bundler.load.specs.map(&:full_gem_path).join(' ')
        system "ctags -R -f .gemtags #{paths}"
      end
      ```
      修改ctags的配置文件
      ```json
      {
        "command": "ctags_for_ruby"
      }
      ```

      使用 ^ + T + R生成tag文件，使用^ + T + T/^ + T + Y

    - pretty json 绑定快捷键是 ⌘ + ^ + J
    - BracketHighlighter
    - Better CoffeeScript
    - Slim
    - SCSS
