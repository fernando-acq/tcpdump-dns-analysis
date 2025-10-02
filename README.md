# 🔍 tcpdump DNS Analysis

Análise de falha de comunicação na porta DNS (UDP 53) utilizando tcpdump para diagnóstico e resolução de problemas de rede.

## 📋 Descrição

Este projeto documenta a análise de um incidente de rede onde usuários não conseguiam acessar sites devido a uma falha no serviço DNS. Utilizando a ferramenta **tcpdump**, foi possível capturar e analisar o tráfego de rede, identificando que a porta UDP 53 estava inacessível, impedindo a resolução de nomes de domínio.

O relatório demonstra competências em diagnóstico de rede, interpretação de logs de tráfego, análise de protocolos DNS e ICMP, além de propor medidas efetivas de resolução e prevenção.

## 🎯 Objetivos do Projeto

- Diagnosticar falhas críticas em serviços de rede
- Utilizar tcpdump para captura e análise de tráfego
- Identificar problemas de comunicação DNS
- Interpretar mensagens ICMP de erro
- Propor soluções práticas de mitigação

## 🛠️ Tecnologias Utilizadas

- **tcpdump** - Captura de pacotes em linha de comando
- **DNS (Domain Name System)** - Resolução de nomes
- **UDP (User Datagram Protocol)** - Protocolo de transporte
- **ICMP** - Mensagens de controle e erro
- Análise de protocolos de rede

## 📁 Estrutura do Projeto

```
tcpdump-dns-analysis/
│
├── README.md                    # Documentação do projeto
├── incident_report.md           # Relatório completo do incidente
├── tcpdump_guide.md             # Guia prático de uso do tcpdump
└── dns_troubleshooting.md       # Guia de resolução de problemas DNS
```

## 🚨 Resumo do Incidente

### Problema Identificado

**Falha de comunicação na porta UDP 53 (DNS)**

Durante a tentativa de acessar o website "www.yummyrecipesforme.com", foi identificado um erro de comunicação no protocolo UDP, indicando que o pacote não pôde ser entregue à porta 53 do servidor DNS.

### Sintomas Observados

- Usuários não conseguiam acessar websites
- Mensagem de erro: "porta de destino inalcançável"
- Timeout no carregamento de páginas
- Falha na resolução de nomes de domínio

### Mensagem de Erro ICMP

```
ICMP 203.0.113.2 udp port 53 unreachable
```

Esta mensagem confirma que a porta 53 (DNS) está inacessível no servidor.

## 🔍 Análise Técnica

### Log Capturado pelo tcpdump

```
13:24:32.192571 IP 192.51.100.15.52444 > 203.0.113.2.domain: 35084+ A? yummyrecipesforme.com. (24)
13:24:36.098564 IP 203.0.113.2 > 192.51.100.15: ICMP 203.0.113.2 udp port 53 unreachable length 254
```

### Interpretação do Log

**Linha 1:**
- **Timestamp:** 13:24:32.192571
- **Origem:** 192.51.100.15 (cliente) porta 52444
- **Destino:** 203.0.113.2 porta domain (53)
- **Tipo de consulta:** A? (registro A - endereço IPv4)
- **Domínio:** yummyrecipesforme.com
- **Tamanho:** 24 bytes

**Linha 2:**
- **Timestamp:** 13:24:36.098564 (4 segundos depois)
- **Origem:** 203.0.113.2 (servidor)
- **Destino:** 192.51.100.15 (cliente)
- **Tipo:** ICMP (mensagem de erro)
- **Erro:** udp port 53 unreachable
- **Tamanho:** 254 bytes

### Análise do Protocolo DNS

**Funcionamento Normal do DNS:**
1. Cliente envia consulta UDP para porta 53
2. Servidor DNS processa a consulta
3. Servidor responde com endereço IP do domínio
4. Cliente pode conectar ao site

**O que ocorreu no incidente:**
1. Cliente enviou consulta UDP para porta 53
2. Porta 53 estava inacessível
3. Servidor respondeu com mensagem ICMP de erro
4. Resolução de nome falhou

## 🔎 Possíveis Causas

### 1. Falha no Servidor DNS

O servidor DNS pode estar:
- Inoperante (offline)
- Sobrecarregado
- Com serviço DNS parado
- Com configuração incorreta

### 2. Bloqueio no Firewall

Regras de firewall podem estar:
- Bloqueando tráfego de saída na porta 53
- Bloqueando tráfego de entrada na porta 53
- Com configuração incorreta após atualização

### 3. Ataque DoS ao Serviço DNS

Possível ataque de negação de serviço:
- Saturação da porta 53
- Esgotamento de recursos do servidor DNS
- Bloqueio proposital do serviço

## 🛡️ Medidas Recomendadas

### Ações Imediatas

**1. Verificar Configurações do Firewall**
```bash
# Linux - verificar regras do firewall
sudo iptables -L -n | grep 53

# Verificar se porta 53 está aberta
sudo netstat -tulpn | grep :53
```

**2. Testar Disponibilidade do Servidor DNS**
```bash
# Testar com ping
ping 203.0.113.2

# Testar resolução DNS
nslookup yummyrecipesforme.com 203.0.113.2

# Testar com dig
dig @203.0.113.2 yummyrecipesforme.com
```

**3. Substituir Temporariamente o DNS**
```bash
# Usar DNS público do Google
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

# Ou Cloudflare
echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf
```

### Ações de Longo Prazo

**1. Monitorar Tráfego DNS**
- Implementar alertas para falhas DNS
- Monitorar volume de consultas
- Detectar padrões anômalos

**2. Implementar DNS Redundante**
- Configurar servidores DNS secundários
- Usar balanceamento de carga
- Ter failover automático

**3. Proteção Contra Ataques**
- Rate limiting para consultas DNS
- Filtros contra DNS amplification
- Proteção DDoS para serviço DNS

## 📊 Comandos tcpdump Úteis

### Captura Básica
```bash
# Capturar todo o tráfego
sudo tcpdump -i eth0

# Capturar apenas DNS (porta 53)
sudo tcpdump -i eth0 port 53

# Capturar e mostrar em formato legível
sudo tcpdump -i eth0 -n port 53
```

### Filtros Específicos
```bash
# Capturar apenas UDP DNS
sudo tcpdump -i eth0 udp port 53

# Capturar de/para IP específico
sudo tcpdump -i eth0 host 203.0.113.2

# Capturar e salvar em arquivo
sudo tcpdump -i eth0 port 53 -w dns_capture.pcap
```

### Análise Detalhada
```bash
# Mostrar conteúdo dos pacotes em hexadecimal
sudo tcpdump -i eth0 -X port 53

# Mostrar timestamp absoluto
sudo tcpdump -i eth0 -tttt port 53

# Não resolver nomes (mais rápido)
sudo tcpdump -i eth0 -n port 53
```

## 🔒 Aplicações em Cibersegurança

Este projeto demonstra competências essenciais para:

- **Network Operations:** Diagnóstico de problemas de rede
- **Incident Response:** Análise de falhas de serviço
- **Security Monitoring:** Detecção de anomalias em DNS
- **Troubleshooting:** Resolução de problemas de conectividade
- **Infrastructure Security:** Proteção de serviços críticos

## 📚 Conceitos Aplicados

- Análise de tráfego de rede com tcpdump
- Protocolo DNS e resolução de nomes
- Protocolo UDP e comunicação sem conexão
- Mensagens ICMP de erro
- Diagnóstico de rede em Linux
- Técnicas de troubleshooting

## 🎓 Aprendizados

- Uso prático do tcpdump para diagnóstico
- Interpretação de logs de tráfego de rede
- Análise de protocolos DNS, UDP e ICMP
- Identificação de falhas em serviços críticos
- Elaboração de relatórios técnicos
- Proposta de soluções práticas

## 🤝 Contribuições

Sugestões e melhorias são bem-vindas! Sinta-se à vontade para abrir issues ou pull requests.

## 📝 Licença

Este projeto está sob a licença MIT. Veja o arquivo LICENSE para mais detalhes.

## 👤 Autor

**Fernando Acquesta**
- GitHub: [@fernando-acq](https://github.com/fernando-acq)
- LinkedIn: [Fernando Acquesta](https://www.linkedin.com/in/fernando-acquesta-cybersecurity)

---

⭐ Se este projeto ajudou você a entender melhor análise de rede e diagnóstico DNS, considere dar uma estrela no repositório!
