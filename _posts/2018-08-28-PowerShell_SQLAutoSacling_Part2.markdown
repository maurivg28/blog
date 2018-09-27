---
title:  "Autoescalado de una base en SQL Azure - Parte 2"
date:   2018-09-27 18:30:00
categories: [Powershell, Azure, SQL]
tags: [PowerShell, Azure, Cloud, Microsoft, DevOps]
---
Continuando con las mejoras del script, tuve la necesidad de hacer algunas modificaciones para que dicho script me avise si el job demora mas de "x" tiempo o si el progreso se encuentra "colgado" y ejecute un proceso de cancelacion.
Es por eso que continuando con lo que les explicaba en el captitulo 1, les voy a compartir esta segunda version mejorada del script.

![PowerShell_Logo]({{ site.baseurl }}/images/PowerShell_Logo.JPG)

### Script de autoescalado version mejorada ###

```powershell
#Script para autoescalado de SQL como servicio.
#Autor: Mauricio Villagran.
#Fecha: 20/8/2018
#Version: 2.0
#En la automation acount de Azure, se deberan cargar las siguientes variables:
#SendGrid_UserName (ira el usuario de Send Grid).
#SendGrid_Password (password del usario de Send Grid).
#SendGrid_EmailFrom (direccion de correo desde donde enviaremos el estatus de ejecucion del script).
#SendGrid_EmailTo (direccion o lista de distribucion donde recibiremos el estatus del script).
#SendGrid_Subject (asunto del correo.)

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
[string] $tier,

[parameter(Mandatory=$false,HelpMessage="Minutos maximo que durara el proceso")]
[string] $minutes = "90"
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

  Send-MailMessage -smtpServer $SMTPServer `
  -Credential $credential `
  -Usessl -Port 587 `
  -from $EmailFrom `
  -to $EmailTo `
  -subject $Subject `
  -Body $Body -BodyAsHtml

}

try{

#Obtengo status inicial de la DB (Plan y Size)
$sqlDB = Get-AzureRmSqlDatabase `
-ResourceGroupName $resourceGroupName `
-ServerName $serverName `
-DatabaseName $databaseName

Write-Output "DB name: $($sqlDB.DatabaseName)" | timestamp
Write-Output "DB estatus: $($sqlDB.Status), edition: $($sqlDB.Edition), tier: $($sqlDB.CurrentServiceObjectiveName)" | timestamp

#Ejecuto comandos de escalamiento de DB (Antes compruebo que la DB no este el es size y plan que paso por paramero.)
if ($edition.String -ne $sqlDB.Edition) {

  Write-Output "Escalando DB: $($sqlDB.DatabaseName)" |timestamp

  Set-AzureRmSqlDatabase -ResourceGroupName $resourceGroupName `
  -DatabaseName $databaseName `
  -ServerName $serverName `
  -Edition $edition `
  -RequestedServiceObjectiveName $tier -AsJob

  #Obtengo estado del job
  $sqlDBactivity = Get-Job | ?{$_.State -eq "Running"}

  #Obtengo el tiempo y fecha de inicio del job
  $startTime = $sqlDBactivity.PSBeginTime

  while ($sqlDBactivity.State -eq "Running") {

    sleep -Seconds 5

    #Obtengo estado actualizado del job
    $sqlDBactivity = Get-Job | ?{$_.State -eq "Running"}

    #Obtengo la hora actual
    $nowTime = Get-Date

    $diffTimes = New-TimeSpan -Start $startTime -End $nowTime

    if ($diffTimes.Minutes -gt $minutes) {

      Write-Output "El proceso de autoescalado esta demorado.! Estado de la DB: $($sqlDB.DatabaseName), es: $($sqlDB.Status)" | timestamp

      $sqlDB = Get-AzureRmSqlDatabaseActivity `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName

      #Obtengo solo el job que este con estatus InProgress y los siguientes datos State, OperationId, PercentComplete
      $sqlDBstatus = $sqlDB | ?{$_.State -eq "InProgress"} | Select State, OperationId, PercentComplete

      if ($sqlDBstatus.State -like "InProgress" -and $sqlDBstatus.PercentComplete -eq "0") {

      #Obtengo el operationId
      #$id = $sqlDB | ?{$_.State -eq "InProgress"} | Select OperationId
      $id = $sqlDBstatus

      Write-Output "Ejecutando comando para cancelar update de la DB: $($sqlDB.DatabaseName)" | timestamp
      Write-Output "El id del job es: $($id.OperationId)" | timestamp

      Stop-AzureRmSqlDatabaseActivity `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName `
      -OperationId $id.OperationId.Guid -Confirm:$false
      }

      Write-Output "Proceso Cancelado.!" | timestamp

      #Obtengo status post cancelacion de la DB (Plan y Size)
      $sqlDB = Get-AzureRmSqlDatabase `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName

      #Cuerpo del correo en formato HTML
    $Body = @"
    <html>
    <body>
    <p><strong>El escalamiento de la siguiente DB:&nbsp;<span style="color: #993300;">$($sqlDB.DatabaseName) <span style="color: #000000;">esta demorado.!</span></span></strong></p>
    <p><strong><span style="color: #993300;"><span style="color: #000000;">Se procedio a cancelar el escalamiento!</span></span></strong></p>
    <p><strong><span style="color: #993300;"><span style="color: #000000;">Favor de verificar!!!</span></span></strong></p>
    <p><strong><span style="color: #993300;"><span style="color: #000000;">El estado de la DB es.!</span></span></strong></p>
    <table style="height: 93px; border-color: #000000; width: 654px;" border="1">
    <tbody>
    <tr style="height: 22px;">
    <td style="width: 182px; background-color: #fe2e2e; text-align: center;">
    <p><strong>Nombre</strong></p>
    </td>
    <td style="width: 74px; background-color: #fe2e2e; text-align: center;">
    <p><strong>Estado</strong></p>
    </td>
    <td style="width: 195px; background-color: #fe2e2e; text-align: center;">
    <p><strong>Plan</strong></p>
    </td>
    <td style="width: 202px; height: 22px; background-color: #fe2e2e; text-align: center;">
    <p>Size</p>
    </td>
    </tr>
    <tr style="height: 20.75px;">
    <td style="width: 182px; text-align: center;">$($sqlDB.DatabaseName)</td>
    <td style="width: 74px; text-align: center;">$($sqlDB.Status)</td>
    <td style="width: 195px; text-align: center;">$($sqlDB.Edition)</td>
    <td style="width: 202px; height: 20.75px; text-align: center;">$($sqlDB.CurrentServiceObjectiveName)</td>
    </tr>
    </tbody>
    </table>
    <p>&nbsp;</p>
    </body>
    </html>
"@

      Write-Output "Enviando mail de alerta.!" | timestamp
      sendEmail -Body $Body
      Exit
   }
}

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

  Write-Output "El size de la DB fue cambiado!" | timestamp
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

[Autoescalado SQL Azure captitulo 1][Autoescalado Cap1]

[Autoescalado Cap1]: https://blog.mauriciovillagran.uy/2018/PowerShell_SQLAutoSacling/

[Set-AzureRmSqlDatabase][Set-AzureRmSqlDatabase]

[Set-AzureRmSqlDatabase]:https://docs.microsoft.com/en-us/powershell/module/azurerm.sql/set-azurermsqldatabase?view=azurermps-6.8.0
