# 📄 Relatório de Incidente de Cibersegurança - Falha DNS

**Tipo de Incidente:** Falha de Comunicação DNS (Porta 53)  
**Data/Hora:** 13:24  
**Ferramenta Utilizada:** tcpdump  
**Analista:** Fernando Acquesta

---

## 1. Resumo Executivo

Foi identificada uma falha crítica no serviço DNS que impediu usuários de acessar websites. A análise de tráfego com **tcpdump** revelou que a porta UDP 53 estava inacessível, resultando em mensagens ICMP de erro "udp port 53 unreachable". O incidente afetou múltiplos usuários que relataram timeouts ao tentar acessar o site "www.yummyrecipesforme.com".

---

## 2. Descrição do Problema

### Sintomas Relatados

- Usuários não conseguiam acessar o website "www.yummyrecipesforme.com"
- Mensagem de erro: "porta de destino inalcançável"
- Timeout após tempo de carregamento esgotar
- Falha generalizada na resolução de nomes de domínio

### Erro Identificado

**Mensagem ICMP:** "udp port 53 unreachable"

A porta 53 é crucial para o funcionamento da rede, pois é responsável pelo tráfego **DNS (Domain Name System)**, que traduz nomes de domínio legíveis por humanos em endereços IP utilizáveis por máquinas.

---

## 3. Análise dos Dados

### Timestamp do Incidente

**Data/Hora:** 13:24

### Log de Tráfego Capturado

```
13:24:32.192571 IP 192.51.100.15.52444 > 203.0.113.2.domain: 35084+ A? yummyrecipesforme.com. (24)
13:24:36.098564 IP 203.0.113.2 > 192.51.100.15: ICMP 203.0.113.2 udp port 53 unreachable length 254
```

### Interpretação Detalhada

**Linha 1: Consulta DNS do Cliente**
- **IP Origem:** 192.51.100.15 (cliente interno)
- **Porta Origem:** 52444 (porta aleatória do cliente)
- **IP Destino:** 203.0.113.2 (servidor DNS)
- **Porta Destino:** domain (porta 53)
- **Tipo de Consulta:** 35084+ A? (consulta de registro A)
- **Domínio Consultado:** yummyrecipesforme.com
- **Tamanho do Pacote:** 24 bytes

**Linha 2: Resposta de Erro ICMP**
- **IP Origem:** 203.0.113.2 (servidor)
- **IP Destino:** 192.51.100.15 (cliente)
- **Tipo de Mensagem:** ICMP (Internet Control Message Protocol)
- **Erro:** udp port 53 unreachable
- **Tamanho do Pacote:** 254 bytes
- **Tempo de Resposta:** ~4 segundos após consulta inicial

### Conclusão da Análise

O log confirma que:
1. Cliente tentou fazer consulta DNS legítima
2. Pacote foi enviado corretamente para porta 53
3. Servidor respondeu com mensagem ICMP de erro
4. Porta 53 está inacessível ou bloqueada

---

## 4. Causas Prováveis

### 4.1 Falha no Servidor DNS

**Descrição:** O servidor DNS pode estar inoperante ou sobrecarregado.

**Evidências:**
- Porta 53 não responde a consultas
- Mensagem ICMP indica serviço inacessível
- Múltiplos usuários afetados simultaneamente

**Possíveis Motivos:**
- Serviço DNS parado
- Servidor DNS offline
- Configuração incorreta
- Esgotamento de recursos

### 4.2 Bloqueio no Firewall

**Descrição:** Regra de firewall pode estar bloqueando tráfego DNS.

**Evidências:**
- Pacote UDP chega ao servidor
- Servidor responde com ICMP unreachable
- Bloqueio específico na porta 53

**Possíveis Motivos:**
- Atualização recente de regras de firewall
- Configuração incorreta de segurança
- Bloqueio acidental de tráfego DNS legítimo

### 4.3 Ataque DoS ao DNS

**Descrição:** Ataque de negação de serviço direcionado ao serviço DNS.

**Evidências:**
- Indisponibilidade repentina do serviço
- Afeta múltiplos usuários
- Porta crítica inacessível

**Possíveis Motivos:**
- Ataque de saturação (DNS flood)
- DNS amplification attack
- Tentativa de interromper operações

---

## 5. Impacto do Incidente

### Impacto Técnico

- Impossibilidade de resolução de nomes DNS
- Falha no acesso a websites externos
- Interrupção de serviços dependentes de DNS
- Potencial impacto em aplicações internas

### Impacto nos Negócios

- Interrupção de acesso a recursos online
- Perda de produtividade dos usuários
- Possível impacto em serviços ao cliente
- Potencial perda de receita

---

## 6. Medidas de Mitigação Implementadas

### 6.1 Verificação de Configurações do Firewall

**Ação:** Revisar regras de firewall relacionadas à porta 53

**Comandos Executados:**
```bash
sudo iptables -L -n | grep 53
sudo netstat -tulpn | grep :53
```

### 6.2 Teste de Disponibilidade do Servidor DNS

**Ação:** Verificar se servidor DNS está respondendo

**Comandos de Diagnóstico:**
```bash
ping 203.0.113.2
nslookup yummyrecipesforme.com 203.0.113.2
dig @203.0.113.2 yummyrecipesforme.com
```

### 6.3 Configuração de DNS Alternativo

**Ação:** Substituir temporariamente por DNS público

**DNS Público Configurado:**
```bash
# Google DNS
nameserver 8.8.8.8
nameserver 8.8.4.4

# Cloudflare DNS (alternativo)
nameserver 1.1.1.1
nameserver 1.0.0.1
```

**Resultado:** Acesso a websites restaurado temporariamente

---

## 7. Recomendações para Prevenção

### 7.1 Monitoramento Contínuo do Tráfego DNS

**Implementar:**
- Alertas automáticos para falhas DNS
- Monitoramento de volume de consultas
- Detecção de padrões anômalos
- Dashboard de saúde do DNS

**Ferramentas Sugeridas:**
- Nagios/Zabbix para monitoramento
- ELK Stack para análise de logs
- Grafana para visualização

### 7.2 Implementação de DNS Redundante

**Configurar:**
- Servidores DNS primário e secundário
- Balanceamento de carga entre servidores
- Failover automático em caso de falha
- DNS em localizações geograficamente distribuídas

### 7.3 Proteção Contra Ataques DNS

**Medidas de Segurança:**
- Rate limiting para consultas DNS
- Filtros contra DNS amplification attacks
- Proteção DDoS específica para DNS
- Lista de bloqueio de IPs maliciosos

**Configuração Sugerida:**
```bash
# Rate limiting com iptables
iptables -A INPUT -p udp --dport 53 -m limit --limit 25/s -j ACCEPT
iptables -A INPUT -p udp --dport 53 -j DROP
```

### 7.4 Documentação e Procedimentos

**Criar:**
- Runbook para incidentes DNS
- Procedimentos de escalação
- Lista de verificação para troubleshooting
- Contatos de suporte 24/7

---

## 8. Próximos Passos

### Imediato (24 horas)

1. Confirmar causa raiz do problema
2. Restaurar serviço DNS principal
3. Documentar configurações corretas
4. Comunicar resolução aos usuários

### Curto Prazo (1 semana)

1. Implementar monitoramento DNS
2. Configurar DNS secundário
3. Revisar regras de firewall
4. Realizar testes de failover

### Médio Prazo (1 mês)

1. Implementar proteções contra DDoS
2. Criar runbooks de incidentes
3. Treinar equipe em diagnóstico DNS
4. Estabelecer SLA para resolução DNS

---

## 9. Conclusão

O incidente foi diagnosticado como uma falha de comunicação com a porta UDP 53, essencial para resolução de nomes DNS. A evidência do log tcpdump confirma que o serviço DNS está inacessível.

**Causa Raiz Provável:**
- Falha no servidor DNS, ou
- Bloqueio por firewall, ou
- Ataque DoS ao serviço DNS

**Ação Imediata:** Restauração do serviço através de checagem de firewall e testes de disponibilidade do servidor.

**Ação de Longo Prazo:** Implementação de monitoramento contínuo e redundância para prevenir recorrências.

A prioridade é garantir a disponibilidade e integridade do serviço DNS, componente crítico para operações de rede.

---

**Relatório elaborado por:** Fernando Acquesta  
**Data de Análise:** Janeiro 2025  
**Ferramenta:** tcpdump