# Arquitetura do Projeto: Agente de Cobrança Inteligente

## Tratamento de Dados Iniciais

| Atividade                                           | Estimativa | Observação                                               |
|-----------------------------------------------------|------------|----------------------------------------------------------|
| Webhook (entrada)                                   | 10min      | Captura da mensagem recebida.                           |
| If - FromMe                                         | 10min      | Verifica se a mensagem é do cliente.                    |
| If - FromAPI (evolution)                            | 10min      | Impede a continuidade se for da API.                    |
| Redis Get (verificar bloqueio 15min)                | 10min      | Coloca uma pausa do agente se tiver  intervenção humana |
| Set campos iniciais                                 | 20min      | Prepara os dados para o fluxo.                          |
| Verificação se já é cliente com situação resolvida  | 20min      | Para direcionar corretamente.                           |

**Subtotal:** **1h**

---

## Consulta e Cadastro no Banco de Dados

| Atividade                   | Estimativa | Observação                                           |
|-----------------------------|------------|------------------------------------------------------|
| Verificar cliente           | 10min      | Consulta básica pelo número.                        |
| Se WhatsApp já existe       | 10min      | Evita duplicidade.                                  |
| Criar novo lead             | 10min      | Caso não exista.                                    |
| Merge dos dados             | 10min      | Unifica os dados para o fluxo.                      |

**Subtotal:** **40min**

---

## Tratamento do Tipo de Mensagem

| Atividade                       | Estimativa | Observação                                                   |
|---------------------------------|------------|--------------------------------------------------------------|
| Verificação do tipo (If)        | 10min      | Diferencia texto, áudio e imagem.                           |
| Texto → OpenAI                  | 20min      | Envia a mensagem para interpretação.                        |
| Áudio → Transcrição + OpenAI    | 20min      | Converte o áudio antes de enviar ao agente.                 |
| Imagem → Conversão ou Ignorar   | 10min      | Pode ser tratado ou apenas armazenado.                      |

**Subtotal:** **1h**

---

## Acúmulo de Mensagens Múltiplas

| Atividade                        | Estimativa | Observação                                                   |
|----------------------------------|------------|--------------------------------------------------------------|
| Wait                             | 10min      | Dá tempo para acumular mensagens.                           |
| Lista de mensagens do Redis      | 10min      | Recupera anteriores do mesmo número.                        |
| Merge tipo conversa              | 20min      | Junta as mensagens acumuladas.                              |
| Condicional (se tem mensagens)   | 10min      | Segue ou ignora.                                            |
| Limpa mensagens do Redis         | 10min      | Evita duplicidade futura.                                   |

**Subtotal:** **1h**

---

## Agente (Interpretação com IA)

| Atividade                         | Estimativa | Observação                                                    |
|-----------------------------------|------------|---------------------------------------------------------------|
| OpenAI ou (GROQ) com memória      | 30min      | Faz análise contextual da mensagem.                          |
| Supabase Vector Search (opcional) | 30min      | Personaliza com base em dados passados.                      |
| Parser Chain (OpenAI)             | 30min      | Classifica a intenção: pagar, reclamar, etc.                 |
| Construção do system prompt       | 6 horas    | Construir e ajustar o system prompt para que seja capaz de identificar e responder corretamente as mensagens |

**Subtotal:** **7h30**

---

## Resposta

| Atividade                         | Estimativa | Observação                                                       |
|-----------------------------------|------------|------------------------------------------------------------------|
| Segmentos (If ou Switch)          | 20min      | Direciona para o fluxo correto (pagamento, humano, etc).        |
| Loop/Replace/Formatar Resposta    | 20min      | Simula digitação, quebra em partes.                             |
| Espera (Wait) + "Digitando..."    | 10min      | Simula uma conversa natural.                                    |
| Envia Texto/Imagem                | 10min      | Retorno via WhatsApp.                                           |

**Subtotal:** **1h**

---

## Atualização de Banco de Dados

| Atividade                         | Estimativa | Observação                                                   |
|-----------------------------------|------------|--------------------------------------------------------------|
| Salvar status "Recebeu boleto"    | 10min      | Após o envio automático.                                    |
| Salvar status "Efetuou pagamento" | 10min      | Quando enviar comprovante.                                  |
| Atualiza Lead no Supabase         | 10min      | Atualiza o status.                                          |
| Condicional para histórico        | 10min      | Evita sobrescrever sem critério.                            |

**Subtotal:** **40min**

---

## Criação de Credenciais e Testes

### Estimativa para Criação de Credenciais

| Sistema/Ferramenta                | Estimativa | Observação                                               |
|-----------------------------------|------------|----------------------------------------------------------|
| WhatsApp API (360dialog ou Z-API) | 20min      | Geração de token e validação de número                   |
| Supabase                          | 15min      | Criação do projeto, configuração de tabelas e API Key    |
| Redis (Railway, Upstash, etc)     | 15min      | Criação da instância e obtenção da URL e senha           |
| OpenAI                            | 15min      | Obtenção de API Key e configuração no Make/n8n           |
| n8n                               | 15min      | Configuração de cenários e variáveis                     |
| Plataforma de Vetorização (opc.)  | 20min      | Criação de base vetorial no Supabase ou outro sistema    |

**Subtotal:** **1h40min**

---

### Estimativa de Testes e Ajustes

| Atividade                        | Estimativa | Observação                                             |
|----------------------------------|------------|--------------------------------------------------------|
| Testes de ponta a ponta          | 4h         | Simulação completa de interações e correções           |
| Ajustes finos em cada etapa      | 3h         | Refinamento do comportamento e melhoria de respostas   |

**Subtotal:** **7h**

## Tempo Total Estimado: **21h30min**
