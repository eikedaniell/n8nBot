# 💧 Água Limpa — Automação de Fotos WhatsApp → Google Drive

Automação self-hosted que recebe fotos de produtos enviadas em um grupo do WhatsApp, salva automaticamente no Google Drive com nomenclatura inteligente, e responde no próprio grupo confirmando o salvamento.

## ✨ Funcionalidades

- 📸 Recebe fotos enviadas em um grupo específico do WhatsApp
- 🗂️ Salva automaticamente no Google Drive, em uma pasta configurável
- 🏷️ Nomeia o arquivo com base na legenda da foto — ou com data/hora, se não houver legenda
- ✅ Responde no grupo confirmando o salvamento, reenviando a própria foto com legenda formatada
- 👋 Envia mensagem de boas-vindas automática para novos membros do grupo, explicando como usar
- 🔒 Filtra mensagens por origem, ignorando conversas fora do grupo configurado

## 🏗️ Arquitetura

```
WhatsApp (grupo) → Evolution API → Webhook → n8n → Google Drive
                                                 ↓
                                        Confirmação de volta ao WhatsApp
```

| Componente | Papel |
|---|---|
| **Evolution API** | Conexão com o WhatsApp (via Baileys), leitura de QR Code, disparo de webhooks |
| **PostgreSQL** | Persistência de dados da Evolution API |
| **Redis** | Cache e controle de sessões/filas da Evolution API |
| **n8n** | Orquestrador: recebe o webhook, filtra, processa e decide as ações |
| **Google Drive** | Armazenamento final das fotos |

Todos os serviços rodam via Docker Compose, na mesma rede interna.

## 🚀 Como rodar

### Pré-requisitos

- Docker e Docker Compose instalados
- Uma conta Google (para o Drive)
- Um número de WhatsApp para conectar
- Pelo menos 15-20 GB de espaço livre em disco

### Passos

1. Clone este repositório
2. Copie `.env.example` para `.env` e preencha com valores reais:
   ```bash
   cp .env.example .env
   ```
3. Suba os containers:
   ```bash
   docker compose up -d
   docker compose ps
   ```
   Aguarde todos os serviços ficarem `healthy`.
4. Acesse `http://localhost:8080/manager`, crie uma instância e conecte o WhatsApp via QR Code
5. Acesse `http://localhost:5678` (n8n), importe o workflow em `/workflow/agua-limpa-workflow.json`
6. Configure a credencial OAuth do Google Drive dentro do n8n (veja `docs/MANUAL.md` para o passo a passo detalhado, incluindo os erros mais comuns e como evitá-los)
7. Configure o webhook da instância da Evolution API apontando para a **Production URL** do workflow no n8n
8. Publique o workflow e teste enviando uma foto no grupo configurado

📖 Consulte [`docs/MANUAL.md`](docs/MANUAL.md) para o guia completo, passo a passo, com todos os erros comuns já mapeados e suas soluções.

## ⚙️ Variáveis de ambiente

Veja `.env.example` para a lista completa. Nunca commite o arquivo `.env` real — ele contém credenciais.

## 📁 Estrutura do repositório

```
.
├── docker-compose.yml       # Definição dos serviços
├── .env.example              # Modelo de variáveis de ambiente
├── workflow/
│   └── agua-limpa-workflow.json   # Workflow do n8n (importável)
├── docs/
│   └── MANUAL.md              # Guia completo de instalação e erros comuns
└── README.md
```

## 🗺️ Roadmap / ideias futuras

- [ ] Sincronizar exclusão: apagar mensagem no WhatsApp → apagar arquivo no Drive
- [ ] Comandos de texto no grupo (`!menu`, `!deletar`)
- [ ] Agrupar confirmações quando várias fotos chegam de uma vez
- [ ] Deploy em VPS para operação 24/7 independente de máquina local

## ⚠️ Avisos importantes

- Este projeto usa a [Evolution API](https://github.com/EvolutionAPI/evolution-api), que se conecta ao WhatsApp através de uma biblioteca não-oficial (Baileys). Use por sua conta e risco — o WhatsApp pode restringir números que usem clientes não-oficiais.
- Nunca versione o arquivo `.env` real, nem exporte workflows do n8n com credenciais, IDs de grupos, ou chaves de API reais embutidas nos nós.
- Recomendado rodar primeiro em ambiente de teste antes de usar com um número de produção.

## 📄 Licença

MIT
