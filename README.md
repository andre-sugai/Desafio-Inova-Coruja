# Desafio Inova Coruja - Automação para Qualificação de Leads

Este repositório contém a solução para o desafio de criação de um agente SDR (Sales Development Representative) utilizando N8N. O workflow foi projetado com foco em uma solução de classificação de mensagens recebidas via WhatsApp para áudio e texto, também possui um tratamento de erros explícito para garantir que nenhuma interação com um lead seja perdida.

## Entregáveis

- **Workflow:** O arquivo `Agente_SDR_Inova_Coruja.json` está neste repositório.
- **Planilha de Teste:** Os resultados dos testes, tanto de sucesso quanto de falha, podem ser visualizados na seguinte planilha do Google Sheets.
  - **Link:** https://docs.google.com/spreadsheets/d/1jCjgChf0DcvYph-tjEc0BAC9cjP3nzgO90v3nC8JpLE/edit?usp=sharing

---

## 1. Como Importar e Configurar o Workflow

Siga os passos abaixo para importar e executar o workflow no seu ambiente N8N.

### 1.1. Importando o Arquivo JSON

1.  Na sua dashboard do N8N, clique em **"Create Workflow"**.
2.  Clique nos 3 pontinhos no canto superior direito da tela e escolha a opção **"Import from file..."**.
3.  Faça o upload do arquivo `Agente_SDR_Inova_Coruja.json` contido neste repositório.
4.  O workflow será carregado no editor.

### 1.2. Configurando as Credenciais

Este workflow requer duas credenciais para funcionar corretamente.

#### a) Credencial da OpenAI

Utilizada para a transcrição de áudio (Whisper) e para a qualificação das mensagens (GPT).

1.  No workflow, abra qualquer um dos nós **OpenAI** (ex: `Transcreve Audio`, `Classifica Audio`).
2.  No campo `Credential to connect with`, selecione a opção OpaneAI account ou crie uma nova credencial.
3.  No campo `API Key`, cole a sua chave secreta da API da OpenAI (que começa com `sk-...`).
4.  Salve a credencial. O N8N a aplicará automaticamente em todos os nós OpenAI.

#### b) Credencial do Google Sheets

Utilizada para registrar os leads qualificados (caminho de sucesso) e as falhas de processamento (caminho de erro).

1.  No workflow, abra qualquer um dos nós **"Planilha de Registro"**.
2.  No campo `Credential to connect with`, selecione a credencial Google Sheets account ou crie uma nova credencial.
3.  Siga os passos de autenticação da sua conta do Google.
4.  **Importante:** Este workflow utiliza múltiplos nós do Google Sheets. Certifique-se de que todos estão apontando para o documento correto (Leads SDR Teste).

---

## 2. Exemplo de Payload para Teste do Webhook

Você pode testar o workflow enviando requisições `POST` para a URL de teste do nó `Recebe Mensagem` via terminal.

### 2.1. Teste com Mensagem de Texto

Classificação Quente

```bash
curl -X POST https://n8n.inovacoruja.com.br/webhook-test/lead-whatsapp-sdr \
  -H "Content-Type: application/json" \
  -d '{"nome":"Johnny Teste","mensagem":"Quero comprar o produto X"}'
```

Classificação Frio

```bash
curl -X POST https://n8n.inovacoruja.com.br/webhook-test/lead-whatsapp-sdr \
  -H "Content-Type: application/json" \
  -d '{"nome”:”Rob Teste","mensagem”:”Não quero saber mais sobre o produto X"}'
```

Classificação Morno

```bash
curl -X POST https://n8n.inovacoruja.com.br/webhook-test/lead-whatsapp-sdr \
  -H "Content-Type: application/json" \
  -d '{"nome”:”Richard Teste","mensagem":"Quero saber mais sobre o produto "}'
```

###2.2. Teste com Mensagem de Áudio

Classificação Morno

```bash
curl -X POST https://n8n.inovacoruja.com.br/webhook-test/lead-whatsapp-sdr \
  -H "Content-Type: application/json" \
  -d '{"nome":"Lemmy Teste", "audio_url":"https://ar-projeto.netlify.app/1.mp3", "type":"audio"}'
```

Classificação Frio

```bash
curl -X POST https://n8n.inovacoruja.com.br/webhook-test/lead-whatsapp-sdr \
  -H "Content-Type: application/json" \
  -d '{"nome":"George Teste", "audio_url":"https://ar-projeto.netlify.app/2.mp3", "type":"audio"}'
```

Classificação Quente

```bash
curl -X POST https://n8n.inovacoruja.com.br/webhook-test/lead-whatsapp-sdr \
  -H "Content-Type: application/json" \
  -d '{"nome":"Sid Teste", "audio_url":"https://ar-projeto.netlify.app/3.mp3", "type":"audio"}'
```

Nota: Lembre-se de substituir POST https://n8n.inovacoruja.com.br/webhook-test/lead-whatsapp-sdr pela URL correta fornecida pelo nó Recebe Mensagem no editor do N8N.

###2.2. Teste de erro com Mensagem de Áudio

Antes de executar o teste de erro para mensagem de áudio, clique no nó Classifica Audio, no menu dropdown de Credential to connect with escolha a opção OpenAI account 2.

Execute qualquer teste de mensagem de áudio.

###2.3. Teste de erro com Mensagem de Texto

Antes de executar o teste de erro para mensagem de texto, clique no nó Classifica Texto, no menu dropdown de Credential to connect with escolha a opção OpenAI account 2.

Execute qualquer teste de mensagem de texto.

3. Principais Validações e Tratamento de Erros
   A arquitetura do workflow foi desenhada para ser compacta e funcional, tratando tanto os "caminhos felizes" quanto os "caminhos de falha".
   3.1. Workflow de Mensagens
   O primeiro nó, chamado Recebe Mensagem, recebe o payload e extrai as principais informações.
   O segundo nó, chamado Captura Timestamp, é responsável pelo registro de hora e data do recebimento da informação.
   Nó IF: O fluxo utiliza um nó IF como ponto central de decisão. Ele verifica a existência do campo audio_url. Se presente, o fluxo é direcionado para a rota de processamento de áudio; caso contrário, segue pela rota de texto. Isso garante uma separação clara e inequívoca dos tipos de mensagem.
   3.2. Arquitetura de Tratamento de Erros Explícito
   Para garantir a robustez e a não perda de dados, foi implementado um tratamento de erro dedicado para os pontos mais críticos do fluxo: a classificação pela IA.
   Captura de Falhas: A saída de erro dos nós Classifica Audio e Classifica Texto foi utilizada. Se a API da OpenAI falhar (por chave inválida, instabilidade ou qualquer outro motivo), o fluxo é automaticamente desviado para um caminho de tratamento de erro, em vez de simplesmente parar.
   Formatação do Erro: Um nó Formata Erro (Set) é acionado no caminho da falha. Sua função é capturar a mensagem de erro técnica ({{ $json.error.message }}) e prepará-la para um registro legível. Isso também preserva os dados originais do lead (nome, timestamp, etc.).
   Registro de Falhas: As falhas são registradas na coluna status.
