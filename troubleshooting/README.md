# Troubleshooting - Implementação de Sistemas

Esta pasta contém guias de solução de problemas específicos para cada laboratório da disciplina.

## 📁 Estrutura

```
troubleshooting/
├── README.md                    # Este arquivo
├── Lab001-Troubleshooting.md    # Deploy Manual em Linux Local
├── Lab002-Troubleshooting.md    # Virtual Hosts e HTTPS
├── Lab003-Troubleshooting.md    # Docker Básico
├── Lab004-Troubleshooting.md    # Docker Compose
├── Lab005-Troubleshooting.md    # AWS CLI e IAM
├── Lab006-Troubleshooting.md    # EC2 e Security Groups
├── Lab008-Troubleshooting.md    # S3 e CloudFront
├── Lab009-Troubleshooting.md    # RDS e Banco de Dados
├── Lab010-Troubleshooting.md    # Load Balancer e Auto Scaling
├── Lab011-Troubleshooting.md    # CloudWatch e Monitoramento
├── Lab012-Troubleshooting.md    # Route 53 e DNS
├── Lab013-Troubleshooting.md    # Lambda e Serverless
├── Lab014-Troubleshooting.md    # CI/CD com GitHub Actions
└── Lab015-Troubleshooting.md    # Projeto Final E-commerce
```

## 🎯 Como Usar

1. **Identifique o lab** onde está tendo problemas
2. **Abra o arquivo correspondente** (ex: Lab001-Troubleshooting.md)
3. **Procure pelo sintoma** que está enfrentando
4. **Siga as soluções** na ordem apresentada
5. **Execute os diagnósticos** quando solicitado

## 🚨 Problemas Gerais (Todos os Labs)

### Problemas de Conectividade
```bash
# Testa conectividade básica
ping google.com
ping 8.8.8.8

# Verifica DNS
nslookup google.com
```

### Problemas de Permissão
```bash
# Verifica usuário atual
whoami
id

# Verifica grupos
groups $USER
```

### Problemas de Espaço em Disco
```bash
# Verifica espaço disponível
df -h

# Verifica uso por diretório
du -sh /var/www/html
du -sh ~/
```

## 📞 Escalação de Problemas

### Nível 1: Auto-diagnóstico
- Execute os comandos de diagnóstico do arquivo específico
- Tente as soluções sugeridas
- Documente o que já tentou

### Nível 2: Colegas de Classe
- Compartilhe o problema no grupo
- Compare configurações
- Trabalhe em dupla para resolver

### Nível 3: Professor
- Tenha em mãos:
  - Arquivo de troubleshooting consultado
  - Comandos já executados
  - Mensagens de erro completas
  - Logs relevantes

## 🔧 Ferramentas Úteis

### Comandos de Sistema
```bash
# Informações do sistema
uname -a
lsb_release -a

# Processos rodando
ps aux | grep nginx
ps aux | grep apache

# Portas em uso
sudo netstat -tulpn
sudo lsof -i
```

### Logs Importantes
```bash
# Logs do sistema
sudo journalctl -f

# Logs de aplicações web
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/apache2/error.log
```

### AWS CLI Úteis
```bash
# Verifica configuração AWS
aws configure list
aws sts get-caller-identity

# Verifica recursos principais
aws ec2 describe-instances
aws s3 ls
aws rds describe-db-instances
```

## 📝 Contribuindo

Se você encontrou um problema não documentado:

1. **Documente o problema** com detalhes
2. **Anote a solução** que funcionou
3. **Compartilhe com o professor** para inclusão no guia

---

**Desenvolvido por:** Professor Alexandre Tavares - UniFAAT  
**Versão:** 1.0 - Semestre 2026.1  
**Última atualização:** Janeiro 2025