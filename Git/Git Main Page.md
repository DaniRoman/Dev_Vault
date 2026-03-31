[Recurso Flujo de trabjo](https://ako-team.atlassian.net/wiki/spaces/SD/pages/24248327/Flujo+de+trabajo+GitFlow)
[CheetSeet](https://www.atlassian.com/git/tutorials/atlassian-git-cheatsheet)

>[!example] Flujo para crear rama y track con la origin

```bash
git fetch origin
git switch -c feature/KNT-2253-no-arch --track origin/feature/KNT-2253-no-arch
```

>[!example] Flujo para borrar rama

```bash
git branch -D nombreRama 
```

>[!error] Revertir commits una vez echo el ***`push`***.
>
>`git reset --soft <destino>` mueve el `HEAD` al commit indicado y deja los cambios en **staged**.  
`git reset --mixed <destino>` mueve el `HEAD` al commit indicado y deja los cambios en local **sin staged**.  
`git reset --hard <destino>` mueve el `HEAD` al commit indicado y **borra** los cambios locales posteriores en archivos trackeados.  
`HEAD~1`, `HEAD~2`, etc. sirven para volver atrás un número de commits; usar un `hash` te lleva a un commit exacto.

