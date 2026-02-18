[Recurso Flujo de trabjo](https://ako-team.atlassian.net/wiki/spaces/SD/pages/24248327/Flujo+de+trabajo+GitFlow)

[CheatShhet](https://git-scm.com/cheat-sheet)

>[!warning] Caso De uso con Error
>Hiciste un _push_ y, mientras tanto, el remoto recibió commits nuevos.  
Tu rama local y la remota quedaron **divergentes** y Git no supo cómo hacer el `pull`.
**Resultado aplicado:**  
Descargaste el estado del remoto y forzaste tu rama local a ser **idéntica** a él, descartando tu commit```git fetch
git reset --hard origin/develop```




