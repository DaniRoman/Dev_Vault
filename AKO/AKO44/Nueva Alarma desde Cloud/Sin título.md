param_SP_setpoint, el sp
param_C1-sp_delta, cuantos grados por encima por abajo del SP para que suene la alarma

la alarma max y min es:

param_A1_alarm_prb1_max?
param_A2_alarm_prb1_min

Con esto definir una alarma nueva en `conf { cloud: }`
mirar cada parte que significa:




Mirar el flujo como se dispara un alarma treshold  probando un mensaje de tipo `sample` para ver como se gestiona una alarma.


Ejecutar `sql` para tener las tablas

modificar sql y micro `save-alarms-tresholds `para ese tipo de alarma `continuos` ??


llega sample - calculo entrgadas digiytales y demas  calcula los treshold  () rellena el sample y lo manda al timeseries, y el lo almacena en una tabla, alarm treshold todo esto llega en cada sample solo tti y digital , respecto a los daylos 

meter un caso nuevo 

kver flujo como se dispara un alarma treshold  probando enviar un sample 


asi consigo ref

![[Pasted image 20260612133316.png]]

![[Pasted image 20260612133331.png]]

param_c_ag1_max_s1