# Manual — Instalação Completa do Zero

Guia passo a passo para configurar o sistema **Água Limpa** localmente, incluindo os principais erros encontrados durante a configuração e suas respectivas soluções.

---

## 📋 Pré-requisitos

Antes de começar, certifique-se de ter:

* [ ] Windows com **Docker Desktop** instalado e aberto
* [ ] Uma conta Google para utilização do **Google Drive**
* [ ] Um número de WhatsApp disponível para conectar à automação

---

## 📁 Passo 1 — Preparar os arquivos do projeto

Crie uma pasta para o projeto, por exemplo:

```text
C:\Users\SEU_USUARIO\evolution-api
```

Dentro dessa pasta:

1. Coloque o arquivo `docker-compose.yml` deste repositório.
2. Copie o arquivo `.env.example`.
3. Crie um novo arquivo chamado `.env`.
4. Preencha os valores do `.env` com suas credenciais e senhas reais.

> ⚠️ **Importante:** nunca deixe os placeholders ou senhas de exemplo no arquivo `.env`. Isso já causou uma exposição de credenciais neste projeto.

---

## 🐳 Passo 2 — Subir os containers

Abra um terminal dentro da pasta do projeto e execute:

```bash
docker compose up -d
```

O comando deverá iniciar os seguintes containers:

* `evolution_postgres`
* `evolution_redis`
* `evolution_api`
* `n8n`

Abra o **Docker Desktop** e confirme se todos estão com o status:

```text
Running
```

---

## 📱 Passo 3 — Conectar o WhatsApp

Acesse o painel da Evolution API:

```text
http://localhost:8080/manager
```

Em seguida:

1. Crie uma instância chamada `agua-limpa`.
2. Escaneie o QR Code utilizando o WhatsApp que será conectado à automação.
3. Confirme se a conexão foi estabelecida.

Para consultar o estado da conexão pelo PowerShell:

```powershell
Invoke-RestMethod `
    -Uri "http://localhost:8080/instance/connectionState/agua-limpa" `
    -Headers @{"apikey"="SUA_CHAVE"} `
    -Method Get
```

O resultado esperado deverá indicar:

```text
state: open
```

---

## 🔗 Passo 4 — Configurar o Webhook da Evolution API

O webhook deve apontar para o n8n e receber os dois eventos utilizados pelo projeto:

* `MESSAGES_UPSERT`
* `GROUP_PARTICIPANTS_UPDATE`

Execute no PowerShell:

```powershell
$body = @{
    webhook = @{
        enabled = $true
        url = "http://n8n:5678/webhook/agua-limpa"
        events = @(
            "MESSAGES_UPSERT",
            "GROUP_PARTICIPANTS_UPDATE"
        )
    }
} | ConvertTo-Json -Depth 5

Invoke-RestMethod `
    -Uri "http://localhost:8080/webhook/set/agua-limpa" `
    -Headers @{
        "apikey" = "SUA_CHAVE"
        "Content-Type" = "application/json"
    } `
    -Method Post `
    -Body $body
```

### ⚠️ Erro comum no PowerShell

O `curl` do PowerShell não funciona exatamente como o `curl` tradicional.

Por isso, neste projeto, utilize:

```powershell
Invoke-RestMethod
```

Em vez de:

```bash
curl
```

Os headers devem ser enviados como um dicionário:

```powershell
-Headers @{"chave"="valor"}
```

e não utilizando a sintaxe:

```bash
-H
```

---

## ⚙️ Passo 5 — Importar o workflow no n8n

Acesse:

```text
http://localhost:5678
```

Depois:

1. Crie um novo workflow.
2. Importe o arquivo `agua-limpa-workflow.json` disponível neste repositório.
3. Edite os placeholders presentes no workflow.

Substitua:

| Placeholder               | Substituir por               | Ocorrências |
| ------------------------- | ---------------------------- | ----------: |
| `SUA_APIKEY_AQUI`         | Sua API Key real             |           3 |
| `SEU_GRUPO_AQUI@g.us`     | ID real do grupo do WhatsApp |           2 |
| `SUA_PASTA_DO_DRIVE_AQUI` | ID da pasta do Google Drive  |           1 |

> 🔐 Nunca publique API Keys, senhas ou outras credenciais reais no GitHub.

---

## 👥 Passo 6 — Descobrir o ID do grupo do WhatsApp

A maneira mais simples de descobrir o ID do grupo é:

1. Envie qualquer mensagem no grupo desejado.
2. Abra o n8n.
3. Acesse a aba **Executions**.
4. Abra a execução correspondente.
5. Abra o nó **Webhook**.
6. Localize:

```text
data.key.remoteJid
```

O valor encontrado será o ID do grupo.

Exemplo:

```text
1234567890-1234567890@g.us
```

---

## ☁️ Passo 7 — Configurar a credencial do Google Drive

Dentro do n8n:

1. Abra o nó **Upload no Drive**.
2. Crie uma nova credencial **OAuth2**.
3. Siga o processo de autorização do Google.
4. Faça login na conta Google.
5. Conceda as permissões solicitadas.
6. Selecione a pasta de destino no campo `folderId`.

### ⚠️ Erro comum — OAuth2

Ao executar o n8n localmente, a URL de redirecionamento do OAuth deve apontar para:

```text
localhost:5678
```

Ao migrar o sistema para um VPS, essa URL deverá ser atualizada para o IP ou domínio do servidor.

Caso contrário, a autenticação OAuth poderá deixar de funcionar.

---

## 🟢 Passo 8 — Ativar o workflow

No canto superior direito do editor do n8n, ative o toggle:

```text
Active
```

> ⚠️ **Importante:** se o workflow estiver desativado, o webhook não processará as mensagens, mesmo que toda a configuração esteja correta.

---

# 🛠️ Erros comuns

| Sintoma                                                                  | Causa                                                          | Solução                                                                                    |
| ------------------------------------------------------------------------ | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Execução nunca aparece no n8n ao enviar uma foto                         | Workflow inativo ou containers parados                         | Verifique o toggle `Active` e o status dos containers no Docker Desktop                    |
| Execução nunca aparece ao entrar um membro no grupo                      | Evento `GROUP_PARTICIPANTS_UPDATE` não está habilitado         | Execute novamente o comando do [Passo 4](#-passo-4--configurar-o-webhook-da-evolution-api) |
| Erro `ReferenceError` no nó de filtro                                    | Variável declarada sem `const`/`let` em modo estrito           | Declare as variáveis utilizando `const`                                                    |
| Mensagem de boas-vindas falha mesmo com execução "verde" até certo ponto | O formato do participante mudou entre versões da Evolution API | Verifique o payload real na aba **Executions** antes de assumir o formato                  |
| `curl` retorna erro de parâmetro no PowerShell                           | O PowerShell trata `curl` como alias de `Invoke-WebRequest`    | Utilize `Invoke-RestMethod` com `-Headers @{}`                                             |
| Sistema para de responder do nada                                        | PC suspendeu ou Docker Desktop foi fechado                     | Reabra o PC/Docker Desktop e aguarde a reconexão                                           |

---

# 🚀 Próximos passos

Os seguintes itens estão **fora do escopo deste manual**, mas são recomendados para uma implantação definitiva:

### ☁️ Migração para um VPS

Migrar o sistema para um VPS elimina a dependência de manter o computador local ligado.

Configuração recomendada:

* **Datacenter:** São Paulo
* **Sistema operacional:** Ubuntu 24.04
* **RAM:** 2 GB ou superior

### 🔐 Segurança

Antes de expor o sistema à internet:

* Troque todas as senhas de exemplo.
* Gere credenciais fortes e exclusivas.
* Nunca publique arquivos `.env` no GitHub.
* Nunca coloque API Keys diretamente no workflow público.
* Adicione arquivos contendo credenciais ao `.gitignore`.

---

## 📌 Checklist final

* [ ] Docker Desktop instalado e funcionando
* [ ] Arquivo `.env` configurado
* [ ] Containers em execução
* [ ] Instância `agua-limpa` criada
* [ ] WhatsApp conectado
* [ ] Webhook configurado
* [ ] Workflow importado no n8n
* [ ] API Key configurada
* [ ] ID do grupo configurado
* [ ] Google Drive conectado
* [ ] Pasta de destino configurada
* [ ] Workflow ativado
* [ ] Credenciais reais protegidas
