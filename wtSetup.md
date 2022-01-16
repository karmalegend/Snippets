### PS $profile
```
Import-Module oh-my-posh
oh-my-posh --init --shell pwsh --config 
%PATH TO THEMES REPLACE THIS% oh-my-posh\themes\capr4n.omp.json | Invoke-Expression
Import-Module -Name Terminal-Icons
```

#### oh my posh:
```winget install JanDeDobbeleer.OhMyPosh```

[font](https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/Meslo.zip)
<br>
<br>
## USE MesloLGL NF
<br>
<br>

### Terminal icons:
```PS> Install-Module -Name Terminal-Icons -Repository PSGallery```
<br>
<br>
<br>
<br>
  
# Wt (Windows Terminal):
-  Default profile: [Powershell 7](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.2)

- Default terminal app: Windows terminal

Replace `"defaults":`
<br> 
with:


```json 

"actions": 
    [
        {
            "command": "unbound",
            "keys": "ctrl+shift+w"
        },
        {
            "command": "unbound",
            "keys": "ctrl+2"
        },
        {
            "command": "unbound",
            "keys": "ctrl+shift+2"
        },
        {
            "command": 
            {
                "action": "copy",
                "singleLine": false
            },
            "keys": "ctrl+c"
        },
        {
            "command": 
            {
                "action": "newTab"
            },
            "keys": "ctrl+t"
        },
        {
            "command": "paste",
            "keys": "ctrl+v"
        },
        {
            "command": 
            {
                "action": "newTab",
                "index": 1
            },
            "keys": "ctrl+shift+t"
        },
        {
            "command": "find",
            "keys": "ctrl+shift+f"
        },
        {
            "command": 
            {
                "action": "splitPane",
                "split": "auto",
                "splitMode": "duplicate"
            },
            "keys": "alt+shift+d"
        },
        {
            "command": "closePane",
            "keys": "ctrl+w"
        }
    ],

    {   
        "defaults": 
        {
            "acrylicOpacity": 0.80000000000000004,
            "font": 
            {
                "face": "MesloLGL NF"
            },
            "useAcrylic": true
        },
        
```