# 🔧 Guia de Resolução de Problemas DNS

Guia completo para diagnosticar e resolver problemas relacionados ao serviço DNS.

---

## 🚨 Sintomas Comuns de Problemas DNS

- Sites não carregam (timeout)
- Mensagem "servidor não encontrado"
- Erro "porta de destino inalcançável"
- Lentidão ao acessar websites
- Aplicações não conseguem conectar a servidores

---

## 🔍 Diagnóstico Passo a Passo

### Etapa 1: Verificar Conectividade Básica

**Testar conexão com Internet:**
```bash
ping 8.8.8.8
```

Se funciona: problema é DNS  
Se não funciona: problema é conectividade de rede

### Etapa 2: Testar Resolução DNS

**Verificar se DNS está funcionando:**
```bash
nslookup google.com
```

**Saída normal:**
```
Server:         192.168.1.1
Address:        192.168.1.1#53

Non-authoritative answer:
Name:   google.com
Address: 142.250.185.78
```

**Saída com problema:**
```
;; connection timed out; no servers could be reached
```

### Etapa 3: Identificar o Servidor DNS em Uso

**Linux:**
```bash
cat /etc/resolv.conf
```

**Windows:**
```cmd
ipconfig /all
```

### Etapa 4: Testar Servidor DNS Específico

**Testar com dig:**
```bash
dig @8.8.8.8 google.com
```

**Testar com nslookup:**
```bash
nslookup google.com 8.8.8.8
```

---

## 🛠️ Comandos de Diagnóstico

### nslookup

**Uso básico:**
```bash
nslookup dominio.com
```

**Consultar servidor DNS específico:**
```bash
nslookup dominio.com 8.8.8.8
```

**Consultar tipo de registro específico:**
```bash
nslookup -type=MX dominio.com
nslookup -type=NS dominio.com
```

### dig

**Uso básico:**
```bash
dig dominio.com
```

**Consulta detalhada:**
```bash
dig dominio.com +trace
```

**Consultar servidor específico:**
```bash
dig @8.8.8.8 dominio.com
```

**Consultar tipo de registro:**
```bash
dig dominio.com MX
dig dominio.com NS
dig dominio.com TXT
```

### host

**Uso básico:**
```bash
host dominio.com
```

**Consulta reversa (IP para nome):**
```bash
host 8.8.8.8
```

---

## 🔧 Soluções Comuns

### Solução 1: Limpar Cache DNS

**Linux:**
```bash
sudo systemd-resolve --flush-caches
sudo resolvectl flush-caches
```

**Windows:**
```cmd
ipconfig /flushdns
```

**macOS:**
```bash
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

### Solução 2: Alterar Servidor DNS

**Temporário (Linux):**
```bash
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

**Permanente (Ubuntu/Debian):**

Editar `/etc/systemd/resolved.conf`:
```ini
[Resolve]
DNS=8.8.8.8 8.8.4.4
FallbackDNS=1.1.1.1 1.0.0.1
```

Reiniciar serviço:
```bash
sudo systemctl restart systemd-resolved
```

**Windows:**
```
1. Painel de Controle > Rede e Internet > Central de Rede
2. Alterar configurações do adaptador
3. Propriedades da conexão
4. Protocolo IP versão 4 (TCP/IPv4)
5. Usar os seguintes endereços de servidor DNS
6. DNS preferencial: 8.8.8.8
7. DNS alternativo: 8.8.4.4
```

### Solução 3: Verificar Firewall

**Verificar se porta 53 está aberta:**
```bash
sudo netstat -tulpn | grep :53
sudo ss -tulpn | grep :53
```

**Testar conexão à porta DNS:**
```bash
telnet 8.8.8.8 53
nc -zv 8.8.8.8 53
```

### Solução 4: Reiniciar Serviço DNS Local

**systemd-resolved (Ubuntu/Debian):**
```bash
sudo systemctl restart systemd-resolved
sudo systemctl status systemd-resolved
```

**NetworkManager:**
```bash
sudo systemctl restart NetworkManager
```

**dnsmasq:**
```bash
sudo systemctl restart dnsmasq
```

---

## 📊 Servidores DNS Públicos Recomendados

### Google Public DNS
```
Primário: 8.8.8.8
Secundário: 8.8.4.4
```

### Cloudflare
```
Primário: 1.1.1.1
Secundário: 1.0.0.1
```

### OpenDNS
```
Primário: 208.67.222.222
Secundário: 208.67.220.220
```

### Quad9
```
Primário: 9.9.9.9
Secundário: 149.112.112.112
```

---

## 🚨 Problemas Específicos e Soluções

### Problema: "udp port 53 unreachable"

**Causa:** Porta DNS inacessível

**Diagnóstico:**
```bash
sudo tcpdump -i eth0 port 53
```

**Soluções:**
1. Verificar se serviço DNS está rodando
2. Verificar regras de firewall
3. Testar servidor DNS alternativo

### Problema: "connection timed out"

**Causa:** Timeout na conexão DNS

**Diagnóstico:**
```bash
dig google.com +timeout=5
```

**Soluções:**
1. Aumentar timeout
2. Verificar conectividade de rede
3. Trocar servidor DNS

### Problema: "SERVFAIL"

**Causa:** Servidor DNS não conseguiu resolver

**Diagnóstico:**
```bash
dig dominio.com +trace
```

**Soluções:**
1. Verificar se domínio existe
2. Verificar configuração de zona DNS
3. Usar servidor DNS diferente

### Problema: "NXDOMAIN"

**Causa:** Domínio não existe

**Diagnóstico:**
```bash
nslookup dominio.com
```

**Soluções:**
1. Verificar se digitou corretamente
2. Confirmar se domínio está registrado
3. Verificar se DNS está atualizado

---

## 🔒 Verificações de Segurança

### Verificar se DNS está sendo sequestrado

**Teste de resolução:**
```bash
nslookup facebook.com
nslookup twitter.com
```

Se resolver para IPs estranhos, pode haver sequestro de DNS.

### Verificar DNSSec

**Testar com dig:**
```bash
dig +dnssec dominio.com
```

### Verificar propagação DNS

Sites úteis:
- https://www.whatsmydns.net/
- https://dnschecker.org/

---

## 📝 Checklist de Troubleshooting

### Nível 1: Verificações Básicas
- [ ] Testar conectividade (ping 8.8.8.8)
- [ ] Verificar servidor DNS em uso
- [ ] Limpar cache DNS
- [ ] Testar com nslookup

### Nível 2: Diagnóstico Intermediário
- [ ] Capturar tráfego com tcpdump
- [ ] Testar servidor DNS alternativo
- [ ] Verificar firewall
- [ ] Testar com dig +trace

### Nível 3: Análise Avançada
- [ ] Verificar logs do sistema
- [ ] Analisar configuração de rede
- [ ] Verificar serviço DNS local
- [ ] Testar em diferentes redes

---

## 🎓 Interpretando Mensagens de Erro

### Mensagens ICMP

| Mensagem | Significado | Ação |
|----------|-------------|------|
| port unreachable | Porta fechada/filtrada | Verificar firewall |
| host unreachable | Host não alcançável | Verificar rota |
| network unreachable | Rede não alcançável | Verificar gateway |
| ttl exceeded | TTL expirou | Verificar loops |

### Códigos de Resposta DNS

| Código | Significado | Ação |
|--------|-------------|------|
| NOERROR | Sucesso | Tudo OK |
| SERVFAIL | Falha no servidor | Verificar servidor |
| NXDOMAIN | Domínio não existe | Verificar nome |
| REFUSED | Consulta recusada | Verificar ACL |

---

## 💡 Dicas de Prevenção

### Configuração Redundante
- Configure pelo menos 2 servidores DNS
- Use servidores em localizações diferentes
- Tenha DNS primário e secundário

### Monitoramento
- Configure alertas para falhas DNS
- Monitore tempo de resposta
- Registre consultas suspeitas

### Manutenção
- Mantenha servidores DNS atualizados
- Revise configurações regularmente
- Teste failover periodicamente

---

**Lembre-se:** DNS é um serviço crítico. Sempre tenha um plano de contingência!
