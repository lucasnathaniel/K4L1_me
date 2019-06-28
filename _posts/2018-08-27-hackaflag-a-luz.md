---
title: HACKAFLAG 2018 - Etapa Belo Horizonte - Luz [PT-BR]
layout: post
date: '2018-08-27 9:00:00'
description: Writeup do desafio "Luz" da Etapa Belo Horizonte - HACKAFLAG 2018
image: https://i.imgur.com/aX3n7wf.png
tags:
- ctf
- writeup
author: shrimpgo
---
### "Luz" (500)

> Descrição:
>Há uma luz no fim do túnel. Acenda e encontre a saída.
>
>Autor: @dr1nKoRdi3
>
>Anexo: [challmisc.tar](https://ctf.hackaflag.com.br/135a1a234ec2eab47ff1cea3b8c34e45/challmisc.tar)
---

O arquivo `challmisc.tar` contém dois arquivos: uma foto chamada `A luz.jpg` e um binário chamado `hacking`. A foto contém um desenho de um circuito digital com várias conexões e o binário é um executável PE compilado com .NET:

```bash
$ file hacking
hacking: PE32 executable (GUI) Intel 80386 Mono/.Net assembly, for MS Windows
```

Executando o binário, aparece apenas um campo para inserir e o nome _Deja vú_, que dura apenas alguns segundos em execução e depois é finalizado.

![alt text](https://i.imgur.com/5I0hqQR.png)

Inicialmente, tentei realizar engenharia reversa no binário para tentar obter o máximo de informação possível com o decompilador dnSpy, porém o código estava ofuscado:

![alt text](https://i.imgur.com/ezPTRP6.png)

Tentei tirar a ofuscação com o de4dot, mas mesmo assim o código não ficou muito claro pra mim (mesmo porque eu não entendo muito de C#). Então resolvi tentar alguma coisa com o circuito, se pudesse dar alguma dica. A imagem é a seguinte:

![alt text](https://i.imgur.com/aX3n7wf.png)

Eu tive sistemas digitais na minha graduação, mas faz taaaanto tempo que muitas coisas eu havia esquecido. Tive que recorrer ao Google para relembrar os símbolos e até fiz alguns exercícios para fixar o aprendizado (pasmem!). Depois de analisar novamente o circuito, percebi que a saída para ligar a luz necessita de uma combinação na entrada para que a saída final seja 1 e, para acelerar o processo, eu fiz um código em python para obter quais combinações possuem a saída 1 (para acender a luz):

```python
#!/usr/bin/env python3

from binascii import *

T = 0
# São 256 possibilidades, pois há 8 entradas binárias (2**8)
while T < 256:
    # Transformo o inteiro em binário
    S = bin(T)[2:]
    # Nas duas próximas linhas, eu preencho de 0's à esquerda para complementar os oito dígitos das entradas
    size = 8 - len(S)
    M = '0'*size + S
    # Nesse ponto, eu interpretei o circuito, que nada mais é ligações lógicas de OR, AND, NOT etc.
    formula1 = ~ int(M[0]) & int(M[1]) & int(M[2]) & ~ int(M[3]) & ~ int(M[4]) & ~ int(M[5]) & ~ int(M[6]) & int(M[7])
    formula2 = ~ int(M[0]) & int(M[1]) & int(M[2]) & int(M[3]) & ~ int(M[4]) & ~ int(M[5]) & int(M[6]) & int(M[7])
    formula3 = ~ int(M[0]) & int(M[1]) & ~ int(M[2]) & ~ int(M[3]) & ~ int(M[4]) & int(M[5]) & int(M[6]) & ~ int(M[7])
    formula4 = ~ int(M[0]) & ~ int(M[1]) & int(M[2]) & ~ int(M[3]) & ~ int(M[4]) & ~ int(M[5]) & ~ int(M[6]) & ~ int(M[7])
    formula = formula1 | formula2 | formula3 | formula4
    # Aqui eu pego apenas a combinação que der saída 1 e imprimo quais satisfizeram a condição. Como são 8 dígitos, eu supus que poderia ser ASCII em binário e já transformei a saída binária em ASCII
    if formula == 1:
        print(M + " = " + str(chr(int(M, 2))))
    T += 1
```

A saída é a seguinte:

```bash
00100000 =  
01000110 = F
01100001 = a
01110011 = s
```

Opa! Aqui percebemos que 4 caracteres foram formados: espaço, a letra `F`, `a` e `s`. Nesse ponto foi o mais complicado porque eu não tinha mais pra onde ir, então comecei a tentar coisas malucas. A primeira tentativa que fiz foi verificar se a foto tinha algo escondido: usei a stegsolve.jar, strings e outras tool de stego e me sobrou tentar bruteforce com steghide. Comecei com números binários porque a foto e as lógicas levavam pra eu tentar por esse lado e, também, costuma ser mais rápido. Criei uma lista com o crunch:

```bash
$ crunch 8 8 01 -o lista.txt
Crunch will now generate the following amount of data: 2304 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 256 

crunch: 100% completed generating output
```

E depois usei o seguinte comando para tentar todas as senhas:

```bash
$ while read i; do echo -n "$i => "; steghide extract -sf A\ luz.jpg -p $i; if [[ $? == 0 ]]; then break; fi; done < lista.txt 
00000000 => steghide: could not extract any data with that passphrase!
00000001 => steghide: could not extract any data with that passphrase!
00000010 => steghide: could not extract any data with that passphrase!
00000011 => steghide: could not extract any data with that passphrase!
00000100 => steghide: could not extract any data with that passphrase!
00000101 => steghide: could not extract any data with that passphrase!
00000110 => steghide: could not extract any data with that passphrase!
00000111 => steghide: could not extract any data with that passphrase!
00001000 => steghide: could not extract any data with that passphrase!
00001001 => steghide: could not extract any data with that passphrase!
(...)
01110000 => steghide: could not extract any data with that passphrase!
01110001 => steghide: could not extract any data with that passphrase!
01110010 => steghide: could not extract any data with that passphrase!
01110011 => wrote extracted data to "ShowCode".
```

Eita! Saiu alguma coisa ali. Curiosidade: olhando a senha, ela corresponde a uma das saídas do circuito (a letra 's'). Eu acabei indo pelo lado mais difícil fazendo bruteforce, sendo que era somente tentar uma daquelas saídas dadas do meu script do circuito. Olhando o arquivo `ShowCode`:

```bash
01110101 ...DD... 01101001 01101110 01100111 ...AA... 01010011 01111001 ...DD... 01110100 01100101 01101101 00111011 00001101 00001010 01110101 ...DD... 01101001 01101110 01100111 ...AA... 01010011 01111001 ...DD... 01110100 01100101 01101101 00101110 01001101 ...CC... 01101110 ...CC... 01100111 01100101 01101101 01100101 01101110 01110100 00111011 00001101 00001010 01110101 ...DD... 01101001 01101110 01100111 ...AA... 01010011 01111001 ...DD... 01110100 01100101 01101101 00101110 01010111 01101001 01101110 01100100 01101111 01110111 ...DD... 00101110 ...BB... 01101111 01110010 01101101 ...DD... 00111011 00001101 00001010 00001101 00001010 01101110 ...CC... 01101101 01100101 ...DD... 01110000 ...CC... 01100011 01100101 ...AA... 01100100 01010010 01101001 01101110 01001011 01101111 01110010 01100100 01101001 01000101 00001101 00001010 01111011 00001101 00001010 ...AA... ...AA... ...AA... ...AA... 01110000 01110101 01100010 01101100 01101001 01100011 ...AA... 01100011 01101100 ...CC... ...DD... ...DD... ...AA... 01001101 01111001 01010001 01110101 01100101 01110010 01111001 00001101 00001010 ...AA... ...AA... ...AA... ...AA... 01111011 00001101 00001010 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 01110000 01110101 01100010 01101100 01101001 01100011 ...AA... ...DD... 01110100 ...CC... 01110100 01101001 01100011 ...AA... 01110110 01101111 01101001 01100100 ...AA... 01001101 ...CC... 01101001 01101110 00101000 00101001 00001101 00001010 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 01111011 00001101 00001010 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 01110100 01110010 01111001 00001101 00001010 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 01111011 00001101 00001010 00001001 00001001 00001001 00001001 ...DD... 01110100 01110010 01101001 01101110 01100111 ...AA... 01110000 ...CC... 01110100 01101000 ...AA... 00111101 ...AA... 00100010 01110010 01101111 01101111 01110100 01011100 01011100 01000011 01001001 01001101 01010110 00110010 00100010 00111011 00001101 00001010 00001001 00001001 00001001 00001001 ...DD... 01110100 01110010 01101001 01101110 01100111 ...AA... 01110001 01110101 01100101 01110010 01111001 01010011 01010001 01001100 ...AA... 00111101 ...AA... 00100010 01010011 01000101 01001100 01000101 01000011 01010100 ...AA... 00101010 ...AA... ...BB... 01010010 01001111 01001101 ...AA... 01010111 01101001 01101110 00110011 00110010 01011111 01000100 01101001 ...DD... 01101011 01000100 01110010 01101001 01110110 01100101 00100010 00111011 00001101 00001010 00001001 00001001 00001001 00001001 01001101 ...CC... 01101110 ...CC... 01100111 01100101 01101101 01100101 01101110 01110100 01001111 01100010 01101010 01100101 01100011 01110100 01010011 01100101 ...CC... 01110010 01100011 01101000 01100101 01110010 ...AA... ...DD... 01100101 ...CC... 01110010 01100011 01101000 01100101 01110010 ...AA... 00111101 ...AA... 01101110 01100101 01110111 ...AA... 01001101 ...CC... 01101110 ...CC... 01100111 01100101 01101101 01100101 01101110 01110100 01001111 01100010 01101010 01100101 01100011 01110100 01010011 01100101 ...CC... 01110010 01100011 01101000 01100101 01110010 00101000 01110000 ...CC... 01110100 01101000 00101100 ...AA... 01110001 01110101 01100101 01110010 01111001 01010011 01010001 01001100 00101001 00111011 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 00001101 00001010 00001001 00001001 00001001 00001001 01100110 01101111 01110010 01100101 ...CC... 01100011 01101000 ...AA... 00101000 01001101 ...CC... 01101110 ...CC... 01100111 01100101 01101101 01100101 01101110 01110100 01001111 01100010 01101010 01100101 01100011 01110100 ...AA... 01110001 01110101 01100101 01110010 01111001 01001111 01100010 01101010 ...AA... 01101001 01101110 ...AA... ...DD... 01100101 ...CC... 01110010 01100011 01101000 01100101 01110010 00101110 01000111 01100101 01110100 00101000 00101001 00101001 00001101 00001010 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 01111011 00001001 00001001 00001001 00001001 00001001 00001001 00001001 00001101 00001010 00001001 00001001 00001001 00001001 00001001 ...DD... 01110100 01110010 01101001 01101110 01100111 ...AA... 01110010 01100101 ...DD... 01110101 01101100 01110100 ...AA... 00111101 ...AA... 01000011 01101111 01101110 01110110 01100101 01110010 01110100 00101110 01010100 01101111 01010011 01110100 01110010 01101001 01101110 01100111 00101000 01110001 01110101 01100101 01110010 01111001 01001111 01100010 01101010 01011011 00100010 01010011 01101001 01100111 01101110 ...CC... 01110100 01110101 01110010 01100101 00100010 01011101 00101001 00111011 00001101 00001010 00001001 00001001 00001001 00001001 00001001 01110010 01100101 ...DD... 01110101 01101100 01110100 ...AA... 00111101 ...AA... 01000011 01101111 01101110 01110110 01100101 01110010 01110100 00101110 01010100 01101111 01010011 01110100 01110010 01101001 01101110 01100111 00101000 01110001 01110101 01100101 01110010 01111001 01001111 01100010 01101010 01011011 00100010 01010011 01100101 01110010 01101001 ...CC... 01101100 01001110 01110101 01101101 01100010 01100101 01110010 00100010 01011101 00101001 00111011 00001001 00001001 00001001 00001001 00001001 00001101 00001010 00001001 00001001 00001001 00001001 00001001 01110010 01100101 ...DD... 01110101 01101100 01110100 ...AA... 00111101 ...AA... 01110010 01100101 ...DD... 01110101 01101100 01110100 ...AA... 00101011 ...AA... 01000011 01101111 01101110 01110110 01100101 01110010 01110100 00101110 01010100 01101111 01010011 01110100 01110010 01101001 01101110 01100111 00101000 01110001 01110101 01100101 01110010 01111001 01001111 01100010 01101010 01011011 00100010 01010011 01101001 01111010 01100101 00100010 01011101 00101001 00111011 00001101 00001010 00001001 00001001 00001001 00001001 00001001 01010011 01111001 ...DD... 01110100 01100101 01101101 00101110 01000100 01101001 ...CC... 01100111 01101110 01101111 ...DD... 01110100 01101001 01100011 ...DD... 00101110 01010000 01110010 01101111 01100011 01100101 ...DD... ...DD... 00101110 01010011 01110100 ...CC... 01110010 01110100 00101000 00100010 01000011 01001101 01000100 00101110 01100101 01111000 01100101 00100010 00101100 ...AA... 01000000 00100010 00101111 01000011 ...AA... 01110111 01101101 01101001 01100011 ...AA... 01110111 01101101 01101001 ...DD... 01100101 01110100 ...AA... 00100110 00100110 ...AA... 01100101 01100011 01101000 01101111 ...AA... 00100010 ...AA... 00101011 ...AA... 01110010 01100101 ...DD... 01110101 01101100 01110100 ...AA... 00101011 ...AA... 01110010 01100101 ...DD... 01110101 01101100 01110100 00101110 01001100 01100101 01101110 01100111 01110100 01101000 ...AA... 00101011 ...AA... 00100010 ...AA... 00100110 00100110 ...AA... 01110111 01101101 01101001 01100011 ...AA... 01110111 01101101 01101001 ...DD... 01100101 01110100 00100010 00101001 00111011 00001101 00001010 00001001 00001001 00001001 00001001 00001001 01000011 01101111 01101110 ...DD... 01101111 01101100 01100101 00101110 01010111 01110010 01101001 01110100 01100101 01001100 01101001 01101110 01100101 00101000 00100010 01010001 01110101 ...CC... ...DD... 01100101 ...AA... 01101100 ...CC... 00101110 00101110 00101110 00100010 00101001 00111011 00001101 00001010 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 01100010 01110010 01100101 ...CC... 01101011 00111011 00001101 00001010 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 01111101 00001101 00001010 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 01111101 00001101 00001010 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 01100011 ...CC... 01110100 01100011 01101000 ...AA... 00101000 01001101 ...CC... 01101110 ...CC... 01100111 01100101 01101101 01100101 01101110 01110100 01000101 01111000 01100011 01100101 01110000 01110100 01101001 01101111 01101110 ...AA... 01100101 00101001 00001101 00001010 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 01111011 00001101 00001010 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 01001101 01100101 ...DD... ...DD... ...CC... 01100111 01100101 01000010 01101111 01111000 00101110 01010011 01101000 01101111 01110111 00101000 00100010 01010101 01101101 ...AA... 01100101 01110010 01110010 01101111 ...AA... ...CC... 01100011 01101111 01101110 01110100 01100101 01100011 01100101 01110101 00101100 ...AA... 01101100 ...CC... 01101101 01100101 01101110 01110100 01101111 ...AA... 00111011 01111110 00101000 ...AA... 00100010 ...AA... 00101011 ...AA... 01100101 00101110 01001101 01100101 ...DD... ...DD... ...CC... 01100111 01100101 00101001 00111011 00001101 00001010 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 01111101 00001101 00001010 ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... ...AA... 01111101 00001101 00001010 ...AA... ...AA... ...AA... ...AA... 01111101 00001101 00001010 01111101
```

WTF! Mais uma coisa pra desbravar. Observei que há muitos binários de 8 dígitos e, mais uma vez, eu os converti em ASCII e apareceu algumas coisas:

```cs
uingSytem;
uingSytem.Mngement;
uingSytem.Window.orm;

nmepcedRinKordiE
{
publicclMyQuery
{
publictticvoidMin()
{
try
{
				tringpth="root\\CIMV2";
				tringquerySQL="SELECT*ROMWin32_DikDrive";
				MngementObjectSercherercher=newMngementObjectSercher(pth,querySQL);
				forech(MngementObjectqueryObjinercher.Get())
{							
					tringreult=Convert.ToString(queryObj["Signture"]);
					reult=Convert.ToString(queryObj["SerilNumber"]);					
					reult=reult+Convert.ToString(queryObj["Size"]);
					Sytem.Dignotic.Proce.Strt("CMD.exe",@"/Cwmicwmiet&&echo"+reult+reult.Length+"&&wmicwmiet");
					Conole.WriteLine("Quel...");
brek;
}
}
ctch(MngementExceptione)
{
MegeBox.Show("Umerroconteceu,lmento;~("+e.Mege);
}
}
}
}
```

Hum... parece um código-fonte que tá faltando algumas letras. No arquivo `ShowCode`, percebe-se que há 4 grupos relativamente estranhos: `...AA...`, `...BB...`, `...CC...`, `...DD...`. Nas posições em que se encontram, nota-se que eles substituem algumas letras. Ex.: `Sytem` deveria ser `System`, então tá faltando a letra `s`. Depois da análise, percebi que as letras que faltaram eram exatamente aqueles binários que foram resultado da saída do circuito: `F`, `a`, `s` e espaço. Fiz um script para fazer as devidas substituições:

```python
#!/usr/bin/env python3

f = open('ShowCode', 'r').read().split(' ')
g = open('ShowCodeFixed.txt', 'w')

for i in f:
    try:
        g.write(chr(int(i, 2)))
    except ValueError:
        if i == '...DD...':
            g.write(i.replace('...DD...', 's'))
        elif i == '...BB...':
            g.write(i.replace('...BB...', 'F'))
        elif i == '...AA...':
            g.write(i.replace('...AA...', ' '))
        elif i == '...CC...':
            g.write(i.replace('...CC...', 'a'))
g.close()
```

E o resultado:

```cs
using System;
using System.Management;
using System.Windows.Forms;

namespace dRinKordiE
{
    public class MyQuery
    {
        public static void Main()
        {
            try
            {
				string path = "root\\CIMV2";
				string querySQL = "SELECT * FROM Win32_DiskDrive";
				ManagementObjectSearcher searcher = new ManagementObjectSearcher(path, querySQL);                
				foreach (ManagementObject queryObj in searcher.Get())
                {							
					string result = Convert.ToString(queryObj["Signature"]);
					result = Convert.ToString(queryObj["SerialNumber"]);					
					result = result + Convert.ToString(queryObj["Size"]);
					System.Diagnostics.Process.Start("CMD.exe", @"/C wmic wmiset && echo " + result + result.Length + " && wmic wmiset");
					Console.WriteLine("Quase la...");
                    break;
                }
            }
            catch (ManagementException e)
            {
                MessageBox.Show("Um erro aconteceu, lamento ;~( " + e.Message);
            }
        }
    }
}
```

Confesso que não sabia que linguagem era essa e resolvi buscar no Google alguma função definida ali e descobri que era C#. Então eu resolvi compilar o código. Busquei como fazer a compilação no Windows e fiz com o csc.exe, baseado nesse [link](https://docs.microsoft.com/pt-br/dotnet/csharp/language-reference/compiler-options/command-line-building-with-csc-exe). Renomeei o `ShowCodeFixed` para `code.cs` e, quando compilei e executei o código, abriu uma nova janela com uma saída gigante, ela fecha e o texto `Quase la...`. Com a primeira pesquisa realizada do conteúdo da string `root\\CIMV2`, encontrei esse [link](https://docs.microsoft.com/pt-br/windows/desktop/WinRM/windows-remote-management-and-wmi), que explica um pouco da estrutura WMI (Windows Management Instrumentation), que nada mais é uma infraestrutura de gerenciamento de dados no Windows. Pensei, então, em dissecar o código, entender realmente o que ele está fazendo:

```cs
using System;
using System.Management;
using System.Windows.Forms;

namespace dRinKordiE
{
    public class MyQuery
    {
        public static void Main()
        {
            try
            {
				//Aqui é escolhida a classe CIMV2 (dispositivos) e definida em uma variável
				string path = "root\\CIMV2";
				//Define outra variável para selecionar todos os objetos relacionados ao disco rígido
				string querySQL = "SELECT * FROM Win32_DiskDrive";
				//Cria novo objeto para manipular as informações da base com a classe definida
				ManagementObjectSearcher searcher = new ManagementObjectSearcher(path, querySQL);
				//Para cada item, ele vai armazenar o resultado obtido na variável "result"
				foreach (ManagementObject queryObj in searcher.Get())
                {							
					string result = Convert.ToString(queryObj["Signature"]);
					result = Convert.ToString(queryObj["SerialNumber"]);					
					result = result + Convert.ToString(queryObj["Size"]);
					//Aqui que é o problema, pois ele abre outro cmd.exe, executa o comando "wmic wmiset" antes e depois de imprimir o resultado de "result" mais o tamanho da "result" (resposta esperada)
					System.Diagnostics.Process.Start("CMD.exe", @"/C wmic wmiset && echo " + result + result.Length + " && wmic wmiset");
					Console.WriteLine("Quase la...");
                    break;
                }
            }
            catch (ManagementException e)
            {
                MessageBox.Show("Um erro aconteceu, lamento ;~( " + e.Message);
            }
        }
    }
}
```

Atentando ao detalhe da impressão do resultado obtido, resolvi executar _por fora_ esse comando que a função executou (wmic wmiset) e ele simplesmente imprime tudo que há dentro da base do WMI, ou seja, a saída é muito grande. Portanto, resolvi mudar o script para obter somente o resultado da variável "result":

```cs
(...)
					string result = Convert.ToString(queryObj["Signature"]);
					result = Convert.ToString(queryObj["SerialNumber"]);					
					result = result + Convert.ToString(queryObj["Size"]);
					Console.WriteLine(result + result.Lenght) //Removi o lixo que ele tava imprimindo anteriormente e deixei apenas o essencial
					//System.Diagnostics.Process.Start("CMD.exe", @"/C wmic wmiset && echo " + result + result.Length + " && wmic wmiset");
					Console.WriteLine("Quase la...");
(...)
```

O resultado é a impressão da `"Signature"+"SerialNumber"+"Size"+"result.Lenght"` do disco rígido, que dá um número bem grande. Por curiosidade, você pode obter esses mesmos objetos da classe utilizando o comando `wmic` disponível no Windows.
Com esse resultado obtido, decidi colocá-lo no campo oferecido pelo binário `hacking` e ver no que dá.

![alt text](https://i.imgur.com/zntrTK8.png)

![alt text](https://i.imgur.com/vdPdTqh.png)

A flag saiu quando inseri os dados. Portanto, o que o binário `hacking` fazia era uma comparação entre a consulta WMI do dispositivo no momento da execução do binário com os dados inseridos no campo oferecido. Se fossem iguais, a flag era impressa.
