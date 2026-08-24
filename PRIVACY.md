# Política de Privacidade — n8nBot (Água Limpa Automation)

**Última atualização:** 24 de agosto de 2026

## Sobre este aplicativo
Este é um projeto de automação pessoal (self-hosted), que utiliza a API do Google Drive para monitorar uma pasta específica e processar arquivos automaticamente através de um workflow no n8n. Não é um serviço comercial e não é distribuído a terceiros.

## Dados acessados
O aplicativo solicita acesso à API do Google Drive exclusivamente para:

- Detectar a criação de novos arquivos em uma pasta específica e pré-configurada pelo usuário.
- Ler e baixar o conteúdo desses arquivos para processamento local.

Nenhum outro dado da conta Google (e-mails, contatos, agenda, etc.) é acessado.

## Uso dos dados
Os arquivos baixados são processados localmente pela instância self-hosted do n8n, operada e controlada exclusivamente pelo proprietário do projeto. Os dados não são compartilhados, vendidos, armazenados em servidores de terceiros, nem utilizados para nenhuma finalidade além da automação descrita no repositório do projeto.

## Compartilhamento de dados
Este aplicativo não compartilha dados com terceiros. Nenhuma informação coletada via Google Drive é transmitida a serviços externos, exceto quando explicitamente configurado pelo próprio usuário (ex.: envio de arquivos para um grupo de WhatsApp específico, controlado pelo mesmo usuário, através da Evolution API auto-hospedada).

## Retenção de dados
Os dados permanecem apenas no ambiente self-hosted do usuário (Docker/n8n) e no Google Drive original. Não há retenção adicional em servidores externos.

## Contato
Dúvidas sobre esta política podem ser enviadas através do repositório: [https://github.com/eikedaniell/n8nBot](https://github.com/eikedaniell/n8nBot)
