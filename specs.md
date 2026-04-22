# Definição do Ambiente para o TCC

## Disponibilização do Ambiente
Para criação do ambiente, instalação dos serviços e dos servidores MCP e para integração com os agentes, será configurada uma VPS com um domínio.

## Sistema Operacional
**Ubuntu Server**
*(Em ambientes corporativos, o padrão é o uso do Red Hat Enterprise System, mas como se trata de uma solução paga, para o desenvolvimento deste trabalho será utilizado o Ubuntu Server, que tem o suporte para todo o ambiente que será criado).*

## Ferramentas Utilizadas

### Splunk
Uma plataforma líder de mercado para coleta, indexação, busca e análise de grandes volumes de dados (logs) gerados por ativos de TI. No contexto de segurança, atua frequentemente como um SIEM (Security Information and Event Management). O projeto utilizará a versão gratuita (Splunk Free), que possui limitações de ingestão diária de dados, mas atende perfeitamente aos requisitos de armazenamento e monitoramento para o escopo deste laboratório.

- **MCP Utilizado:** [Splunk MCP Server](https://github.com/splunk/splunk-mcp-server2)

### MITRE ATT&CK
Uma base de conhecimento global e *framework* padronizado que mapeia as Táticas, Técnicas e Procedimentos (TTPs) utilizados em ataques cibernéticos reais. Ele categoriza o comportamento dos adversários e os atribui a grupos de ameaças conhecidos (APTs) e setores industriais. A utilização desta solução fornece uma linguagem comum para a defesa, permitindo correlacionar os eventos e anomalias detectados (via Splunk) com táticas de ataque catalogadas, facilitando a compreensão da cadeia do incidente.

- **MCP Utilizado:** [MITRE ATT&CK MCP Server](https://github.com/imouiche/complete-mitre-attack-mcp-server)

### VirusTotal
Um serviço de inteligência de ameaças (Threat Intelligence) que analisa arquivos, *hashes*, domínios, IPs e URLs, submetendo-os a dezenas de motores de antivírus e ferramentas de escaneamento. Com os artefatos suspeitos coletados, esta solução enriquece a análise ao verificar a reputação desses elementos, correlacionando-os com campanhas maliciosas conhecidas, táticas do MITRE ATT&CK e inteligência compartilhada pela comunidade global de segurança.

- **MCP Utilizado:** [VirusTotal MCP Server](https://github.com/BurtTheCoder/mcp-virustotal)

---

## Arquitetura e Integração do Ambiente

A infraestrutura do laboratório será centralizada em uma única máquina virtual operando com o sistema Ubuntu Server. O Splunk funcionará como o núcleo analítico do ambiente, recebendo e indexando os eventos gerados. 

A automação e o suporte à Análise e Resposta a Incidentes ocorrerão por meio da integração dos três MCPs (Model Context Protocols). O fluxo de trabalho integrado funcionará da seguinte forma: 

1. Quando uma atividade anômala for investigada, o agente utilizará o **MCP do Splunk** para realizar buscas diretas e extrair logs detalhados. 
2. A partir desses logs, artefatos suspeitos (como *hashes* de arquivos ou endereços IP) serão passados ao **MCP do VirusTotal** para enriquecimento de dados e verificação de reputação. 
3. Simultaneamente, o **MCP do MITRE ATT&CK** será acionado para classificar esses comportamentos maliciosos dentro do seu *framework*. 

Essa arquitetura permite que o modelo colete os dados brutos, avalie a periculosidade dos artefatos e mapeie a anatomia do ataque em uma única esteira de análise.
