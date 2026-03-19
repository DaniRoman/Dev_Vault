Conexión layer 

Administración > FirmwareUpdate > Launch firmware, buscar este endPoint y hacer un fork, si es nuevo va a un topic si es viejo a otro

Esos topics me llevan a la conexión layer para que esta envie un cmd con petición de firmware update

En el Script de pruebas tengo que ver como puedo hacer un envio para solicitar el _/fota_ o el _/PLMN_

Una vez echa esa solicitud en la conexión layer llega esa petición y cuando llega una petición _/fota_ o el _/PLMN_ tiene que devolver el ultimo fichero o firmware disponible para ese numero de serie

