---
title: HACKAFLAG 2018 - Etapa São Paulo - Siga a api [PT-BR]
layout: post
date: '2018-09-18 10:21:00'
description: Writeup do desafio "Siga a api" da Etapa São Paulo - HACKAFLAG 2018
image: https://i.imgur.com/xWUW5rP.png
tags:
- ctf
- writeup
author: shrimpgo
---
### "Siga a api" (500)

> Descrição:
> Essa misc é um net enrustido, siga o dump até o usário.
>
>
> #sejabonzinho não owne o full stack ;) 
>
> Autor: @boot 
> Anexo: [rappi.pcapng](https://ctf.hackaflag.com.br/0901f1be29d16e6e0fed1e5c46382f6e/rappi.pcapng)
---

O pcap mostra diversos acessos em HTTPS, mas há alguns acessos em HTTP. Filtrando os acessos apenas por HTTP, percebi diversos acessos à API de um aplicativo de pedidos chamado Rappi.

![alt text](https://imgur.com/heRWiYa.png)

Cada acesso aos endpoints da API não informa muita coisa, mas possui a autenticação do usuário em todos os acessos.

![alt text](https://imgur.com/zp2XhCA.png)

Simulei todos esses acessos usando o `BurpSuite` e realmente as respostas são as mesmas do pcap, mas não pude obter maiores detalhes de como explorar todas as funcionalidades que a API fornecia, portanto eu instalei o aplicativo, fiz um cadastro e tentei capturar os pacotes forçando no meu celular o acesso do aplicativo com o certificado do `Burpsuite`. Acabei me dando mal porque o aplicativo possuía a proteção de `Certificate Pinning` (o apk ignora os certificados imposto pelo sistema Android e só acessa à Internet através do certificado fixo no apk).

Extraí o apk [dessa forma](https://www.wikihow.tech/Extract-APK-File-of-Any-App-on-Your-Android-Phone) e decompilei o mesmo com o `apktool`:

```bash
$ apktool d -o output rappi.apk
I: Using Apktool 2.3.3-dirty on Rappi_com.grability.rappi.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /home/shrimp/.local/share/apktool/framework/1.apk
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Baksmaling classes2.dex...
I: Baksmaling classes3.dex...
I: Baksmaling classes4.dex...
I: Baksmaling classes5.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
```

A partir deste ponto, eu perdi noites de sono tentando reconstruir o apk e sempre dava erro nas seções de resources e resolvi dar um tempo nesse desafio. Depois que o campeonato acabou, recebi uma hint: haviam outros endpoints existentes no código decompilado. Voltei a tentar e comecei a tentar diversos filtros e um dos filtros que deu um resultado melhor foi algo relacionado a `user` (de acordo com a hint do enunciado):

```bash
$ cd output; grep -ri .*-user *
smali/com/crashlytics/android/c/u.smali:    const-string v1, "crashlytics-userlog-"
smali/com/crashlytics/android/c/u.smali:    const-string v1, "crashlytics-userlog-"
smali_classes2/com/rappi/base/s/b/b.smali:        value = "ms/application-user/auth"
smali_classes2/com/rappi/base/s/b/c.smali:        value = "api/application-users/profile-pic"
smali_classes2/com/rappi/base/s/b/c.smali:        value = "api/application-users"
smali_classes2/com/rappi/base/s/b/c.smali:        value = "api/application-users/update-phone"
smali_classes2/com/rappi/base/s/b/c.smali:        value = "api/application-users/media-notifications"
smali_classes2/com/rappi/base/p/b/a/e.smali:        value = "api/orders/history-user"
smali_classes2/com/rappi/base/p/b/a/a.smali:        value = "ms/application-user/auth/addresses"
smali_classes2/com/rappi/base/p/b/a/a.smali:        value = "api/application-users/address"
smali_classes2/com/rappi/base/p/b/a/a.smali:        value = "user-data-center/application-user/change-location"
smali_classes2/com/rappi/base/p/b/a/a.smali:        value = "api/application-users/address/{addressId}"
smali_classes2/com/rappi/base/p/b/a/a.smali:        value = "api/application-users/address"
smali_classes2/com/rappi/base/p/b/a/b.smali:        value = "api/application-users/check-email"
smali_classes3/com/rappi/favorites/g/a.smali:        value = "/api/orders/favorites-user"
smali_classes3/com/rappi/favor/h/a.smali:        value = "api/orders/history-user?store_type=courier_hours"
smali_classes4/com/rappi/market/i/b/e.smali:        value = "api/orders/history-user"
smali_classes4/com/rappi/market/i/b/e$b.smali:        value = "api/orders/history-user"
smali_classes4/com/rappi/market/i/b/h.smali:        value = "api/application-users"
smali_classes4/com/rappi/referralcode/d/a.smali:        value = "api/application-users/referred-codes"
smali_classes4/com/rappi/pay/g/a/a.smali:        value = "api/application-users/find-users-by-phone"
smali_classes5/io/conekta/conektasdk/Connection$a.smali:    const-string v2, "Conekta-Client-User-Agent"
smali_classes5/com/rappi/restaurants/i/a/a/b.smali:        value = "api/orders/history-user"
smali_classes5/com/rappi/whims/f/a.smali:        value = "api/orders/history-user?store_type=whim"
```

Alguns dos endpoints mostrados já estavam no pcap, mas haviam outros que eu não conhecia. O primeiro endpoint do resultado já me entregou a flag:

![alt text](https://imgur.com/xWUW5rP.png)

### Observações

* Acredito que há outra forma de buscar a flag, que é pelo próprio site do [Rappi](https://www.rappi.com.br), mas eu não consegui me logar na plataforma porque eu tinha esquecido a senha (haha!) e achei que conseguiria reconstruir o apk, mas falhei. Por eu ter ficado frustrado, acabei não tentando novamente;
* Mesmo depois de muito tempo após o fim do campeonato, resolvi escrever o writeup porque achei o desafio muito legal, bem realista, mereceu!
