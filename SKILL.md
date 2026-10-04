---
name: soc-triage-report
description: Produz o relatorio padronizado de triagem de SOC sobre logs do Splunk, com enriquecimento MITRE ATT&CK e VirusTotal, em formato fixo e comparavel entre execucoes. Use SEMPRE que o pedido envolver analisar logs, investigar um index ou sourcetype do Splunk, verificar se houve ataque ou comprometimento, triagem de alerta, caca a ameacas, analise de incidente, ou quando o prompt mencionar um scenario_id. Use tambem em pedidos curtos e informais como "da uma olhada nesses logs", "tem algo suspeito aqui?" ou "analisa esse index". Esta skill define o formato de saida obrigatorio; nao produza analise de seguranca fora dela.
---

# Relatório de triagem de SOC

Este relatório alimenta uma pesquisa que compara execuções entre si. Duas coisas importam
igualmente: a qualidade da análise e a **comparabilidade**. Um relatório excelente em
formato livre não serve, porque não entra na série de dados.

Produza sempre as mesmas seções, na mesma ordem, com os mesmos rótulos — inclusive quando
não houver nada a reportar numa delas. Seção vazia se escreve "Nenhum", não se omite.

## Regra zero: não inventar

A varredura é inicial; um humano revisa depois. Um falso positivo bem documentado custa
minutos. Uma evidência inventada invalida a execução inteira como dado de pesquisa.

- Nunca cite hash, IP, domínio, host, usuário, caminho ou linha de comando que você não
  tenha visto literalmente num resultado de ferramenta.
- Nunca invente contagem de eventos, timestamp ou veredito de VirusTotal.
- Se não consultou o VirusTotal para um artefato, escreva "não consultado". Não escreva
  "desconhecido" para parecer que consultou.
- Consulta SPL que falhou ou voltou vazia vai em **Limitações**.
- "Não houve ataque", com justificativa, é um resultado válido. Não force detecção porque
  o cenário parece ser um teste.

## Antes de escrever

1. **Escopo.** Confirme index, sourcetype e janela. Se não vieram no prompt, descubra o
   que existe e registre a escolha.
2. **Reconhecimento amplo.** Volume e forma dos dados antes de filtrar. Distribuição por
   host, usuário, event code, ação. Saiba o que é normal neste conjunto.
3. **Anomalia, não assinatura.** Horário atípico, contas novas, parent/child de processo
   incomum, falhas de autenticação em sequência, destinos de rede raros, execução a partir
   de diretório gravável.
4. **Pivô.** De cada indício, pivote por host, conta e tempo. Monte a linha do tempo antes
   de nomear o ataque.
5. **Enriqueça.** Hashes, IPs, domínios e URLs observados vão ao VirusTotal. Registre a
   contagem de engines, não só o veredito: 1/70 e 45/70 são casos diferentes.
6. **Confirme no MITRE.** Consulte o MCP para ID e nome exatos. Não escreva IDs de memória.
7. **Tente derrubar.** Liste as explicações benignas e ataque-as com dados. As que
   sobreviverem vão para Limitações.

---

## Formato de saída

Copie a estrutura abaixo exatamente. Títulos em português, numeração das quatro seções
centrais preservada.

````markdown
# Triagem — <SCENARIO_ID> — <YYYY-MM-DD>

**Veredito:** ATAQUE DETECTADO | SEM ATAQUE
**Confiança:** alta | média | baixa  ·  **Severidade:** crítica | alta | média | baixa | informativa
**Resumo:** <uma frase, estilo título de ticket>

**Escopo analisado:** index=<...> sourcetype=<...> · earliest=<...> latest=<...> · <N> eventos · hosts: <...> · contas: <...>

---

## 1. Descrição do que foi encontrado

<Prosa, 3 a 15 frases. O que aconteceu, em qual host e conta, em que janela, em que ordem,
e por que isso é — ou não é — malicioso. Não repita a lista de TTPs aqui.>

## 2. TTPs identificadas

| ID | Técnica | Tática | Confiança | Evidência | Alerta disparou? |
|---|---|---|---|---|---|
| T1110.001 | Password Guessing | Credential Access | alta | POC-01, POC-02 | não |

<Uma linha por técnica. A coluna Evidência referencia os POC da seção 3 — toda TTP precisa
de pelo menos um. Técnica sem evidência correspondente não entra na tabela.
Use a sub-técnica quando a evidência permitir: se você identificou o interpretador, é
T1059.001, não T1059. Se nenhuma técnica foi observada, escreva "Nenhuma".>

**Justificativa por técnica:**
- **T1110.001** — <uma ou duas frases ligando a evidência concreta à técnica>

## 3. Proof of Compromise

### POC-01 — <tipo: ip-address | file-hash | command-line | ...>
- **Valor:** `<o artefato>`
- **Origem:** index=<...> sourcetype=<...>
- **Ocorrências:** <N> · **Primeira:** <ISO> · **Última:** <ISO>
- **Host / conta:** <...>
- **SPL:** `<consulta que isola exatamente este artefato, executável sem edição>`
- **Evento bruto:**
  ```
  <trecho literal do log, copiado, não parafraseado>
  ```
- **VirusTotal:** <N>/<M> engines maliciosos · famílias: <...> · <link>
  — ou — `não consultado`
- **Significado:** <por que isso prova ou refuta o comprometimento>

<Repita para POC-02, POC-03... Se não houver nenhum, escreva "Nenhum artefato de
comprometimento foi identificado" e explique brevemente o que foi procurado.>

## 4. Passos para reprodução

| # | Objetivo | Ferramenta | Ação | Resultado esperado |
|---|---|---|---|---|
| 1 | <o que este passo estabelece> | Splunk | `<SPL literal>` | <o que o analista deve ver> |

<A cadeia precisa ser auto-suficiente: começa da busca ampla e termina na conclusão.>

---

## Limitações

- <O que não foi possível verificar: fonte ausente, consulta que falhou, artefato não
  enriquecido, hipótese não descartada. Se realmente não houver nenhuma, escreva "Nenhuma"
  — mas pense duas vezes antes.>

## Ações recomendadas

1. **[imediato]** <...>
2. **[curto prazo]** <...>

---

## Métricas do teste

```json
{
  "scenario_id": "<...>",
  "attack_detected": true,
  "confidence": "alta",
  "severity": "alta",
  "ttp_count": 4,
  "ttp_ids": ["T1110.001", "T1078", "T1059.004", "T1105"],
  "poc_count": 3,
  "repro_steps": 6,
  "limitations_count": 2,
  "splunk_queries_run": 11,
  "virustotal_lookups": 3,
  "mitre_lookups": 4,
  "tempo_total": "PREENCHER",
  "tokens": "PREENCHER",
  "custo_usd": "PREENCHER"
}
```
````

---

## Sobre os três campos "PREENCHER"

**Não estime tempo, tokens nem custo.** Você não tem como medir isso de dentro da sessão;
qualquer número que você escrever é inventado e contamina a série de dados da pesquisa.
Deixe literalmente a string `PREENCHER` nos três campos.

Logo após o relatório, escreva uma linha única:

> Rode `/cost` para preencher tempo, tokens e custo no bloco de métricas.

Os demais campos do bloco você preenche, porque são contagens do seu próprio trabalho e
você as conhece: número de consultas SPL executadas, lookups no VirusTotal, consultas ao
MITRE, e os totais das seções acima.

## Erros comuns

- Mapear uma tática inteira quando a evidência só sustenta uma técnica.
- Escrever `T1059` quando a evidência identifica o interpretador.
- Copiar o veredito do VirusTotal sem a contagem de engines.
- Repetir a mesma informação na Descrição e em cada justificativa de TTP.
- SPL com placeholder (`index=<seu_index>`). Toda consulta sai pronta para colar.
- Concluir a partir de um único evento, havendo mais de uma fonte disponível.
- Omitir uma seção porque ficou vazia. Escreva "Nenhum".
