
>[!error] Error seteando un nuevo manufactured device 
> ya que este manufactured device es nuevo y no esta la `comercialVersion` especificada en el doc de licencias

`manufacturedDevice`

![[Pasted image 20260209120031.png]]

El atributo `licenseCode` me dice que tipo de licencia tiene si vamos al documento de este ..

![[Pasted image 20260209120149.png]]

En `commercialVersions` me dice para las versiones que esta esta licencia
El dispositivo que estoy registrando es un panel de 1 o 2 reles con lo cual para encontrar la `commercialVersion` tengo que ir a `ako44 > akocloud-api > schemas > device-definitions > panel_1ry_7102, panel_2ry_7102`)

![[Pasted image 20260209120850.png]]

Cojo los names y lo copio en la colección `licensecodes` `
 