# MIKROTIK-SCRIPT-LATENCIA-Y-ENVIO-EMAIL
MIKROTIK SCRIPT LATENCIA Y ENVIO CORREO ELECTRONICO


## Requisistos 

Configuracíon del correo electronico: Tools->Email ——– server, port, user, password, to y from (Correo electronico).

## Scritp agrega source del mikrotik

````
:global iptest IP-A-LA-CUAL-SE-LA-VA-SER-PING
:global Servicio Nombre_Servicio
:global email  correo@dominio.com
:global Empresa Nombre_Empresa
:global latenciAceitavel 20
:global tempoEntrePings 2
:global NumPings 10
:global Asunto "Alarma! Monitor de ping a IP: $iptest Empresa: $Empresa del servicio $Servicio"
##################################################

:global msg
:global status
:global oldstatus $status
:global somaLatencia 0
:global somaRecebidos 0 
##################################################

for i from=1 to=$NumPings do={
/tool flood-ping $iptest size=100 count=1 do={
:if ($received = 1) do={
:set somaLatencia ($somaLatencia + $"avg-rtt")
}
:set somaRecebidos ($somaRecebidos + $received)
}
delay $tempoEntrePings
}

:if ($somaRecebidos = 0 ) do={
:global status "HOST $iptest FUERA DE ALCANCE o ENLACE ESTA CAIDO de la empresa $Empresa"
:if ($oldstatus = $status) do={quit} else={


:log error $status
:global msg "Enlace caido"
#Enviar correo
/tool e-mail send to=$email subject=$Asunto body=("$status")
:log error "La alarma ha sido enviada.";
quit
}
}

:global media ($somaLatencia/$somaRecebidos)
:global percaPacotes (100 - (($somaRecebidos * 100) / $NumPings))

:global msg ("LA MEDIA de PING para LA IP $iptest FUE DE ".[:tostr $media]."ms y LA PERDIDA DE PAQUETES FUE DE ". [:tostr $percaPacotes]."%. FUERON ".$NumPings." Pings enviados y ".$somaRecebidos." recibidos.")

:if ($media < $latenciAceitavel ) do={
:global status "LATENCIA OK POR DEBAJO DE 20ms"
:if ($oldstatus = $status) do={quit} else={
:log warning $msg
:log warning $status
/tool e-mail send to=$email subject=$Asunto body=("$status \n\n $msg")
:log warning "La alarma ha sido enviada.";

}
} else={
:global status "LATENCIA DE PROVEEDOR (por encima de 20ms hacia $Empresa)"
:if ($oldstatus = $status) do={quit} else={


:log error $msg
:log error $status
/tool e-mail send to=$email subject=$Asunto body=("$status \n\n $msg")
:log warning "La alarma ha sido enviada.";
}
}
