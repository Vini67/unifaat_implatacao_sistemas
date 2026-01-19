# Troubleshooting Lab 12 - Route 53 e DNS

**Lab:** 012 - Route 53 e DNS  
**Foco:** DNS, domínios, roteamento

---

## 🚨 Problemas Mais Comuns

### 1. Domínio não Resolve

#### **Sintoma:**
```bash
nslookup meusite.com
# Server can't find meusite.com: NXDOMAIN
```

#### **Diagnóstico:**
```bash
# Verifica hosted zone
aws route53 list-hosted-zones

# Verifica records
aws route53 list-resource-record-sets --hosted-zone-id Z1234567890ABC

# Testa DNS público
dig @8.8.8.8 meusite.com
```

#### **Soluções:**
```bash
# Cria record A
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "meusite.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{"Value": "1.2.3.4"}]
      }
    }]
  }'
```

### 2. Propagação DNS Lenta

#### **Sintoma:**
- Domínio funciona em alguns lugares, outros não
- Mudanças demoram para aparecer

#### **Diagnóstico:**
```bash
# Testa diferentes servidores DNS
dig @8.8.8.8 meusite.com
dig @1.1.1.1 meusite.com
dig @208.67.222.222 meusite.com

# Verifica TTL
dig meusite.com | grep -E "^meusite.com.*IN.*A"
```

#### **Soluções:**
```bash
# Reduz TTL para mudanças futuras
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "meusite.com",
        "Type": "A",
        "TTL": 60,
        "ResourceRecords": [{"Value": "1.2.3.4"}]
      }
    }]
  }'

# Aguarda propagação (pode levar até 48h)
# Usa ferramentas online: whatsmydns.net
```

### 3. CNAME não Funciona

#### **Sintoma:**
- www.meusite.com não resolve
- "CNAME lookup failed"

#### **Soluções:**
```bash
# Cria CNAME record
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "www.meusite.com",
        "Type": "CNAME",
        "TTL": 300,
        "ResourceRecords": [{"Value": "meusite.com"}]
      }
    }]
  }'

# Ou usa ALIAS para Load Balancer
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "www.meusite.com",
        "Type": "A",
        "AliasTarget": {
          "DNSName": "my-load-balancer-1234567890.us-east-1.elb.amazonaws.com",
          "EvaluateTargetHealth": false,
          "HostedZoneId": "Z35SXDOTRQ7X7K"
        }
      }
    }]
  }'
```

### 4. Health Check Falha

#### **Sintoma:**
- Route 53 health check marca como unhealthy
- Failover não funciona

#### **Diagnóstico:**
```bash
# Verifica health checks
aws route53 list-health-checks

# Verifica status específico
aws route53 get-health-check-status --health-check-id 12345678-1234-1234-1234-123456789012
```

#### **Soluções:**
```bash
# Cria health check
aws route53 create-health-check \
  --caller-reference $(date +%s) \
  --health-check-config '{
    "Type": "HTTP",
    "ResourcePath": "/health",
    "FullyQualifiedDomainName": "meusite.com",
    "Port": 80,
    "RequestInterval": 30,
    "FailureThreshold": 3
  }'

# Associa health check ao record
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "meusite.com",
        "Type": "A",
        "SetIdentifier": "primary",
        "Failover": "PRIMARY",
        "TTL": 60,
        "ResourceRecords": [{"Value": "1.2.3.4"}],
        "HealthCheckId": "12345678-1234-1234-1234-123456789012"
      }
    }]
  }'
```

### 5. Subdomain Delegation

#### **Sintoma:**
- api.meusite.com não resolve
- Subdomínio não funciona

#### **Soluções:**
```bash
# Cria hosted zone para subdomínio
aws route53 create-hosted-zone \
  --name api.meusite.com \
  --caller-reference $(date +%s)

# Obtém name servers da nova zone
aws route53 get-hosted-zone --id Z9876543210DEF

# Cria NS record no domínio principal
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "api.meusite.com",
        "Type": "NS",
        "TTL": 300,
        "ResourceRecords": [
          {"Value": "ns-1.awsdns-01.com"},
          {"Value": "ns-2.awsdns-02.net"},
          {"Value": "ns-3.awsdns-03.org"},
          {"Value": "ns-4.awsdns-04.co.uk"}
        ]
      }
    }]
  }'
```

### 6. Geolocation Routing não Funciona

#### **Sintoma:**
- Usuários não são direcionados para servidor mais próximo
- Geolocation não detecta localização

#### **Soluções:**
```bash
# Cria records com geolocation
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "meusite.com",
        "Type": "A",
        "SetIdentifier": "US-East",
        "GeoLocation": {"CountryCode": "US"},
        "TTL": 60,
        "ResourceRecords": [{"Value": "1.2.3.4"}]
      }
    }, {
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "meusite.com",
        "Type": "A",
        "SetIdentifier": "Default",
        "GeoLocation": {"CountryCode": "*"},
        "TTL": 60,
        "ResourceRecords": [{"Value": "5.6.7.8"}]
      }
    }]
  }'

# Testa geolocation
curl -H "CF-IPCountry: BR" http://meusite.com
```

---

**Desenvolvido por:** Professor Alexandre Tavares - UniFAAT