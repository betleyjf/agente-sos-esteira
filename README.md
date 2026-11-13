# 👑 Agente Autônomo SOS — Esteira Inteligente de Conteúdo Multicanal

> **Case Real de Automação com IA & No-Code / Low-Code**  
> Desenvolvido por **Betley Ferreira** ([LinkedIn](https://linkedin.com) • [GitHub](https://github.com/betleyjf))  
> Projeto: Campanha de Lançamento do Álbum *"Da Sobrevivência à Soberania"* — Grupo **SOS** (`@sos3oficial`)

---

## 🎯 Visão Geral do Projeto

Este projeto é uma **esteira de inteligência e distribuição de conteúdo 100% autônoma**, construída para orquestrar a campanha de retorno do lendário grupo de rap de Belo Horizonte **SOS**, após 20 anos de hiato histórico.

O sistema elimina o trabalho manual de agendamento e adaptação de conteúdo, conectando um banco de dados relacional de copywriting à inteligência generativa multimodal e distribuindo mídias dinâmicas (Fotos e Vídeos) para **Instagram Feed, Instagram Reels e YouTube Shorts** com base em regras de roteamento inteligente.

---

## 🏗️ Arquitetura da Solução

A esteira foi projetada sob o princípio de **arquitetura desacoplada**: a inteligência e as regras de negócio rodam no **n8n local (Docker)**, enquanto a entrega segura para as APIs das redes sociais roda no **Make.com** via webhooks assinados.

```
                  ┌──────────────────────────────────────────────┐
                  │                 NOTION DATABASE              │
                  │   Calendário de Copywriting, Mídias e Status │
                  └──────────────────────┬───────────────────────┘
                                         │
                                         ▼ (Query: Status = 'Em andamento')
                  ┌──────────────────────────────────────────────┐
                  │              n8n (Self-Hosted Docker)        │
                  │   - Orquestrador de Fluxo e Agendamento      │
                  │   - Limiter & Filas de Prioridade            │
                  └──────────────┬───────────────────────────────┘
                                 │
                 ┌───────────────┴───────────────┐
                 ▼                               ▼
  ┌─────────────────────────────┐  ┌────────────────────────────┐
  │   GOOGLE GEMINI 1.5/3.0     │  │     TELEGRAM BOT API       │
  │   Gera Chamadas de Stories  │  │   Alerta em Tempo Real     │
  │   Contextuais da Narrativa  │  │   no Celular do Operador   │
  └──────────────┬──────────────┘  └────────────────────────────┘
                 │
                 ▼ (Atualiza Notion: Status ➔ 'Concluído')
  ┌─────────────────────────────────────────────────────────────┐
  │           HTTP REQUEST / WEBHOOK PAYLOAD DINÂMICO           │
  │     { media_url, caption, content_type: 'Foto' | 'Vídeo' }   │
  └──────────────────────────────┬──────────────────────────────┘
                                 │
                                 ▼
                  ┌──────────────────────────────────────────────┐
                  │              MAKE.COM (ROUTER CORE)          │
                  │          Roteamento por Tipo de Conteúdo     │
                  └──────────────┬────────────────┬──────────────┘
                                 │                │
            [Se content_type = 'Foto']     [Se content_type = 'Vídeo']
                                 │                │
                                 ▼                ▼
                    ┌─────────────────────────┐  ┌─────────────────────────┐
                    │ INSTAGRAM FOR BUSINESS  │  │   HTTP STREAM GET       │
                    │ Publica no Feed (Foto)  │  │   Download do Vídeo HD  │
                    └─────────────────────────┘  └───────────┬─────────────┘
                                                             │
                                             ┌───────────────┴───────────────┐
                                             ▼                               ▼
                              ┌─────────────────────────┐     ┌─────────────────────────┐
                              │ INSTAGRAM REELS (API)   │     │ YOUTUBE DATA API v3     │
                              │ Publica Reel com Áudio  │     │ Upload de YouTube Shorts│
                              └─────────────────────────┘     └─────────────────────────┘
```

---

## 💡 Decisões de Engenharia & Boas Práticas

### 1. Desacoplamento de Responsabilidades (n8n + Make)
* **Por que não postar direto no Instagram pelo n8n?**  
  A Graph API da Meta exige renovação periódica de tokens OAuth e aprovação estrita de aplicativos. Ao utilizar o Make.com como gateway de mensageria para a Meta e Google, garantimos tokens corporativos perpétuos e risco zero de interrupção da campanha por expiração de credenciais.
* **O n8n atua como cérebro:** Consulta o banco, orquestra o LLM, trata os dados, notifica a equipe e aciona o webhook.

### 2. Roteamento Inteligente (Content Router)
* O sistema não força o mesmo formato em todas as redes. 
* Se a linha no Notion for marcada como **`Foto`**, o webhook direciona exclusivamente para o Feed do Instagram.
* Se for marcada como **`Vídeo`**, a esteira baixa o arquivo binário em alta resolução e distribui simultaneamente como **Reel no Instagram** e **Shorts no YouTube**, respeitando os limites rígidos de metadados de cada plataforma (ex: títulos de até 100 caracteres no YouTube).

### 3. Integração de IA Multimodal (Google Gemini)
* Cada post aprovado passa pelo modelo generativo com um prompt especialista em rap nacional.
* O modelo analisa a letra, o tom da música e o conceito do post para **gerar ganchos e chamadas exclusivas para Stories**, salvando o resultado de volta no Notion de forma assíncrona.

### 4. Observabilidade & Feedback em Tempo Real (Telegram)
* A cada execução bem-sucedida, o bot do Telegram envia um resumo executivo diretamente para o smartphone do gestor da campanha com o turno, a faixa e os stories gerados pela IA.

---

## 🛠️ Stack Tecnológica

| Camada | Ferramenta / Serviço | Função |
|---|---|---|
| **Orquestração** | `n8n` (Docker Self-Hosted) | Motor principal de fluxos e regras de negócio |
| **Inteligência Artificial** | `Google Gemini 1.5/3.0 Flash` | Geração de copywriting derivado e chamadas de stories |
| **Banco de Dados** | `Notion API` | CMS relacional com status, datas, mídias e cópias |
| **Distribuição Social** | `Make.com` | Roteador e conexão segura com Meta & Google APIs |
| **Redes Finais** | `Instagram Graph API` & `YouTube Data API v3` | Publicação automática no Feed, Reels e Shorts |
| **Notificações** | `Telegram Bot API` | Alertas de execução e monitoramento em tempo real |
| **Storage de Mídia** | `Google Drive Direct Stream` | Hospedagem de imagens e vídeos de alta fidelidade |

---

## 📦 Como Reproduzir / Estrutura dos Arquivos

Neste repositório você encontra:

* 📄 [`workflow_n8n_agente_sos.json`](./workflow_n8n_agente_sos.json): Exportação completa do workflow do n8n pronto para importação (com credenciais sanitizadas).
* 📄 [`AGENT.md`](./AGENT.md): System prompt, persona e diretrizes de tom de voz do Agente de IA.
* 📄 [`Projetosos.md`](./Projetosos.md): O dossiê completo da campanha, história do grupo e os 3 Atos narrativos.
* 📄 [`POSTS_PRE_LANCAMENTO.md`](./POSTS_PRE_LANCAMENTO.md): Estrutura de conteúdo diário (12h e 17h) da campanha.

---

## 👤 Autor

**Betley Ferreira**  
*Copywriter & AI Agents Builder*  
Especialista em automações no-code/low-code, orquestração de LLMs e arquitetura de agentes autônomos.  
* Transição de carreira acelerada via formação **DIO (Digital Innovation One)**.
