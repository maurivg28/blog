---
title:  "Autoescalado de una base en SQL Azure - Parte 1"
date:   2018-08-28 21:46:00
categories: [Powershell, Azure, SQL]
tags: [PowerShell, Azure, Cloud, Microsoft, DevOps]
---
Hace unas semanas que estamos resolviendo diferentes problemas de carga sobre una infraetsructura montada en Azure para un importante cliente. La infraestructura soporta una aplicacion web para medios de pago, hay dias que la infraestructura debe soportar mayor carga. Luego de un analisis con el equipo de infra de mi trabajo, vimos que era conveniente autoescalar la base de datos de sql como servicio, los dias en que hay mayor carga. Si bien en la comunidad Microsoft ya hay algunos script, decidi hacer uno a nuestra medida. La idea de este capitulo es mostrarles como funciona el script, y no ir al detalle de los demas servicios que he utilizado para que funcione.

![PowerShell_Logo]({{ site.baseurl }}/images/PowerShell_Logo.JPG)

## Introducción ##

Generar el autoescalado de una base de datos como servicio en Azure, es tan sencillo como contar con los siguientes "ingredientes":

- Automation Account - Es un servicio de Azure el cual nos permite lanzar diferentes tipos de scripts, para esta esta cuenta debemos ejecutar una actualizacion de todos los modulos.
- Servicio SMTP - En mi caso utilice un servicio de terceros que esta publicado en Azure llamado SendGrid, el cual te permite enviar hasta 25 mil mails por mes.
- Script - Un script creado en PowerShell, utilizando las cmdlets disponibles para manejar una DB de SQL como servicio en Azure.

## Funcionamiento ##

Antes que nada, debemos preguntarnos que quiero que realice el script y que puntos de falla podriamos llegar a tener.
Partimos de la base, que vamos a utilizar un comando para el escalado de la base de datos.
Aqui tenemos un ejemplo claro del comando base por el cual va a nacer el script.

En este ejemplo escalaremos a un plan Standard S2

```
PS C:\>Set-AzureRmSqlDatabase -ResourceGroupName "ResourceGroup01" -DatabaseName "Database01" -ServerName "Server01" -Edition "Standard" -RequestedServiceObjectiveName "S2"
ResourceGroupName             : ResourceGroup01
ServerName                    : Server01
DatabaseName                  : Database01
Location                      : Central US
DatabaseId                    : a1e6bd1a-735a-4d48-8b98-afead5ef1218
Edition                       : Standard
CollationName                 : SQL_Latin1_General_CP1_CI_AS
CatalogCollation              :
MaxSizeBytes                  : 268435456000
Status                        : Online
CreationDate                  : 7/3/2015 7:33:37 AM
CurrentServiceObjectiveId     : 455330e1-00cd-488b-b5fa-177c226f28b7
CurrentServiceObjectiveName   : S2
RequestedServiceObjectiveId   : 455330e1-00cd-488b-b5fa-177c226f28b7
RequestedServiceObjectiveName :
ElasticPoolName               :
EarliestRestoreDate           :
Tags                          :
```

El objetivo principal del script es que reciba ciertos parametros, se ejecute a una hora especifica y escale a un plan mayor al que se encuentra actualmente. Cuando termina el proceso de escalamiento envie un mail en formato html, presentando el estado anterior y el estado actual.

A continuacion les voy a compartir el script, el cual es bastante sencillo, y luego les voy a dejar un video para mostrarles su funcionamiento. Esta es la version 1.0 del script, pero ya estoy trabajando en una version 2.0 la cual es capaz de avisarnos y cancelar el proceso de autoescalado si el mismo no se completa en el tiempo que le pasamos por parametro.
En un proximo capitulo les voy a estar comentando y compartiendo como me fue con esta segunda version del script.

### Script de autoescalado V1.0 ###

```
#Script para autoescalado de SQL como servicio.
#Autor: Mauricio Villagran.
#Fecha: 20/8/2018
#Version: 1.0
#En la automation acount de Azure, se deberan cargar las siguientes variables:
#SendGrid_UserName (ira el usuario de Send Grid).
#SendGrid_Password (password del usario de Send Grid).
#SendGrid_EmailFrom (direccion de correo desde donde enviaremos el estatus de ejecucion del script).
#SendGrid_EmailTo (direccion o lista de distribucion donde recibiremos el estatus del script).
#SendGrid_Subject (asunto del correo electronico.)

param(
[parameter(Mandatory=$true,HelpMessage="Resource Group Name")]
[string] $resourceGroupName,

[parameter(Mandatory=$false)]
[string] $azureRunAsConnectionName = "AzureRunAsConnection",

[parameter(Mandatory=$true,HelpMessage="SQL Name Server")]
[string] $serverName,

[parameter(Mandatory=$true,HelpMessage="Database Name")]
[string] $databaseName,

[parameter(Mandatory=$true,HelpMessage="Basic,Standard,Premium")]
[string] $edition,

[parameter(Mandatory=$true,HelpMessage="Basic,S1,S2,S4,S6")]
[string] $tier
)

filter timestamp {"[$(Get-Date -Format G)]: $_"}

Write-Output "Iniciando script de escalado." | timestamp

$VerbosePreference = "Continue"
$ErrorActionPreference = "Stop"

#Autentico con Azure Automation Run As account (service principal)
$runAsConnectionProfile = Get-AutomationConnection `
-Name $azureRunAsConnectionName
Add-AzureRmAccount -ServicePrincipal `
-TenantId $runAsConnectionProfile.TenantId `
-ApplicationId $runAsConnectionProfile.ApplicationId `
-CertificateThumbprint ` $runAsConnectionProfile.CertificateThumbprint | Out-Null
Write-Output "Autenticando con Automation Run As Account."  | timestamp

#Funcion para el envio de correo.
function sendEmail {
  Param(
    [parameter(Mandatory=$true)]
    [String] $Body
)
#Variables de conexion SMTP
  $Username = Get-AutomationVariable -Name 'SendGrid_UserName'
  $Password = Get-AutomationVariable -Name 'SendGrid_Password'
  $Password = $Password | ConvertTo-SecureString -AsPlainText -Force
  $credential = New-Object System.Management.Automation.PSCredential $Username, $Password
  $SMTPServer = "smtp.sendgrid.net"
  $EmailFrom = Get-AutomationVariable -Name 'SendGrid_EmailFrom'
  $EmailTo = Get-AutomationVariable -Name 'SendGrid_EmailTo'
  $Subject = Get-AutomationVariable -Name 'SendGrid_Subject'

  Send-MailMessage -smtpServer $SMTPServer -Credential $credential -Usessl -Port 587 -from $EmailFrom -to $EmailTo -subject $Subject -Body $Body -BodyAsHtml

}

try{

#Obtengo status inicial de la DB (Plan y Size)
$sqlDB = Get-AzureRmSqlDatabase `
-ResourceGroupName $resourceGroupName `
-ServerName $serverName `
-DatabaseName $databaseName
Write-Output "DB name: $($sqlDB.DatabaseName)" | timestamp
Write-Output "DB estatus: $($sqlDB.Status), edition: $($sqlDB.Edition), tier: $($sqlDB.CurrentServiceObjectiveName)" | timestamp

#Ejecuto comandos de escalamiento de DB (Antes compruebo que la DB no este el es size y plan que parametrizo.)
if ($edition -ne $sqlDB) {

  Write-Output "Escalando DB: $($sqlDB.DatabaseName)" |timestamp

  Set-AzureRmSqlDatabase -ResourceGroupName $resourceGroupName -DatabaseName $databaseName -ServerName $serverName -Edition $edition -RequestedServiceObjectiveName $tier

  Write-Output "El size de la DB fue cambiado!" | timestamp

#Obtengo el status final de la DB (Plan y Size)
  $sqlDB2 = Get-AzureRmSqlDatabase `
 -ResourceGroupName $resourceGroupName `
 -ServerName $serverName `
 -DatabaseName $databaseName

  Write-Output "El nuevo status es: $($sqlDB2.Status), edition: $($sqlDB2.Edition), tier: $($sqlDB2.CurrentServiceObjectiveName)" | timestamp

  #Cuerpo del correo en formato HTML
  $Body = @"
<html>
<body>
<p><strong>Se escalo de la siguiente DB:&nbsp;<span style="color: #993300;">$($sqlDB.DatabaseName)</span></strong></p>
<p>El size inicial de la DB: <span style="color: #993300;"><strong>$($sqlDB.DatabaseName)</strong></span>&nbsp;era.!</p>
<table style="height: 90px; border-color: #000000;" border="1" width="625">
<tbody>
<tr style="height: 22px;">
<td style="width: 149.25px; height: 22px; background-color: #2e9afe; text-align: center;">
<p><strong>Nombre</strong></p>
</td>
<td style="width: 149.25px; height: 22px; background-color: #2e9afe; text-align: center;"><strong>Estatus</strong></td>
<td style="width: 149.25px; height: 22px; background-color: #2e9afe; text-align: center;"><strong>Plan</strong></td>
<td style="width: 149.25px; height: 22px; background-color: #2e9afe; text-align: center;"><strong>Size</strong></td>
</tr>
<tr style="height: 20.75px;">
<td style="width: 149.25px; height: 20.75px; text-align: center;">$($sqlDB.DatabaseName)</td>
<td style="width: 149.25px; height: 20.75px; text-align: center;">$($sqlDB.Status)</td>
<td style="width: 149.25px; height: 20.75px; text-align: center;">$($sqlDB.Edition)</td>
<td style="width: 149.25px; height: 20.75px; text-align: center;">$($sqlDB.CurrentServiceObjectiveName)</td>
</tr>
</tbody>
</table>
<p>El size final de la DB: <span style="color: #993300;"><strong>$($sqlDB.DatabaseName)</strong></span>&nbsp;es.!</p>
<table style="border-color: #000000;" border="1" width="625">
<tbody>
<tr style="height: 29.796875px;">
<td style="background-color: #2e9afe; height: 29.796875px;">
<p style="text-align: center;"><strong>Nombre</strong></p>
</td>
<td style="background-color: #2e9afe; text-align: center; height: 29.796875px;"><strong>Estatus</strong></td>
<td style="background-color: #2e9afe; text-align: center; height: 29.796875px;"><strong>Plan</strong></td>
<td style="background-color: #2e9afe; text-align: center; height: 29.796875px;"><strong>Size</strong></td>
</tr>
<tr style="height: 18px;">
<td style="height: 18px; text-align: center;">$($sqlDB2.DatabaseName)</td>
<td style="height: 18px; text-align: center;">$($sqlDB2.Status)</td>
<td style="height: 18px; text-align: center;">$($sqlDB2.Edition)</td>
<td style="height: 18px; text-align: center;">$($sqlDB2.CurrentServiceObjectiveName)</td>
</tr>
</tbody>
</table>
</body>
</html>
"@
#Invocacion de la funcion de envio por mail, para el envio del estado.
  Write-Output "Enviando resultado por mail.!" | timestamp
  sendEmail -Body $Body
  Write-Output "Resultado enviado.!" | timestamp
}
else {

  Write-Output "El plan ya se encuentra en: $($sqlDB.Edition) y el size es: $($sqlDB2.CurrentServiceObjectiveName)"
  Write-Output "No se ejecuta aumento de plan.!"

 }

}
catch {
  #Paso los errores de ejecucion en la salida de error y envio el estado por mail.
  $ex = $_.Exception.Message
  Write-Error -Message $ex

  Write-Output "Enviando error por mail.!" | timestamp
  sendEmail -Body $ex
  Write-Output "Error enviado.!" | timestamp
}

Write-Output "Script finalizado.!" | timestamp
```
### Demo ###
{%include youtubePlayer.html id="QV5scwBWQoI"%}

Happy Scripting!!! :D
### Links de interés: ###

[Set-AzureRmSqlDatabase][Set-AzureRmSqlDatabase]

[Set-AzureRmSqlDatabase]: https://docs.microsoft.com/en-us/powershell/module/azurerm.sql/set-azurermsqldatabase?view=azurermps-6.8.0
