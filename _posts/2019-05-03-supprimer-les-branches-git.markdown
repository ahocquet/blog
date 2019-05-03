---
layout: single
title:  "Comment supprimer les branches locales de Git sous Windows"
date:   2019-05-03 09:00:00 +0200
comments: true
tags: git powershell devops
---

Lorsque ca fait longtemps que vous bossez sur un repo, il est fort probable que vous vous retrouviez avec de nombreuses de `branch` en locale qui n'existent plus sur le `remote origin`. Voici comment faire le ménage sous Windows avec `PowerShell`.

Pour cela, vous aurez besoin d'installer :
- `chocolatey` afin d'installer des packages rapidement & facilement
- `gawk`
- `grep`
- Et bien sur, `git` !

C'est parti :
``` powershell
choco install grep
choco install gawk --version 5.0.0
```


Une fois que tout est installé, il se peut qu'il soit nécessaire de relancer powershell pour prendre en compte le `path` que choco aurait mis à jour.

Premierement, supprimons les `remote branch` qui n'existent plus : 
``` powershell
git remote prune origin 
```

Et maintenant, place à la magie : 
``` powershell
 git branch -vv | grep 'origin/.*: gone]' | awk '{print $1}' | %{git branch -D $_}
 ```

Et voilà ce que ça donne :
[![Console screenshot](/assets/images/2019-05-03-supprimer-les-branches-git/ConEmu64_2019-05-03_08-50-13.png){: .align-center}](/assets/images/2019-05-03-supprimer-les-branches-git/ConEmu64_2019-05-03_08-50-13.png)