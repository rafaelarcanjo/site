---
title: "Para nunca mais esquecer - Debug de segredos do Github Actions"
date: 2024-04-17T18:47:04-03:00
draft: false
toc: false
images:
tags:
  - devops
  - github
---

Todas as vezes que eu preciso *debugar* um segredo do Github Actions, seja para confirmar se o erro foi meu ou do *owner* do repositório, ou simplesmente porque necessito muito desse segredo, perco bastante tempo no Google procurando esta solução.

Então, decidi deixar registrado aqui, tornando fácil consultar posteriormente.

A solução é criar uma sessão SSH através do tmate.

```yaml
# Primeiro, faço echo do segredo para um arquivo dentro do workspace do workflow
- name: Set up secret file
  env:
    DEBUG_PASSWORD: ${{ secrets.DEBUG_PASSWORD }}
    DEBUG_SECRET_KEY: ${{ secrets.DEBUG_SECRET_KEY }}
  run: |
    echo $DEBUG_PASSWORD >> secrets.txt
    echo $DEBUG_SECRET_KEY >> secrets.txt

# Agora, crio a sessão
- name: Run tmate
  uses: mxschmitt/action-tmate@v2
```

Agora, basta acessar a sessão por SSH no link que é disponibilizado na saída do fluxo de trabalho e verificar o conteúdo do arquivo secrets.txt.

Referência: [Stackoverflow](https://stackoverflow.com/questions/63003669/how-can-i-see-my-git-secrets-unencrypted)
