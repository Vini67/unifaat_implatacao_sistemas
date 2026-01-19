# Troubleshooting Lab 15 - Projeto Final E-commerce

**Lab:** 015 - Projeto Final E-commerce  
**Foco:** Integração completa, arquitetura AWS, troubleshooting avançado

---

## 🚨 Problemas Mais Comuns

### 1. Arquitetura Completa não Funciona

#### **Sintoma:**
- Componentes isolados funcionam, mas integração falha
- Load Balancer não encontra targets
- Frontend não conecta com backend

#### **Diagnóstico:**
```bash
# Verifica todos os componentes
aws elbv2 describe-target-health --target-group-arn arn:aws:elasticloadbalancing:...
aws rds describe-db-instances --db-instance-identifier ecommerce-db
aws s3 ls s3://ecommerce-frontend
aws cloudfront list-distributions
```

#### **Soluções:**
```bash
# Verifica conectividade entre componentes
# EC2 → RDS
mysql -h ecommerce-db.cluster-xyz.us-east-1.rds.amazonaws.com -u admin -p

# Frontend → Backend via ALB
curl https://api.meusite.com/health

# Verifica Security Groups
aws ec2 describe-security-groups --group-ids sg-backend sg-database sg-alb
```

### 2. Auto Scaling não Responde à Carga

#### **Sintoma:**
- CPU alta mas novas instâncias não são criadas
- Instâncias criadas mas não recebem tráfego

#### **Soluções:**
```bash
# Verifica políticas de scaling
aws autoscaling describe-policies --auto-scaling-group-name ecommerce-asg

# Força scaling para teste
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name ecommerce-asg \
  --desired-capacity 3

# Verifica alarmes CloudWatch
aws cloudwatch describe-alarms --alarm-names cpu-high-ecommerce

# Testa health check das novas instâncias
curl -I http://nova-instancia-ip/health
```

### 3. Banco de Dados com Performance Ruim

#### **Sintoma:**
- Queries lentas
- Timeouts de conexão
- CPU do RDS em 100%

#### **Diagnóstico:**
```bash
# Verifica métricas RDS
aws cloudwatch get-metric-statistics \
  --namespace AWS/RDS \
  --metric-name CPUUtilization \
  --dimensions Name=DBInstanceIdentifier,Value=ecommerce-db \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-01T23:59:59Z \
  --period 300 \
  --statistics Average,Maximum

# Verifica conexões ativas
mysql -h endpoint -u admin -p -e "SHOW PROCESSLIST;"
```

#### **Soluções:**
```bash
# Otimiza queries
mysql -h endpoint -u admin -p -e "
  EXPLAIN SELECT * FROM produtos WHERE categoria = 'eletrônicos';
  CREATE INDEX idx_categoria ON produtos(categoria);
"

# Implementa connection pooling no backend
# Exemplo Node.js:
cat << EOF > db-pool.js
const mysql = require('mysql2');
const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  connectionLimit: 10,
  acquireTimeout: 60000,
  timeout: 60000
});
module.exports = pool.promise();
EOF

# Upgrade da instância RDS se necessário
aws rds modify-db-instance \
  --db-instance-identifier ecommerce-db \
  --db-instance-class db.t3.medium \
  --apply-immediately
```

### 4. CloudFront Cache Issues

#### **Sintoma:**
- Mudanças no frontend não aparecem
- API calls sendo cacheadas incorretamente

#### **Soluções:**
```bash
# Configura cache behaviors corretos
aws cloudfront update-distribution \
  --id E1234567890ABC \
  --distribution-config '{
    "CallerReference": "ecommerce-'$(date +%s)'",
    "DefaultCacheBehavior": {
      "TargetOriginId": "S3-ecommerce-frontend",
      "ViewerProtocolPolicy": "redirect-to-https",
      "CachePolicyId": "4135ea2d-6df8-44a3-9df3-4b5a84be39ad",
      "Compress": true
    },
    "CacheBehaviors": {
      "Items": [{
        "PathPattern": "/api/*",
        "TargetOriginId": "ALB-ecommerce-backend",
        "ViewerProtocolPolicy": "https-only",
        "CachePolicyId": "4135ea2d-6df8-44a3-9df3-4b5a84be39ad",
        "TTL": 0
      }]
    }
  }'

# Invalida cache quando necessário
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/index.html" "/static/js/*" "/static/css/*"
```

### 5. Monitoramento Insuficiente

#### **Sintoma:**
- Não detecta problemas rapidamente
- Alertas não disparam
- Logs não são coletados

#### **Soluções:**
```bash
# Configura alarmes essenciais
aws cloudwatch put-metric-alarm \
  --alarm-name "ECommerce-HighErrorRate" \
  --alarm-description "High error rate in ALB" \
  --metric-name HTTPCode_Target_5XX_Count \
  --namespace AWS/ApplicationELB \
  --statistic Sum \
  --period 300 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:ecommerce-alerts

# Configura logs estruturados no backend
cat << EOF > logger.js
const winston = require('winston');
const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: '/var/log/ecommerce/app.log' })
  ]
});
module.exports = logger;
EOF

# Envia logs para CloudWatch
sudo tee -a /etc/awslogs/awslogs.conf << EOF
[/var/log/ecommerce/app.log]
datetime_format = %Y-%m-%d %H:%M:%S
file = /var/log/ecommerce/app.log
buffer_duration = 5000
log_stream_name = {instance_id}/ecommerce/app.log
initial_position = start_of_file
log_group_name = /aws/ec2/ecommerce
EOF
```

### 6. Segurança Vulnerável

#### **Sintoma:**
- Security Groups muito permissivos
- Dados sensíveis em logs
- HTTPS não funcionando

#### **Soluções:**
```bash
# Revisa Security Groups
aws ec2 describe-security-groups --group-ids sg-ecommerce-web \
  --query 'SecurityGroups[0].IpPermissions[?IpRanges[?CidrIp==`0.0.0.0/0`]]'

# Corrige regras muito abertas
aws ec2 revoke-security-group-ingress \
  --group-id sg-ecommerce-web \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
  --group-id sg-ecommerce-web \
  --protocol tcp \
  --port 22 \
  --cidr 10.0.0.0/8

# Usa Secrets Manager para credenciais
aws secretsmanager create-secret \
  --name ecommerce/database \
  --description "Database credentials for ecommerce" \
  --secret-string '{"username":"admin","password":"secure-password"}'

# Atualiza aplicação para usar secrets
cat << EOF > get-secret.js
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager();

async function getDatabaseCredentials() {
  const secret = await secretsManager.getSecretValue({
    SecretId: 'ecommerce/database'
  }).promise();
  return JSON.parse(secret.SecretString);
}
EOF
```

### 7. Backup e Disaster Recovery

#### **Sintoma:**
- Sem estratégia de backup
- Recovery time muito alto
- Dados perdidos

#### **Soluções:**
```bash
# Configura backup automático RDS
aws rds modify-db-instance \
  --db-instance-identifier ecommerce-db \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "sun:04:00-sun:05:00"

# Backup do S3 para outra região
aws s3api put-bucket-replication \
  --bucket ecommerce-frontend \
  --replication-configuration '{
    "Role": "arn:aws:iam::123456789012:role/replication-role",
    "Rules": [{
      "Status": "Enabled",
      "Prefix": "",
      "Destination": {
        "Bucket": "arn:aws:s3:::ecommerce-frontend-backup",
        "StorageClass": "STANDARD_IA"
      }
    }]
  }'

# Snapshot das instâncias EC2
aws ec2 create-snapshot \
  --volume-id vol-1234567890abcdef0 \
  --description "Ecommerce app backup $(date)"

# Testa procedimento de recovery
# 1. Cria nova instância RDS de snapshot
# 2. Atualiza DNS para ambiente de backup
# 3. Verifica funcionamento completo
```

### 8. Performance End-to-End

#### **Sintoma:**
- Site lento para usuários finais
- Timeouts intermitentes
- Experiência ruim no mobile

#### **Soluções:**
```bash
# Otimiza CloudFront
aws cloudfront update-distribution \
  --id E1234567890ABC \
  --distribution-config '{
    "PriceClass": "PriceClass_All",
    "HttpVersion": "http2",
    "IsIPV6Enabled": true,
    "DefaultCacheBehavior": {
      "Compress": true,
      "ViewerProtocolPolicy": "redirect-to-https"
    }
  }'

# Implementa CDN para assets estáticos
# Move imagens para S3 + CloudFront
aws s3 sync ./images s3://ecommerce-assets/images/
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/images/*"

# Otimiza queries do banco
mysql -h endpoint -u admin -p << EOF
-- Adiciona índices para queries frequentes
CREATE INDEX idx_produto_categoria ON produtos(categoria);
CREATE INDEX idx_produto_preco ON produtos(preco);
CREATE INDEX idx_pedido_data ON pedidos(data_pedido);

-- Otimiza query de produtos em destaque
EXPLAIN SELECT * FROM produtos 
WHERE categoria = 'eletrônicos' 
AND preco BETWEEN 100 AND 1000 
ORDER BY vendas DESC 
LIMIT 10;
EOF

# Implementa cache Redis (opcional)
# Para sessões e dados frequentemente acessados
```

---

**Desenvolvido por:** Professor Alexandre Tavares - UniFAAT