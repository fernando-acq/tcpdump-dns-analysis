# 📘 Guia Prático de tcpdump

Referência completa para captura e análise de tráfego de rede usando tcpdump.

---

## 🚀 Instalação

### Linux (Debian/Ubuntu)
```bash
sudo apt update
sudo apt install tcpdump
```

### Linux (Red Hat/CentOS)
```bash
sudo yum install tcpdump
```

### Verificar Instalação
```bash
tcpdump --version
```

---

## 📡 Comandos Básicos

### Listar Interfaces de Rede
```bash
tcpdump -D
```

### Capturar em Interface Específica
```bash
sudo tcpdump -i eth0
```

### Parar Captura
```
Ctrl + C
```

---

## 🔍 Filtros Essenciais

### Filtros por Protocolo
```bash
tcpdump tcp                 # Apenas TCP
tcpdump udp                 # Apenas UDP
tcpdump icmp                # Apenas ICMP
```

### Filtros por Porta
```bash
tcpdump port 53             # Porta 53 (DNS)
tcpdump port 80             # Porta 80 (HTTP)
tcpdump port 443            # Porta 443 (HTTPS)
tcpdump udp port 53         # UDP na porta 53
```

### Filtros por Host
```bash
tcpdump host 192.168.1.1                # Tráfego de/para este IP
tcpdump src host 192.168.1.1            # Origem deste IP
tcpdump dst host 192.168.1.1            # Destino para este IP
```

### Filtros por Rede
```bash
tcpdump net 192.168.1.0/24              # Toda a rede
tcpdump src net 192.168.1.0/24          # Origem na rede
tcpdump dst net 192.168.1.0/24          # Destino na rede
```

---

## 🎯 Filtros Específicos para DNS

### Capturar Todo Tráfego DNS
```bash
sudo tcpdump -i eth0 port 53
```

### Capturar Apenas Consultas DNS
```bash
sudo tcpdump -i eth0 udp port 53
```

### Capturar DNS de/para IP Específico
```bash
sudo tcpdump -i eth0 host 203.0.113.2 and port 53
```

### Mostrar Conteúdo das Consultas DNS
```bash
sudo tcpdump -i eth0 -vvv port 53
```

---

## 📊 Opções de Saída

### Mostrar em Formato Legível
```bash
tcpdump -n                  # Não resolver nomes
tcpdump -nn                 # Não resolver nomes nem portas
```

### Mostrar Mais Detalhes
```bash
tcpdump -v                  # Verbose
tcpdump -vv                 # Mais verbose
tcpdump -vvv                # Máximo verbose
```

### Mostrar Conteúdo dos Pacotes
```bash
tcpdump -X                  # Hexadecimal e ASCII
tcpdump -XX                 # Hexadecimal incluindo cabeçalho ethernet
tcpdump -A                  # Apenas ASCII
```

### Timestamp
```bash
tcpdump -tttt               # Timestamp absoluto
tcpdump -t                  # Sem timestamp
```

---

## 💾 Salvar e Ler Capturas

### Salvar em Arquivo
```bash
sudo tcpdump -i eth0 -w captura.pcap
sudo tcpdump -i eth0 port 53 -w dns_capture.pcap
```

### Ler Arquivo Capturado
```bash
tcpdump -r captura.pcap
tcpdump -r captura.pcap port 53
```

### Limitar Tamanho da Captura
```bash
tcpdump -i eth0 -w captura.pcap -C 100     # Arquivos de 100MB
tcpdump -i eth0 -w captura.pcap -W 5       # Manter apenas 5 arquivos
```

---

## 🔬 Análise de Mensagens ICMP

### Capturar ICMP
```bash
sudo tcpdump -i eth0 icmp
```

### Capturar Mensagens "Unreachable"
```bash
sudo tcpdump -i eth0 'icmp[icmptype] == icmp-unreach'
```

### Capturar Ping (Echo Request/Reply)
```bash
sudo tcpdump -i eth0 'icmp[icmptype] == icmp-echo'
```

---

## 🧩 Operadores Lógicos

### AND
```bash
tcpdump host 192.168.1.1 and port 53
```

### OR
```bash
tcpdump port 80 or port 443
```

### NOT
```bash
tcpdump not port 22
```

### Combinações Complexas
```bash
tcpdump 'host 192.168.1.1 and (port 80 or port 443)'
tcpdump 'tcp and not port 22'
```

---

## 📝 Exemplos Práticos

### Diagnóstico de DNS

**Capturar consultas DNS falhadas:**
```bash
sudo tcpdump -i eth0 -n -vvv 'port 53' | grep -i unreachable
```

**Monitorar tráfego DNS em tempo real:**
```bash
sudo tcpdump -i eth0 -l port 53 | grep -i 'A?'
```

### Análise de Conectividade

**Verificar se pacotes chegam ao destino:**
```bash
sudo tcpdump -i eth0 host 203.0.113.2
```

**Ver tentativas de conexão TCP:**
```bash
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0'
```

### Troubleshooting de Rede

**Capturar tráfego HTTP:**
```bash
sudo tcpdump -i eth0 -A port 80
```

**Ver apenas cabeçalhos de requisições:**
```bash
sudo tcpdump -i eth0 -s 0 -A 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```

---

## 🎨 Interpretação de Output

### Formato Básico
```
timestamp protocolo origem > destino: flags dados
```

### Exemplo de Saída DNS
```
13:24:32.192571 IP 192.51.100.15.52444 > 203.0.113.2.domain: 35084+ A? yummyrecipesforme.com. (24)
```

**Componentes:**
- `13:24:32.192571` - Timestamp
- `IP` - Protocolo
- `192.51.100.15.52444` - IP origem e porta
- `203.0.113.2.domain` - IP destino e porta (domain = 53)
- `35084+` - ID da consulta DNS
- `A?` - Tipo de registro (A = IPv4)
- `yummyrecipesforme.com.` - Domínio consultado
- `(24)` - Tamanho do pacote

### Exemplo de Saída ICMP
```
13:24:36.098564 IP 203.0.113.2 > 192.51.100.15: ICMP 203.0.113.2 udp port 53 unreachable length 254
```

**Componentes:**
- `ICMP` - Tipo de mensagem
- `203.0.113.2 > 192.51.100.15` - Origem e destino
- `udp port 53 unreachable` - Mensagem de erro
- `length 254` - Tamanho do pacote

---

## 💡 Dicas Práticas

### Para Análise Eficiente

1. Use `-n` para não resolver nomes (mais rápido)
2. Use `-c` para limitar número de pacotes
3. Salve capturas longas em arquivo para análise posterior
4. Use filtros específicos para reduzir ruído
5. Combine com grep para buscar padrões

### Para Diagnóstico DNS

1. Capture tráfego na porta 53
2. Verifique se consultas estão sendo enviadas
3. Verifique se respostas estão chegando
4. Procure por mensagens ICMP de erro
5. Compare com servidor DNS alternativo

### Para Troubleshooting

1. Identifique primeiro qual protocolo está afetado
2. Use filtros para isolar o tráfego problemático
3. Salve capturas antes e depois de mudanças
4. Compare comportamento normal vs anômalo
5. Documente suas descobertas

---

## 🛠️ Comandos de Diagnóstico Complementares

### Testar DNS
```bash
nslookup dominio.com
dig dominio.com
host dominio.com
```

### Testar Conectividade
```bash
ping 203.0.113.2
traceroute 203.0.113.2
telnet 203.0.113.2 53
```

### Verificar Portas Abertas
```bash
netstat -tulpn | grep :53
ss -tulpn | grep :53
lsof -i :53
```

---

## 📚 Recursos Adicionais

- Manual: `man tcpdump`
- Site oficial: https://www.tcpdump.org/
- Wireshark (GUI): https://www.wireshark.org/

---

**Nota:** tcpdump requer privilégios de superusuário (sudo/root) para capturar pacotes na maioria das interfaces de rede.