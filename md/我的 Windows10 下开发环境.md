> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [gadzan.com](https://gadzan.com/my-windows10-development-environment)

> 安装新版 PowerShell PowerShell 官网 [https://aka.ms/powershell] 在 Windows 上安装 PowerShell [https://docs.mic......

安装新版 PowerShell
---------------

[PowerShell 官网](https://aka.ms/powershell)

[在 Windows 上安装 PowerShell](https://docs.microsoft.com/zh-cn/powershell/scripting/install/installing-powershell-core-on-windows?view=powershell-7)

GitHub[“发布”](https://github.com/PowerShell/PowerShell/releases) 页下载 msi 安装包, 安装后即可.

### 美化 PowerShell

打开 PowerShell 并输入以下命令安装 posh-git 和 oh-my-posh 这两个模块  
oh-my-posh 的文档在[这里](https://ohmyposh.dev/docs/pwsh)

```
Install-Module posh-git -Scope CurrentUser 
Install-Module oh-my-posh -Scope CurrentUser


```

安装完成后可以使用下面命令查看可用主题.

```
Get-PoshThemes


```

优化`ls`命令的颜色，需要管理员权限运行

```
Install-Module Get-ChildItemColor


```

### 让 PowerShell 主题配置生效

新增（/ 或修改）你的 PowerShell 配置文件  
如果之前没有配置文件，就新建一个 PowerShell 配置文件

```
if (!(Test-Path -Path $PROFILE )) { New-Item -Type File -Path $PROFILE -Force }


```

用记事本打开配置文件

```
notepad $PROFILE


```

在其中添加下面的内容

```
Import-Module Get-ChildItemColor

$env:PYTHONIOENCODING="utf-8"

If (Test-Path Alias:curl) {Remove-Item Alias:curl}
If (Test-Path Alias:curl) {Remove-Item Alias:curl}

Set-Alias l Get-ChildItemColor -option AllScope
Set-Alias ls Get-ChildItemColorFormatWide -option AllScope

Import-Module posh-git 
Import-Module oh-my-posh 
Set-PoshPrompt -Theme jandedobbeleer


```

安装 choco
--------

```
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))


```

### 安装 ConEmu

```
choco install ConEmu


```

安装 Scoop
--------

在 PowerShell 中输入下面内容，保证允许本地脚本的执行：  
`set-executionpolicy remotesigned -scope currentuser`

然后执行下面的命令安装 Scoop：  
`iex (new-object net.webclient).downloadstring('https://get.scoop.sh')`

静待脚本执行完成就可以了，安装成功后，执行下面命令查看帮助文档：  
`scoop help`

添加加 Nerd font 的仓库  
`scoop bucket add nerd-fonts`  
这里有 scoop 社区支持添加的[仓库列表](https://github.com/lukesampson/scoop/blob/master/buckets.json)

如果没有 git，会提示你顺便先安装 git，按照提示安装就行。

安装字体

```
scoop install FantasqueSansMono-NF


```

安装 Windows terminal
-------------------

首先安装 Windows terminal ，在 Microsoft store 里搜索 Windows terminal, 现在还是 preview 的版本

### 修改 Windows Terminal 配置

点击 Windows Terminal 顶栏的下拉箭头按钮，选择`Settings`, 按需要修改成下面配置。

```
"alt" while clicking on the "Settings" button.


{
    "$schema": "https://aka.ms/terminal-profiles-schema",
    
    "defaultProfile": "{574e775e-4f2a-5b96-ac1e-a2962a402336}",

    "profiles": {
        "defaults": {
            
            "fontFace" : "FantasqueSansMono NF",
            "fontSize" : 11,
            "colorScheme": "One Half Dark"
        },
    "list": [
        {
                
                "guid": "{574e775e-4f2a-5b96-ac1e-a2962a402336}",
                "hidden": false,
                "name": "PowerShell",
                "source": "Windows.Terminal.PowershellCore",
                "icon" : "ms-appx:///ProfileIcons/{574e775e-4f2a-5b96-ac1e-a2962a402336}.png",
                
                "useAcrylic" : true,
                "acrylicOpacity" : 0.3,
                "background" : "#171717",
                
                "backgroundImage" : "ms-appdata:///roaming/lightm.jpg",
                "backgroundImageOpacity" : 0.1,
                "closeOnExit" : true,
                "cursorColor" : "#FFFFFF",
                "cursorShape" : "vintage",
                "fontFace" : "FantasqueSansMono NF",
                "fontSize" : 11,
                "historySize" : 9001,
                "padding" : "0, 0, 0, 0",
                "snapOnInput" : true,
                "startingDirectory" : "%USERPROFILE%",
                
                "experimental.retroTerminalEffect": true
            },
            {
                
                "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
                "name": "Windows PowerShell",
                "commandline": "powershell.exe",
                "hidden": false,
                "acrylicOpacity" : 0.5,
                "background" : "#000",
                "closeOnExit" : true,
                "cursorColor" : "#FFFFFF",
                "cursorShape" : "bar",
                "fontFace" : "FantasqueSansMono NF",
                "fontSize" : 10,
                "padding" : "0, 0, 0, 0",
                "snapOnInput" : true,
                "useAcrylic" : true
            },
            {
                
                "guid": "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
                "name": "cmd",
                "commandline": "cmd.exe",
                "hidden": false,
                "acrylicOpacity" : 0.5,
                "background" : "#000",
                "closeOnExit" : true,
                "cursorColor" : "#FFFFFF",
                "cursorShape" : "bar",
                "fontFace" : "FantasqueSansMono NF",
                "fontSize" : 10,
                "padding" : "0, 0, 0, 0",
                "snapOnInput" : true,
                "useAcrylic" : true
            },
            {
                "guid": "{b453ae62-4e3d-5e58-b989-0a998ec441b8}",
                "hidden": false,
                "name": "Azure Cloud Shell",
                "source": "Windows.Terminal.Azure"
            },
            
            {
                "guid" : "{07b52e3e-de2c-5db4-bd2d-ba144ed6c273}",
                "acrylicOpacity" : 0.6,
                "background" : "#171717",
                "closeOnExit" : true,
                "hidden": false,
                "source": "Windows.Terminal.Wsl",
                "cursorColor" : "#FFFFFF",
                "cursorShape" : "vintage",
                "fontFace" : "FantasqueSansMono NF",
                "fontSize" : 11,
                "historySize" : 9001,
                "icon" : "ms-appx:///ProfileIcons/{07b52e3e-de2c-5db4-bd2d-ba144ed6c273}.png",
                "backgroundImage" : "ms-appdata:///roaming/lightm.jpg",
                "backgroundImageOpacity" : 0.3,
                "name" : "Ubuntu-20.04",
                "padding" : "0, 0, 0, 0",
                "snapOnInput" : true,
                "//wsl$/DistroName/home/username"。注意修改其中对应wsl发行版的名字以及你的用户名。
                "startingDirectory" : "//wsl$/Ubuntu-20.04/home/gadzan",
                "useAcrylic": true
            }
        ]
    },

    
    "schemes": [
           {
             "name": "Campbell",
             "foreground": "#F2F2F2",
             "background": "#0C0C0C",
             "colors": [
               "#0C0C0C",
               "#C50F1F",
               "#13A10E",
               "#C19C00",
               "#0037DA",
               "#881798",
               "#3A96DD",
               "#CCCCCC",
               "#767676",
               "#E74856",
               "#16C60C",
               "#F9F1A5",
               "#3B78FF",
               "#B4009E",
               "#61D6D6",
               "#F2F2F2"
             ]
           },
           {
             "name": "Solarized Dark",
             "foreground": "#FDF6E3",
             "background": "#073642",
             "colors": [
               "#073642",
               "#D30102",
               "#859900",
               "#B58900",
               "#268BD2",
               "#D33682",
               "#2AA198",
               "#EEE8D5",
               "#002B36",
               "#CB4B16",
               "#586E75",
               "#657B83",
               "#839496",
               "#6C71C4",
               "#93A1A1",
               "#FDF6E3"
             ]
           },
           {
             "name": "Solarized Light",
             "foreground": "#073642",
             "background": "#FDF6E3",
             "colors": [
               "#073642",
               "#D30102",
               "#859900",
               "#B58900",
               "#268BD2",
               "#D33682",
               "#2AA198",
               "#EEE8D5",
               "#002B36",
               "#CB4B16",
               "#586E75",
               "#657B83",
               "#839496",
               "#6C71C4",
               "#93A1A1",
               "#FDF6E3"
             ]
           },
           {
             "name": "Ubuntu",
             "foreground": "#EEEEEC",
             "background": "#2C001E",
             "colors": [
               "#EEEEEC",
               "#16C60C",
               "#729FCF",
               "#B58900",
               "#268BD2",
               "#D33682",
               "#2AA198",
               "#EEE8D5",
               "#002B36",
               "#CB4B16",
               "#586E75",
               "#657B83",
               "#839496",
               "#6C71C4",
               "#93A1A1",
               "#FDF6E3"
             ]
           },
           {
             "name": "UbuntuLegit",
             "foreground": "#EEEEEE",
             "background": "#2C001E",
             "colors": [
               "#4E9A06",
               "#CC0000",
               "#300A24",
               "#C4A000",
               "#3465A4",
               "#75507B",
               "#06989A",
               "#D3D7CF",
               "#555753",
               "#EF2929",
               "#8AE234",
               "#FCE94F",
               "#729FCF",
               "#AD7FA8",
               "#34E2E2",
               "#EEEEEE"
             ]
           },
           {
            "name": "Nord",
            "background": "#2e3440",
            "foreground": "#eceff4",
            "brightBlack": "#2e3440",
            "brightBlue": "#5e81ac",
            "brightCyan": "#8fbcbb",
            "brightGreen": "#a3be8c",
            "brightPurple": "#b48ead",
            "brightRed": "#bf616a",
            "brightWhite": "#eceff4",
            "brightYellow": "#ebcb8b",
            "black": "#2e3440",
            "blue": "#5e81ac",
            "cyan": "#8fbcbb",
            "green": "#a3be8c",
            "purple": "#b48ead",
            "red": "#bf616a",
            "white": "#eceff4",
            "yellow": "#ebcb8b"
          }
        ],
    
    "unbound"
    "keybindings": []
}


```

修改后保存即可看见效果。

### 添加自定义主题

在 powershell 中输入:  
`$Themesettings`  
会输出下面信息：

```
PromptSymbols        : {UNCSymbol, HomeSymbol, ElevatedSymbol, SegmentSeparatorForwardSymbol...}
CurrentUser          : Administrator
CurrentThemeLocation : C:\Users\Administrator\Documents\WindowsPowerShell\Modules\oh-my-posh\2.0.379\Themes\Paradox.psm                        1
Colors               : {AdminIconForegroundColor, PromptBackgroundColor, PromptHighlightColor, GitLocalChangesColor...} GitSymbols           : {OriginSymbols, LocalWorkingStatusSymbol, LocalDefaultStatusSymbol, BranchUntrackedSymbol...}    Options              : {ConsoleTitle, OriginSymbols}
ErrorCount           : 1
MyThemesLocation     : C:\Users\Administrator\Documents\WindowsPowerShell\PoshThemes


```

CurrentThemeLocation 即是现在主题的地址，在该目录下  
`C:\Users\Administrator\Documents\WindowsPowerShell\Modules\oh-my-posh\2.0.379\Themes\`新建一个文件`SpencerTechy.psm1`  
填入一下内容

```
function Write-Theme {
    param(
        [bool]
        $lastCommandFailed,
        [string]
        $with
    )

    $lastColor = $sl.Colors.PromptBackgroundColor
    $prompt += Write-Prompt -Object "╭" -ForegroundColor $sl.Colors.PromptIndicatorForegroundColor1

    $prompt = Write-Prompt -Object " $($sl.PromptSymbols.StartSymbol) " -ForegroundColor $sl.Colors.PromptForegroundColor -BackgroundColor $sl.Colors.PromptUserBackgroundColor
    $prompt += Write-Prompt -Object "$($sl.PromptSymbols.SegmentForwardSymbol) " -ForegroundColor $sl.Colors.PromptIndicatorForegroundColor3 -BackgroundColor $sl.Colors.PromptUserBackgroundColor
    
    If (Test-Administrator) {
        $prompt += Write-Prompt -Object "$($sl.PromptSymbols.ElevatedSymbol)" -ForegroundColor $sl.Colors.AdminIconForegroundColor -BackgroundColor $sl.Colors.PromptUserBackgroundColor
    }


    $user = [System.Environment]::UserName
    $computer = [System.Environment]::MachineName
    $path = Get-ShortPath -dir $pwd
    if (Test-NotDefaultUser($user)) {
        $prompt += Write-Prompt -Object "$user@$computer " -ForegroundColor $sl.Colors.PromptForegroundColor -BackgroundColor $sl.Colors.PromptUserBackgroundColor
    }

    if (Test-VirtualEnv) {
        $prompt += Write-Prompt -Object "$($sl.PromptSymbols.SegmentForwardSymbol)" -ForegroundColor $sl.Colors.SessionInfoBackgroundColor -BackgroundColor $sl.Colors.VirtualEnvBackgroundColor
        $prompt += Write-Prompt -Object "$($sl.PromptSymbols.VirtualEnvSymbol) $(Get-VirtualEnvName) " -ForegroundColor $sl.Colors.VirtualEnvForegroundColor -BackgroundColor $sl.Colors.VirtualEnvBackgroundColor
        $prompt += Write-Prompt -Object "$($sl.PromptSymbols.SegmentForwardSymbol)" -ForegroundColor $sl.Colors.VirtualEnvBackgroundColor -BackgroundColor $sl.Colors.PromptBackgroundColor
    }
    else {
        $prompt += Write-Prompt -Object "$($sl.PromptSymbols.SegmentForwardSymbol)" -ForegroundColor $sl.Colors.PromptUserBackgroundColor -BackgroundColor $sl.Colors.PromptBackgroundColor
    }

    
    $prompt += Write-Prompt -Object " $path " -ForegroundColor $sl.Colors.PromptForegroundColor -BackgroundColor $sl.Colors.PromptBackgroundColor

    $status = Get-VCSStatus
    if ($status) {
        $themeInfo = Get-VcsInfo -status ($status)
        $lastColor = $themeInfo.BackgroundColor
        $prompt += Write-Prompt -Object $($sl.PromptSymbols.SegmentForwardSymbol) -ForegroundColor $sl.Colors.PromptBackgroundColor -BackgroundColor $lastColor
        $prompt += Write-Prompt -Object " $($themeInfo.VcInfo) " -BackgroundColor $lastColor -ForegroundColor $sl.Colors.GitForegroundColor
    }

    
    $prompt += Write-Prompt -Object $sl.PromptSymbols.SegmentForwardSymbol -ForegroundColor $lastColor

    
    If ($lastCommandFailed) {
        $prompt += Write-Prompt -Object " $($sl.PromptSymbols.FailedCommandSymbol)" -ForegroundColor $sl.Colors.CommandFailedIconForegroundColor -BackgroundColor $sl.Colors.SessionInfoBackgroundColor
    } else {
        $prompt += Write-Prompt -Object " $($sl.PromptSymbols.SuccessCommandSymbol)" -ForegroundColor $sl.Colors.CommandSuccessIconForegroundColor -BackgroundColor $sl.Colors.SessionInfoBackgroundColor
    }

    
    

    
    
    
    
    

    $prompt += Set-Newline

    if ($with) {
        $prompt += Write-Prompt -Object "$($with.ToUpper()) " -BackgroundColor $sl.Colors.WithBackgroundColor -ForegroundColor $sl.Colors.WithForegroundColor
    }

    $prompt += Write-Prompt -Object "╰" -ForegroundColor $sl.Colors.PromptIndicatorForegroundColor1
    $prompt += Write-Prompt -Object $sl.PromptSymbols.PromptIndicator -ForegroundColor $sl.Colors.PromptIndicatorForegroundColor1
    $prompt += Write-Prompt -Object $sl.PromptSymbols.PromptIndicator -ForegroundColor $sl.Colors.PromptIndicatorForegroundColor2
    $prompt += Write-Prompt -Object $sl.PromptSymbols.PromptIndicator -ForegroundColor $sl.Colors.PromptIndicatorForegroundColor3
    $prompt += ' '
    $prompt
}

$sl = $global:ThemeSettings 
$sl.PromptSymbols.StartSymbol = [char]::ConvertFromUtf32(0xe70c)
$sl.PromptSymbols.PromptIndicator = [char]::ConvertFromUtf32(0x276F)
$sl.PromptSymbols.SegmentForwardSymbol = "" 
$sl.PromptSymbols.SegmentBackwardSymbol = [char]::ConvertFromUtf32(0xE0B2)
$sl.PromptSymbols.TimeSymbol = ' ' + [char]::ConvertFromUtf32(0x235F)
$sl.PromptSymbols.FailedCommandSymbol = [char]::ConvertFromUtf32(0x2718)
$sl.PromptSymbols.SuccessCommandSymbol = [char]::ConvertFromUtf32(0x2714)
$sl.Colors.PromptForegroundColor = [ConsoleColor]::White
$sl.Colors.PromptBackgroundColor = [ConsoleColor]::DarkBlue
$sl.Colors.PromptSymbolColor = [ConsoleColor]::White
$sl.Colors.PromptHighlightColor = [ConsoleColor]::DarkBlue
$sl.Colors.GitForegroundColor = [ConsoleColor]::Black
$sl.Colors.WithForegroundColor = [ConsoleColor]::DarkRed
$sl.Colors.WithBackgroundColor = [ConsoleColor]::DarkMagenta
$sl.Colors.VirtualEnvBackgroundColor = [System.ConsoleColor]::DarkRed
$sl.Colors.VirtualEnvForegroundColor = [System.ConsoleColor]::White
$sl.Colors.CommandSuccessIconForegroundColor = [System.ConsoleColor]::DarkGreen
$sl.Colors.CommandFailedIconForegroundColor = [System.ConsoleColor]::DarkRed
$sl.Colors.PromptIndicatorForegroundColor1 = [ConsoleColor]::DarkBlue
$sl.Colors.PromptIndicatorForegroundColor2 = [ConsoleColor]::DarkYellow
$sl.Colors.PromptIndicatorForegroundColor3 = [ConsoleColor]::DarkMagenta
$sl.Colors.PromptUserBackgroundColor = [ConsoleColor]::Black


```

或者在这个[地址下载](https://raw.githubusercontent.com/spencerwooo/dotfiles/master/Windows/SpencerTechy.psm1)

保存后，在 PowerShell 中输入`notepad $PROFILE`

修改最后一行的  
`Set-Theme Paradox`改为`Set-Theme SpencerTechy`，保存

### 另一个好主题 Powerlevel10k

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

官网：[https://github.com/romkatv/powerlevel10k](https://github.com/romkatv/powerlevel10k)

但是 Linux 下才能用下面方法安装，PowerShell 版只要把 psm1 文件附上即可

Oh My Zsh 版的安装方法.

```
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel10k


```

在`~/.zshrc`里加上

```
ZSH_THEME="powerlevel10k/powerlevel10k"


```

PowerShell 版的 `Powerlevel10k-Classic.psm1` 主题文件:

```
function Write-Theme {
    param(
        [bool]
        $lastCommandFailed,
        [string]
        $with
    )
    $adminsymbol = $sl.PromptSymbols.ElevatedSymbol
    $venvsymbol = $sl.PromptSymbols.VirtualEnvSymbol
    $clocksymbol = $sl.PromptSymbols.ClockSymbol

    
    $prompt = Write-Prompt -Object " $($sl.PromptSymbols.StartSymbol) " -ForegroundColor $sl.Colors.SessionInfoForegroundColor -BackgroundColor $sl.Colors.SessionInfoBackgroundColor
    $prompt += Write-Prompt -Object "$($sl.PromptSymbols.SegmentSubForwardSymbol) " -ForegroundColor $sl.Colors.PromptForegroundColor -BackgroundColor $sl.Colors.SessionInfoBackgroundColor
    $pathSymbol = if ($pwd.Path -eq $HOME) { $sl.PromptSymbols.PathHomeSymbol } else { $sl.PromptSymbols.PathSymbol }

    
    $path = $pathSymbol + " " + (Get-FullPath -dir $pwd) + " "
    $prompt += Write-Prompt -Object $path -ForegroundColor $sl.Colors.DriveForegroundColor -BackgroundColor $sl.Colors.SessionInfoBackgroundColor

    $status = Get-VCSStatus
    if ($status) {
        $themeInfo = Get-VcsInfo -status ($status)
        $prompt += Write-Prompt -Object $sl.PromptSymbols.SegmentSubForwardSymbol -ForegroundColor $sl.Colors.PromptForegroundColor -BackgroundColor $sl.Colors.SessionInfoBackgroundColor
        $prompt += Write-Prompt -Object " $($themeInfo.VcInfo) " -ForegroundColor $themeInfo.BackgroundColor -BackgroundColor $sl.Colors.SessionInfoBackgroundColor
    }
    If ($with) {
        $sWith = " $($with.ToUpper())"
        $prompt += Write-Prompt -Object $sl.PromptSymbols.SegmentSubForwardSymbol -ForegroundColor $sl.Colors.PromptForegroundColor -BackgroundColor $sl.Colors.SessionInfoBackgroundColor
        $prompt += Write-Prompt -Object $sWith -ForegroundColor $sl.Colors.WithForegroundColor -BackgroundColor $sl.Colors.SessionInfoBackgroundColor
    }
    $prompt += Write-Prompt -Object $sl.PromptSymbols.SegmentForwardSymbol -ForegroundColor $sl.Colors.SessionInfoBackgroundColor
    

    
    $rightElements = New-Object 'System.Collections.Generic.List[Tuple[string,ConsoleColor]]'
    $login = $sl.CurrentUser
    $computer = [System.Environment]::MachineName;

    $rightElements.Add([System.Tuple]::Create($sl.PromptSymbols.SegmentBackwardSymbol, $sl.Colors.SessionInfoBackgroundColor))
    
    if (Test-VirtualEnv) {
        $rightElements.Add([System.Tuple]::Create(" $(Get-VirtualEnvName) $venvsymbol ", $sl.Colors.VirtualEnvForegroundColor))
        $rightElements.Add([System.Tuple]::Create($sl.PromptSymbols.SegmentSubBackwardSymbol, $sl.Colors.PromptForegroundColor))
    }
    if (Test-Administrator) {
        $rightElements.Add([System.Tuple]::Create(" $adminsymbol", $sl.Colors.AdminIconForegroundColor))
    }
    $rightElements.Add([System.Tuple]::Create(" $login@$computer ", $sl.Colors.UserForegroundColor))
    $rightElements.Add([System.Tuple]::Create($sl.PromptSymbols.SegmentSubBackwardSymbol, $sl.Colors.PromptForegroundColor))
    $rightElements.Add([System.Tuple]::Create(" $(Get-Date -Format HH:mm:ss) $clocksymbol ", $sl.Colors.TimestampForegroundColor))
    $lengthList = [Linq.Enumerable]::Select($rightElements, [Func[Tuple[string, ConsoleColor], int]] { $args[0].Item1.Length })
    $total = [Linq.Enumerable]::Sum($lengthList)
    
    $prompt += Set-CursorForRightBlockWrite -textLength $total
    
    $prompt += Write-Prompt -Object $rightElements[0].Item1 -ForegroundColor $sl.Colors.SessionInfoBackgroundColor
    for ($i = 1; $i -lt $rightElements.Count; $i++) {
        $prompt += Write-Prompt -Object $rightElements[$i].Item1 -ForegroundColor $rightElements[$i].Item2 -BackgroundColor $sl.Colors.SessionInfoBackgroundColor
    }
    

    $prompt += Write-Prompt -Object "`r"
    $prompt += Set-Newline

    
    $indicatorColor = If ($lastCommandFailed) { $sl.Colors.CommandFailedIconForegroundColor } Else { $sl.Colors.PromptSymbolColor }
    $prompt += Write-Prompt -Object $sl.PromptSymbols.PromptIndicator -ForegroundColor $indicatorColor
    $prompt += ' '
    $prompt
}

$sl = $global:ThemeSettings 
$sl.PromptSymbols.StartSymbol = [char]::ConvertFromUtf32(0xe62a)
$sl.PromptSymbols.PromptIndicator = [char]::ConvertFromUtf32(0x276F)
$sl.PromptSymbols.SegmentForwardSymbol = [char]::ConvertFromUtf32(0xE0B0)
$sl.PromptSymbols.SegmentSubForwardSymbol = [char]::ConvertFromUtf32(0xE0B1)
$sl.PromptSymbols.SegmentBackwardSymbol = [char]::ConvertFromUtf32(0xE0B2)
$sl.PromptSymbols.SegmentSubBackwardSymbol = [char]::ConvertFromUtf32(0xE0B3)
$sl.PromptSymbols.ClockSymbol = [char]::ConvertFromUtf32(0xf64f)
$sl.PromptSymbols.PathHomeSymbol = [char]::ConvertFromUtf32(0xf015)
$sl.PromptSymbols.PathSymbol = [char]::ConvertFromUtf32(0xf07c)
$sl.Colors.PromptBackgroundColor = [ConsoleColor]::DarkGray
$sl.Colors.SessionInfoBackgroundColor = [ConsoleColor]::DarkGray
$sl.Colors.VirtualEnvBackgroundColor = [ConsoleColor]::DarkGray
$sl.Colors.PromptSymbolColor = [ConsoleColor]::Green
$sl.Colors.CommandFailedIconForegroundColor = [ConsoleColor]::DarkRed
$sl.Colors.DriveForegroundColor = [ConsoleColor]::Cyan
$sl.Colors.PromptForegroundColor = [ConsoleColor]::Gray
$sl.Colors.SessionInfoForegroundColor = [ConsoleColor]::White
$sl.Colors.WithForegroundColor = [ConsoleColor]::Red
$sl.Colors.VirtualEnvForegroundColor = [System.ConsoleColor]::Magenta
$sl.Colors.TimestampForegroundColor = [ConsoleColor]::Green
$sl.Colors.UserForegroundColor = [ConsoleColor]::Yellow
$sl.Colors.GitForegroundColor = [ConsoleColor]::White 


```

按照上节方法把在`$PROFILE`里改`Set-Theme Powerlevel10k-Classic`即可。

安装 WSL
------

在微软应用商店里搜索 Ubuntu 安装即可，安装完成后会在 Windows 任务栏搜索 bash 即可进入 Ubuntu 环境。

初次进入 WSL 会立刻安装 Ubuntu 然后提示添加用户和密码, 按照提示输入即可.

建议使用`lxrunoffline`安装 WSL：

```
scoop bucket add extras

scoop install lxrunoffline


choco install lxrunoffline


```

安装完成后，下载 [CentOS 的 Docker 镜像](https://github.com/CentOS/sig-cloud-instance-images/blob/CentOS-7.8.2003-x86_64/docker/centos-7.8.2003-x86_64-docker.tar.xz)

下载完毕之后，将其保存到一个全英文的目录中，执行以下的命令执行安装：

```
LxRunOffline install -n CentOS -d D:\linux\centos -f D:\softbackup\centos-7.8.2003-x86_64-docker.tar.xz -s


```

查看安装了哪些 WSL

```
LxRunOffline l


```

使用 LxRunOffline 运行 WSL:

```
LxRunOffline run -w -n "CentOS"


```

取消注册 CentOS 的命令:

```
LxRunOffline ur -n CentOS


```

以下命令可以将已经安装的 Ubuntu 移动到 D 盘：

```
LxRunOffline m -n Ubuntu -d D:\Linux\Ubuntu


```

查看 Ubutun 路径:

```
LxRunOffline di -n Ubuntu


```

为 Ubuntu 生成快捷方式：

```
LxRunOffline s -n Ubuntu -f C:\Users\{User}\Desktop\Ubuntu.lnk


```

备份 Ubutun：

```
LxRunOffline e -n Ubuntu -f D:\Linux\backup\ubuntu.tar.gz


```

恢复备份：

```
LxRunOffline i -n Ubuntu -d D:\Linux\ubuntu -f D:\Linux\backup\ubuntu.tar.gz


```

WSL 开发环境
--------

键入以下命令安装环境:

```
sudo apt-get update


sudo apt-get install git


sudo apt-get install zsh


zsh


```

### 安装 Oh-my-zsh

oh-my-zsh 首页在这:  
[https://github.com/ohmyzsh/ohmyzsh](https://github.com/ohmyzsh/ohmyzsh)

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"


```

安装完成后在设置默认 SHELL 时会出现了问题。

Linux 下设置默认 SHELL 方法如下：

```
chsh -s /bin/zsh


```

但重新在 CMD/POWERSHELL 上进入 WSL ，默认的 SHELL 还是 bash ，需要手动执行`$ zsh`才能进入。

于是在 Google 上查了一下，发现在 Microsoft 的 Github 上面有一个提交 Bug 的 Repository：Microsoft/WSL 上有一个 issue：

can’t change default shell #477

出现这个问题的原因是在启动 WSL 时没有执行 login 相关的组件，而这些组件和设置默认 SHELL 有关。

```
We don’t run login which is the component that normally sets those things up.


```

解决方法  
打开 ~./bashrc 添加下列代码进去并保存即可。

```
[[ $- == *i* ]] && $(command -v zsh) || echo "ZSH is not installed"


```

下载 oh-my-zsh 的 powerlevel9k 主题

```
git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k


```

编辑 .zshrc

```
vi ~/.zshrc


```

找到 ZSH_THEME  
添加下面内容

```
POWERLEVEL9K_MODE='nerdfont-complete'

POWERLEVEL9K_LEFT_SUBSEGMENT_SEPARATOR="%F{$(( $DEFAULT_BACKGROUND - 2 ))}|%f"

POWERLEVEL9K_RIGHT_SUBSEGMENT_SEPARATOR="%F{$(( $DEFAULT_BACKGROUND - 2 ))}|%f"
POWERLEVEL9K_SHORTEN_DIR_LENGTH="2"
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(os_icon context dir nvm vcs)
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status root_indicator battery date)
POWERLEVEL9K_OS_ICON_BACKGROUND="magenta"
POWERLEVEL9K_OS_ICON_FOREGROUND="black"
POWERLEVEL9K_DEFAULT_FOREGROUND="white"
POWERLEVEL9K_CONTEXT_DEFAULT_FOREGROUND="black"
POWERLEVEL9K_CONTEXT_DEFAULT_BACKGROUND="cyan"
POWERLEVEL9K_STATUS_CROSS="true"
ZSH_THEME="powerlevel9k/powerlevel9k"


```

重启 bash 可以生效.

或者用上一节的 Powerlevel10k 更简单.

### WSL 下安装 Python

#### 安装 Pyenv

Pyenv 官网: [https://github.com/pyenv/pyenv-installer](https://github.com/pyenv/pyenv-installer)

输入下面命令安装:

```
curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash


```

在 .zshrc 或者 .bashrc 中添加下面 3 行

```
export PATH="/home/gadzan/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"


```

输入`exec $SHELL`生效.

使用 pyenv 安装 python 3.6.6

```
pyenv install 3.6.6


```

如果提示`no acceptable C compiler found in $PATH`, 那就先安装依赖

```
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev


```

安装完成后先创建虚拟环境, 执行命令：

```
pyenv virtualenv 3.6.6 my-env


```

它的格式就是这样固定的，最后一个是你自己想要的环境的名字，可以随便取。稍等片刻，你将会看到：

```
Looking in links: /tmp/tmp0eywgc7v
Requirement already satisfied: setuptools in /home/joit/.pyenv/versions/3.6.6/envs/my-env/lib/python3.6/site-packages (39.0.1)
Requirement already satisfied: pip in /home/joit/.pyenv/versions/3.6.6/envs/my-env/lib/python3.6/site-packages (10.0.1)


```

类似于这样的回显信息，说明环境已经创建成功了，它还告诉了你，该虚拟环境的绝对路径，如果你进去看了，你就会发现，所谓的虚拟环境，就是把 python 装在 pyenv 的安装目录的某个文件夹中，以供它自己调用。

在任意目录下，执行命令激活虚拟环境：

```
pyenv activate my-env


```

你会发现，在你的终端里面，多了一个类似于 (my-env) 这样的一个东西，这时候你如果执行：

```
python --version


```

会显示`python 3.6.6`, 证明 python 已经安装好了.

#### 安装 NVM

NVM 官网:[https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

直接在官网获得下面命令输入以安装:

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash


```

在`.bashrc`或者`.zshrc`中添加下面两行.

```
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" 


```

#### 安装激活 Node 12

```
nvm install 12

nvm use 12


```

#### 安装 Yarn 和 tyarn

```
npm install -g yarn

npm install -g tyarn

yarn config set registry https://registry.npm.taobao.org


```

安装 mobaXterm
------------

Windows 下的 SSH 工具  
[官网下载安装](https://mobaxterm.mobatek.net/)

下载个人用的免费版本即可，在 Settings -> SSH -> SSH Settings 把 SSH keepalive 勾上。

安装 VScode
---------

[官网下载安装](https://code.visualstudio.com/)

### 设置

设置默认显示空格和 tab 符号  
打开 VScode，左上角 文件 -> 首选项 -> 设置，搜索栏里搜索关键字`whitespace` 在 Editor: Render Whitespace 里选择`all`。

### 安装插件

**[](https://marketplace.visualstudio.com/items?item>indent-rainbow</a></strong> 让空格加上颜色，看上去更容易对齐；<br>
<img class=)

**[](https://marketplace.visualstudio.com/items?item>Bracket Pair Colorizer2</a></strong> 用于着色匹配括号；<br>
<img class=)

**[](https://marketplace.visualstudio.com/items?item>GitLens</a></strong> git 日志查看插件<br>
<img class=)

**[](https://marketplace.visualstudio.com/items?item>vscode-icons</a></strong> 目录图标<br>
<img class=)

**[](https://marketplace.visualstudio.com/items?item>Atuo Rename Tag</a></strong> 修改 html 标签，自动帮你完成头部和尾部闭合标签的同步修改;<br>
<img class=)

**[](https://marketplace.visualstudio.com/items?item>Auto Close Tag</a></strong> 自动为你写上 html 关闭标签<br>
<img class=)*   [](https://marketplace.visualstudio.com/items?item>Auto Close Tag</a></strong> 自动为你写上 html 关闭标签<br>
    <img class=)
    
    [](https://marketplace.visualstudio.com/items?item>Auto Close Tag</a></strong> 自动为你写上 html 关闭标签<br>
    <img class=)**[vetur](https://marketplace.visualstudio.com/items?item>beautify</a></strong> 格式化代码工具<br>
    美化 javascript，JSON，CSS，Sass，和 HTML 在 Visual Studio 代码。</p>
    </li>
    <li>
    <p><strong><a href=)** vue 代码高亮插件
    
*   **[Vue VSCode Snippets](https://marketplace.visualstudio.com/items?item>ESLint</a></strong> Javascript 代码规范工具</p>
    </li>
    <li>
    <p><strong><a href=)** 前端框架 Vue.js 快捷代码助手  
    ![](https://cdn.gadzan.com/pic/20200528153519.gif)
    

************

******[

### 安装 VScode 主题

](https://marketplace.visualstudio.com/items?item>CodeSnap</a></strong> 为代码片段生成漂亮的图片<br>
<img class=)

[

### 安装 JetBrain Mono 字体

JetBrains Mono 字体是 JetBrains 开源的字体, 专门为程序员长期专注观看代码而设计, 各个字母数字辨识度非常高, 能很好地分清`1LI`, 有效减少疲劳.  
![](https://cdn.gadzan.com/pic/20200528154114.png)

](https://marketplace.visualstudio.com/items?item>Dracula Official</a><br>
<img class=)

[](https://marketplace.visualstudio.com/items?item>Dracula Official</a><br>
<img class=)[官网下载 https://www.jetbrains.com/lp/mono/](https://www.jetbrains.com/lp/mono/)

下载安装后, 在 VS Code 文件 -> 首选项 -> 设置里搜索`Font Family`项设置字体, 或者在`Font Ligatures`项点击`Edit in setting.json(在setting.json里编辑)`并在`{}`内按需增量输入以下内容.  
**`editor.fontLigatures`** 一定要有, 不然连字效果不会显示.

```
{
  "editor.fontFamily": "'JetBrains Mono', Consolas, 'Courier New', monospace",
  "editor.fontLigatures": true,
  "editor.fontSize": 14,
  "editor.lineHeight": 22,
  "editor.letterSpacing": 0.5,
  "editor.fontWeight": "400",
  "editor.cursorStyle": "line",
  "editor.cursorWidth": 2,
  "editor.cursorBlinking": "solid"
}


```

安装 Edge
-------

最新的 Edge 是基于 Chromium 的，所以支持直接安装 Chrome 的插件，[官网下载安装](https://microsoftedgewelcome.microsoft.com/zh-cn/)  
![](https://cdn.gadzan.com/pic/20200528154316.png)

### 插件

1.  [Proxy SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif) 网络调试代理插件
2.  [Talend API Tester - Free Edition](https://chrome.google.com/webstore/detail/talend-api-tester-free-ed/aejoelaoggembcahagimdiliamlcdmfm?hl=zh-CN) Restful API 调试工具
3.  [Postwoman Extension for Chrome](https://chrome.google.com/webstore/detail/postwoman-extension-for-c/amknoiejhlmhancpahfcfcfhllgkpbld?hl=zh-CN) Postwoman API 调试工具，[Demo 页](https://postwoman.io/)可以直接作为 PWA 使用，调试 http 页面需要使用自带的代理，[Github](https://github.com/liyasthomas/postwoman)
4.  [Charset](https://chrome.google.com/webstore/detail/charset/oenllhgkiiljibhfagbfogdbchhdchml?hl=zh-CN) 改变页面的编码，Google Chrome 在 55 版本以后删除了手动设置网站编码的功能。但是在部分设置不规范不正确的网站，新版浏览器无法准确判断其使用的编码，导致网站显示乱码。

在浏览器可用的 VSCode: code-server
---------------------------

[https://github.com/cdr/code-server](https://github.com/cdr/code-server)

参考[为 iPad 部署基于 VS Code 的远程开发环境](https://sspai.com/post/60456)

下载 [3.1.1 版本的 code-server](https://github.com/cdr/code-server/releases/tag/3.1.1)

```
curl -o code-server-3.1.1.tar.gz https://github.com/cdr/code-server/releases/download/3.1.1/code-server-3.1.1-linux-x86_64.tar.gz


```

```
tar xf code-server-3.1.1.tar.gz


cd code-server-3.1.1-linux-x86_64


```

设置访问密码

```
export PASSWORD="{YOUR_CODE_SERVER_PASSWORD}"


```

把`{YOUR_CODE_SERVER_PASSWORD}`替换成密码即可

可以放到`~/.bashrc`或者`~/.zshrc`里

运行

```
./code-server --host "0.0.0.0"


```

在浏览器里访问`{服务器 IP 地址}:{code-server 端口}`, 例如在本机访问`localhost:8080`(code-server 默认监听 8080 端口)

### iOS 上使用 VSCode remote

手机上的打开浏览器版的 Code server 服务，编辑时的体验不够好，可以用 iOS 的 APP `VSCode remote`，自建的 Code server 服务器可以免费使用。  
官网：[https://vseditor.app/](https://vseditor.app/)

![](https://cdn.gadzan.com/pic/20200528154651.png)

安装 PowerToys
------------

PowerToys 最近的更新，移除了 Window Walker，取而代之的是更为强大的 PowerToys Run，同时也新增了一款工具 Keyboard Manager。

![](https://cdn.gadzan.com/pic/20200528160115.png)  
官方目前给出了两种安装方法：

第一种就在[官网 Release 页](https://github.com/microsoft/PowerToys)下载 MSI 文件安装。  
第二种通过包管理器 WinGet，安装只需要一条命令即可：`winget install powertoys`

下载安装后运行 PowerToy 有可能会提示你必须先安装 .Net Core 服务, 点击确定会自动导航到下载页，下载安装即可，[地址在这里](https://dotnet.microsoft.com/download/dotnet-core/)

`alt + space`可以唤出 PowerToys Run

安装 WinGet
---------

WinGet 以后会在 Windows10 自带，Insider 版本会有，但是 1709 以后的版本可以直接去 WinGet 的官方 GitHub 仓库，在 [Release 页面](https://github.com/microsoft/winget-cli/releases)手动下载 WinGet 的安装程序（.appxbundle 文件）然后手动安装。（安装过程非常快...）  
如果不能双击打开文件安装，需要先去微软商店安装 `应用安装程序` 这个程序。

WinGet 在安装命令之后加上 --rainbow 的参数，会出现🌈进度条，哈哈

******

 ***** * *

![](https://cdn.gadzan.com/pic/20210126162751.jpg)

[![](https://cdn.gadzan.com/pic/cc4.png)](http://creativecommons.org/licenses/by/4.0/) 本作品采用[知识共享署名 4.0 国际许可协议](http://creativecommons.org/licenses/by/4.0/)进行许可。****