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


```json {   
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