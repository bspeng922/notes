## 使用termainator替换ubuntu默认终端

### 安装
```shell
sudo apt-get install terminator
sudo yum install terminator
```


### 常用快捷键

> Ctrl+Shift+E    垂直分割窗口
> Ctrl+Shift+O    水平分割窗口
>     F11         全屏
> Ctrl+Shift+C    复制
> Ctrl+Shift+V    粘贴
> Ctrl+Shift+N    或者 Ctrl+Tab 在分割的各窗口之间切换
> Ctrl+Shift+X    将分割的某一个窗口放大至全屏使用
> Ctrl+Shift+Z    从放大至全屏的某一窗口回到多窗格界面


### 优化配置

安装后可配置让其尽量美化些,terminator配置文件在~/.config/terminator/config 可以通过这个配置文件配置terminator的字体和颜色.
```
font = Monaco 10  #设置体字
background_color = "#204070" # 背景颜色
foreground_color = "#F0F0F0" # 字体颜色
cursor_blink = True          # 设置光标
scrollbar_position = disabled # 禁用滚动条
titlebars = no # 禁用标题栏
background_darkness = 0.4
background_type = transparent # 背景类型可以设置为图片
```

更多配置见: man terminator_config


### 我的配置

```ini
[global_config]
  title_transmit_bg_color = "#d30102"
  focus = system
  suppress_multiple_term_dialog = True
[keybindings]
[profiles]
  [[default]]
    palette = "#073642:#dc322f:#859900:#b58900:#268bd2:#d33682:#2aa198:#eee8d5:#002b36:#cb4b16:#586e75:#657b83:#839496:#6c71c4:#93a1a1:#fdf6e3"
    copy_on_selection = True
    background_image = None
    background_darkness = 0.85
    background_type = transparent
    use_system_font = False
    cursor_color = "#eee8d5"
    foreground_color = "#839496"
    show_titlebar = False
    font = Ubuntu Mono 14
    background_color = "#002b36"
  [[solarized-dark]]
    palette = "#073642:#dc322f:#859900:#b58900:#268bd2:#d33682:#2aa198:#eee8d5:#002b36:#cb4b16:#586e75:#657b83:#839496:#6c71c4:#93a1a1:#fdf6e3"
    background_color = "#002b36"
    background_image = None
    cursor_color = "#eee8d5"
    foreground_color = "#839496"
  [[solarized-light]]
    palette = "#073642:#dc322f:#859900:#b58900:#268bd2:#d33682:#2aa198:#eee8d5:#002b36:#cb4b16:#586e75:#657b83:#839496:#6c71c4:#93a1a1:#fdf6e3"
    background_color = "#fdf6e3"
    background_image = None
    cursor_color = "#002b36"
    foreground_color = "#657b83"
[layouts]
  [[default]]
    [[[child1]]]
      type = Terminal
      parent = window0
      profile = default
    [[[window0]]]
      type = Window
      parent = ""
[plugins]
```


**命令行颜色方案**
[http://ethanschoonover.com/solarized](http://ethanschoonover.com/solarized)
[https://github.com/seebi/dircolors-solarized](https://github.com/seebi/dircolors-solarized)


**快捷键** 
```
KEYBINDINGS
The following keybindings can be used to control Terminator:
Ctrl+Shift+O
Split terminals Horizontally.（上下开新窗口）
Ctrl+Shift+E
Split terminals Vertically.（垂直开新窗口）
Ctrl+Shift+Right
Move parent dragbar Right.（放大当前窗口 向右）
Ctrl+Shift+Left
Move parent dragbar Left.
Ctrl+Shift+Up
Move parent dragbar Up.
Ctrl+Shift+Down
Move parent dragbar Down.
Ctrl+Shift+S
Hide/Show Scrollbar.（隐藏滚动条）
Ctrl+Shift+F
Search within terminal scrollback
Ctrl+Shift+N or Ctrl+Tab
Move to next terminal within the same tab, use Ctrl+PageDown to move to the next tab. If cycle_term_tab is False, cycle within the same tab will be disabled
Ctrl+Shift+P or Ctrl+Shift+Tab
Move to previous terminal within the same tab, use Ctrl+PageUp to move to the previous tab. If cycle_term_tab is False, cycle within the same tab will be disabled
Alt+Up
Move to the terminal above the current one.（切换当前窗口）
Alt+Down
Move to the terminal below the current one.
Alt+Left
Move to the terminal left of the current one.
Alt+Right
Move to the terminal right of the current one.
Ctrl+Shift+C
Copy selected text to clipboard
Ctrl+Shift+V
Paste clipboard text
Ctrl+Shift+W
Close the current terminal.
Ctrl+Shift+Q
Quits Terminator
Ctrl+Shift+X （最大化当前窗口）
Toggle between showing all terminals and only showing the current one (maximise).
Ctrl+Shift+Z
Toggle between showing all terminals and only showing a scaled version of the current one (zoom).
Ctrl+Shift+T
Open new tab
Ctrl+Shift+Alt+T
Open new tab at root level, if using extreme_tabs.
Ctrl+PageDown
Move to next Tab
Ctrl+PageUp
Move to previous Tab
Ctrl+Shift+PageDown
Swap tab position with next Tab
Ctrl+Shift+PageUp
Swap tab position with previous Tab
Ctrl+Shift+F
Open buffer search bar to find substrings in the scrollback buffer. Hit Escape to cancel.
Ctrl+Plus (+)
Increase font size. Note: this may require you to press shift, depending on your keyboard
Ctrl+Minus (-)
Decrease font size. Note: this may require you to press shift, depending on your keyboard
Ctrl+Zero (0)
Restore font size to original setting.
F11
Toggle fullscreen（放大当前窗口）
Ctrl+Shift+R
Reset terminal state
Ctrl+Shift+G
Reset terminal state and clear window
Drag and Drop
The layout can be modified by moving terminals with Drag and Drop. To start dragging a terminal, hold down Ctrl, click and hold the right mouse button. Then, **Release Ctrl**. You can now drag the terminal do the point in the layout you would like it to be. The zone where the terminal would be inserted will be highlighted. 
```
