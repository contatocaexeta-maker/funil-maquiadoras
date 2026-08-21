# Funil Maquiadoras — GreenHub

Formulário de captação e qualificação de leads da GreenHub para a campanha de
**tráfego pago para maquiadoras profissionais**.

Derivado do `funil-shark-v2` (biomédica injetora). **Todas as automações são as
mesmas** — mesmo GTM, mesmo pixel, mesmo CRM, mesmas notificações. O que mudou
foi copy da capa, faixas de faturamento, paleta e link do Calendly.

Página única, sem build: `index.html` é o arquivo que vai pro ar.

**No ar:** https://funil-maquiadoras.vercel.app (Vercel, conta `greenhub1`, deploy
de produção em 21/08/2026). `capas.html` e este README ficam de fora do deploy
pelo `.vercelignore`.

## Fluxo

```
capa → nome → @ do Instagram → WhatsApp → faturamento → investimento → agendamento
```

Todo mundo responde as duas perguntas. **Quem decide o desfecho é a resposta do
investimento**; o faturamento só separa A de B.

| Nível | Regra | Para onde vai |
|---|---|---|
| **A** | toparia investir **e** fatura acima de 5k | tela de agendamento (Calendly) |
| **B** | toparia investir, mas fatura até 5k | tela de agendamento (Calendly) |
| **C** | "não é o meu momento" (qualquer faturamento) | tela de obrigado |

A regra vive na função `nivelDoLead()` e na constante `FATURAMENTO_NIVEL_B` no
topo do `<script>`.

## O que mudou em relação ao funil de biomédica

| | Biomédica | Maquiadora |
|---|---|---|
| Faixas de faturamento | 100k / 50-100k / 30-50k / até 30k | 20k / 10-20k / 5-10k / até 5k |
| Corte do Nível B | `ate_30k` | `ate_5k` |
| Recomendação de investimento | R$ 35/dia (~R$ 1.000/mês) | **igual** |
| Paleta | marrom `#7A5747` | rosé `#8E4A63` |
| Calendly | `/30min` | `/especialistas-em-mamoplastia` |
| GTM, pixel, CRM, CallMeBot, Telegram | — | **idênticos** |

> ⚠️ O evento do Calendly em uso é o `especialistas-em-mamoplastia`. É o nome que
> a maquiadora vê na tela final. Se for criar um evento próprio, troque só a
> constante `CALENDLY_URL`.

## Arquivos

| Arquivo | Pra que serve |
|---|---|
| `index.html` | **produção** — CRM + GTM/pixel + notificação de WhatsApp/Telegram, tudo ligado |
| `teste-debug.html` | painel no canto mostrando cada evento do dataLayer e cada chamada do pixel ao vivo (sem CRM, sem notificação) |
| `teste-gtm.html` | com GTM, pra usar o Preview do Tag Assistant (sem CRM, sem notificação) |
| `teste.html` | só o formulário, sem rastreio nenhum |

Os três arquivos de teste são **gerados a partir do `index.html`** — ao mexer no
formulário, mexa só no `index.html` e regere os outros.

## Rodar local

```bash
python3 -m http.server 8123
```

Depois `http://localhost:8123/teste-debug.html` (ou `index.html` pra testar com
CRM e WhatsApp de verdade — cuidado, cria lead).

## Rastreio

**GTM:** `GTM-K5K5SFP4` (conta GREEN SHARK) · **Pixel Meta:** `1001727392691930`

Os mesmos do funil de biomédica — nenhuma tag nova precisa ser criada no GTM.

O pixel **não** está no HTML: ele entra como tag base no GTM (All Pages), e cada
evento do dataLayer abaixo vira uma tag `trackCustom`.

| Evento no dataLayer | Evento no Meta | Quando |
|---|---|---|
| `iniciou_formulario` | `Etapa1_IniciouForm` | clicou no CTA da capa |
| `colocou_nome` | `Etapa2_Nome` | nome validado |
| `colocou_instagram` | `Etapa3_Instagram` | @ preenchido |
| `colocou_telefone` | `Etapa4_WhatsApp` | número validado |
| `colocou_faturamento` | `Etapa5_Faturamento` | 1ª pergunta (param `faturamento`) |
| `colocou_investimento` | `Etapa6_Investimento` | 2ª pergunta (param `investimento`) |
| `lead_nivel_a/b/c` | `LeadNivelA/B/C` | classificação final (params `nivel`, `faturamento`) |
| `visualizou_calendario` | `VisualizouCalendario` | o calendário apareceu de fato na tela |
| `agendou_reuniao` | — | o Calendly avisou que a reunião foi marcada |
| `lead_classificado` | — | genérico, com `nivel` como parâmetro |

Como os dois funis mandam os mesmos eventos pro mesmo pixel, **separe os
relatórios por URL** (a campanha de maquiadora aponta pra este domínio) ou crie
um parâmetro de origem se quiser separar dentro do próprio evento.

## CRM

Webhook do CRM (Green Hub), **mesmo tenant e mesmo pipeline** do funil de
biomédica, disparado em três momentos: depois do telefone (cria o card), depois
do faturamento (grava a faixa mesmo se ela largar na pergunta seguinte) e no
final (completa com o investimento). A dedupe é por telefone.

Campos enviados: `nome_e_sobrenome`, `instagram`, `whatsapp` (número),
`qual_seu_faturamento_medio_mensal`, `investiria_1000_reais_em_anuncio`.

Os nomes dos campos são os do CRM já existente — não renomeie, senão o card para
de receber a resposta.

## Notificações

- **WhatsApp** via CallMeBot, pros dois números do time (mesmos do funil atual).
- **Telegram** via Edge Function do Supabase (`notify-telegram`), com o token
  guardado no servidor.

Ambas disparam a cada lead fechado, com o nível (A/B qualificado, C
desqualificado) no topo da mensagem.

## Calendly

Embed inline na tela final (`CALENDLY_URL`), com `name` e `a1` (WhatsApp)
preenchidos pela query string — o objeto `prefill` do widget não chega no iframe.

Dois detalhes que já custaram tempo, não mexa sem saber:

- o script do Calendly **não** aplica classe no container; a altura vive na nossa
  `.calendly-box`, senão o iframe colapsa pra ~150px;
- **não** dá pra pré-carregar o iframe fora da tela: o navegador não renderiza
  iframe escondido e o Calendly aparece em branco ao ser revelado.

Se o Calendly não der sinal em 12s, a caixa vira um botão "agendar em outra aba".

## Capa

Ângulo escolhido: **ticket alto / sem desconto** (a versão 02 das cinco que estão
em `capas.html`). Os bullets são `white-space: nowrap` — cada um tem que caber em
327px a 375px de viewport, o que na prática dá **até ~30 caracteres**. Ao trocar a
copy, meça antes de subir.

```
💄 Chega de noiva pedindo desconto: quem chega já quer o [pacote fechado]

💎 Noiva de pacote fechado
👑 Debutante, formanda, madrinha     ← deixa claro que não é só noiva
🔥 Leads que não pedem desconto
✅ Agenda cheia sem indicação

           QUERO NOIVA QUE FECHA PACOTE →
```

`capas.html` guarda as cinco versões renderizadas na capa real, com a contagem de
caracteres de cada bullet.

## Pendências

- [ ] apontar a campanha pra https://funil-maquiadoras.vercel.app
- [ ] decidir se cria um evento próprio no Calendly em vez do de mamoplastia
- [ ] o rodapé da capa ainda é o genérico ("QUE JÁ ATENDEM CLIENTES") — pode
      apertar pra "QUE JÁ FECHAM CONTRATO DE NOIVA OU FORMATURA"
- [ ] conferir se a 1ª pergunta personalizada do evento do Calendly é o telefone
      (é onde o `a1` cai)
