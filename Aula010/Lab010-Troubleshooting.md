# Troubleshooting Lab 10 - Load Balancer e Auto Scaling

**Lab:** 010 - Load Balancer e Auto Scaling  
**Foco:** ALB, Target Groups, Auto Scaling Groups

---

## 🚨 Problemas Mais Comuns

### 1. Load Balancer não Distribui Tráfego

#### **Sintoma:**
- Todas as requisições vão para uma instância
- "503 Service Unavailable"

#### **Diagnóstico:**
```bash
# Verifica health checks
aws elbv2 describe-target-health --target-group-arn arn:aws:elasticloadbalancing:...

# Verifica configuração do ALB
aws elbv2 describe-load-balancers --load-balancer-arns arn:aws:elasticloadbalancing:...
```

#### **Soluções:**
```bash
# Registra targets no target group
aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:... \
  --targets Id=i-1234567890abcdef0,Port=80

# Ajusta health check
aws elbv2 modify-target-group \
  --target-group-arn arn:aws:elasticloadbalancing:... \
  --health-check-path /health \
  --health-check-interval-seconds 30
```

### 2. Health Check Falha

#### **Sintoma:**
- Targets aparecem como "unhealthy"
- Instâncias são removidas do balanceador

#### **Diagnóstico:**
```bash
# Testa health check manualmente
curl -I http://ip-da-instancia/health

# Verifica logs da aplicação
ssh -i key.pem ec2-user@ip-da-instancia
sudo tail -f /var/log/nginx/access.log
```

#### **Soluções:**
```bash
# Cria endpoint de health check
echo "OK" | sudo tee /var/www/html/health

# Ajusta configurações do health check
aws elbv2 modify-target-group \
  --target-group-arn arn:aws:elasticloadbalancing:... \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 5 \
  --health-check-timeout-seconds 10
```

### 3. Auto Scaling não Funciona

#### **Sintoma:**
- Instâncias não são criadas automaticamente
- Scaling policies não disparam

#### **Diagnóstico:**
```bash
# Verifica Auto Scaling Group
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names meu-asg

# Verifica políticas de scaling
aws autoscaling describe-policies --auto-scaling-group-name meu-asg

# Verifica alarmes do CloudWatch
aws cloudwatch describe-alarms
```

#### **Soluções:**
```bash
# Ajusta capacidade mínima/máxima
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name meu-asg \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 3

# Cria política de scaling
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name meu-asg \
  --policy-name scale-up \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    }
  }'
```

### 4. Launch Template Incorreto

#### **Sintoma:**
- Novas instâncias não têm a aplicação
- Instâncias falham no boot

#### **Soluções:**
```bash
# Verifica Launch Template
aws ec2 describe-launch-templates --launch-template-names meu-template

# Atualiza User Data
aws ec2 create-launch-template-version \
  --launch-template-name meu-template \
  --version-description "Fixed user data" \
  --launch-template-data '{
    "UserData": "base64-encoded-script"
  }'

# Atualiza ASG para usar nova versão
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name meu-asg \
  --launch-template '{
    "LaunchTemplateName": "meu-template",
    "Version": "$Latest"
  }'
```

### 5. SSL/TLS no Load Balancer

#### **Sintoma:**
- "Your connection is not private"
- Certificado inválido

#### **Soluções:**
```bash
# Lista certificados disponíveis
aws acm list-certificates

# Adiciona listener HTTPS
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:... \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=arn:aws:acm:... \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:...

# Redireciona HTTP para HTTPS
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:... \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=redirect,RedirectConfig='{
    "Protocol": "HTTPS",
    "Port": "443",
    "StatusCode": "HTTP_301"
  }'
```

### 6. Sticky Sessions não Funcionam

#### **Sintoma:**
- Usuário perde sessão entre requisições
- Login não persiste

#### **Soluções:**
```bash
# Habilita sticky sessions
aws elbv2 modify-target-group-attributes \
  --target-group-arn arn:aws:elasticloadbalancing:... \
  --attributes Key=stickiness.enabled,Value=true \
               Key=stickiness.type,Value=lb_cookie \
               Key=stickiness.lb_cookie.duration_seconds,Value=86400
```

---

**Desenvolvido por:** Professor Alexandre Tavares - UniFAAT