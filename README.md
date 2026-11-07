# Servidor de WhatsApp — NexPlay

Este pacote conecta o seu número de WhatsApp ao painel NexPlay por QR Code.
Ele recebe as mensagens, envia para o robô de IA do painel e devolve a resposta ao cliente.

## O que tem no pacote

| Arquivo | Para que serve |
|---|---|
| `server.js` | o servidor (QR Code, status, envio e recebimento) |
| `package.json` | o que o servidor precisa para funcionar |
| `Dockerfile` | receita de instalação usada pelo Render |
| `render.yaml` | configura o Render sozinho (plano Starter + disco) |
| `.dockerignore` / `.gitignore` | arquivos que não devem subir |

Todos os arquivos ficam na **raiz** do repositório — não crie subpasta.

---

## Parte 1 — Subir no GitHub

1. Acesse github.com e clique em **New repository**.
2. Dê um nome, por exemplo `nexplay-whatsapp`, marque **Private** e clique em **Create repository**.
3. Clique em **Add file → Upload files**.
4. Arraste **todos** os arquivos deste pacote.
5. Confirme que `package.json`, `Dockerfile` e `render.yaml` aparecem na tela principal do repositório.
6. Clique em **Commit changes**.

> Os arquivos que começam com ponto (`.dockerignore` e `.gitignore`) podem ser criados pelo botão
> **Add file → Create new file**, digitando o nome exato.

---

## Parte 2 — Subir no Render

1. Acesse render.com e entre com a conta do GitHub.
2. Clique em **New → Blueprint**.
3. Escolha o repositório criado.
4. Confira apenas dois pontos:
   - **Plano: Starter** (o plano Free dorme e não tem disco, então a sessão do WhatsApp se perde).
   - **Disco montado em `/var/data`**.
5. Clique em **Apply** e espere o deploy ficar **Live** (3 a 8 minutos).
6. Copie o endereço mostrado no topo, algo como `https://nexplay-whatsapp.onrender.com`.

---

## Parte 3 — Testar antes de conectar

Abra no navegador:

```
https://SEU-ENDERECO.onrender.com/api/diagnostico
```

O ideal é ver:

- `"ok": true`
- `"pasta_persistente": true`
- `"permissoes_banco": "completa"`

Se aparecer algo na lista `problemas`, leia a mensagem — quase sempre é o disco fora de `/var/data`.

---

## Parte 4 — Conectar no painel

1. Abra a aba **WhatsApp** no painel.
2. Cole o endereço do servidor e clique em **Salvar**.
3. Clique em **Testar servidor**.
4. Clique em **Gerar QR Code** e leia no celular em *WhatsApp → Dispositivos conectados → Conectar dispositivo*.
5. Quando conectar, a tela mostra o número e o nome do perfil.

---

## Atualizar um servidor que já está no ar

O servidor precisa estar na versão **1.6.0** para responder os contatos novos do WhatsApp
(que chegam com identificador interno, o "LID"), confirmar no painel se a resposta da IA saiu,
**repassar as fotos enviadas pelos clientes** — é a foto da tela do aplicativo que libera o teste
gratuito — e **reconhecer as mensagens que o dono envia pelo celular**, pausando a IA naquela conversa.

1. Baixe o pacote de novo pelo botão **Baixar servidor (.zip)** na aba WhatsApp.
2. No GitHub, abra o repositório do servidor → **Add file → Upload files**.
3. Envie `server.js` (os outros arquivos não mudaram) e confirme em **Commit changes**.
4. O Render faz o deploy sozinho, aguarde ficar **Live**.
5. Volte ao painel e confirme em `/api/diagnostico` que aparece `"versao": "1.6.0"`.
6. Depois disso, mande uma mensagem de outro celular para o número conectado: o robô deve responder em poucos segundos.
7. Envie uma foto qualquer pelo WhatsApp: o robô deve responder confirmando o recebimento. Se responder isso, o teste automático está funcionando.
8. Responda você mesmo pelo celular para um cliente: a IA deve ficar calada naquela conversa por 1 hora — sem responder às suas mensagens.

---

## Rotas disponíveis

| Rota | Método | Função |
|---|---|---|
| `/api/diagnostico` | GET | verifica se está tudo certo |
| `/api/status` | GET | conexão, número, nome e QR |
| `/api/conectar` | POST | inicia a conexão e devolve o QR Code |
| `/api/qr` | GET | devolve o QR Code atual |
| `/api/desconectar` | POST | desconecta e apaga a sessão |
| `/api/enviar` | POST | envia mensagem (`numero` e `mensagem`) |
| `/api/agente` | POST | repassa uma consulta ao robô do painel |

---

## Avisos importantes

- Depois do primeiro QR Code lido, a sessão fica salva no disco e o servidor reconecta sozinho.
- Não troque o plano para Free: ele dorme e perde a sessão.
- Mantenha o endereço do serviço em sigilo — é o controle do seu WhatsApp.
- A IA só responde se a chave do Google Gemini (GEMINI_API_KEY) estiver cadastrada no Backend. Sem ela, as mensagens chegam e ficam aguardando você.