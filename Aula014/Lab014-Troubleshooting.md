# Troubleshooting Lab 14 - CI/CD com GitHub Actions

**Lab:** 014 - CI/CD com GitHub Actions  
**Foco:** Pipelines, deploy automatizado, integração contínua

---

## 🚨 Problemas Mais Comuns

### 1. Workflow não Executa

#### **Sintoma:**
- Push/PR não dispara workflow
- Actions tab vazio

#### **Diagnóstico:**
```bash
# Verifica se arquivo está no local correto
ls -la .github/workflows/

# Verifica sintaxe YAML
cat .github/workflows/deploy.yml | python -c "import yaml, sys; yaml.safe_load(sys.stdin)"
```

#### **Soluções:**
```yaml
# Corrige triggers no workflow
name: Deploy
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy
        run: echo "Deploying..."
```

### 2. Secrets não Funcionam

#### **Sintoma:**
- "Secret not found"
- Variáveis de ambiente vazias

#### **Soluções:**
```yaml
# Usa secrets corretamente
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Configure AWS
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
          aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
```

**Configurar secrets no GitHub:**
1. Repository → Settings → Secrets and variables → Actions
2. New repository secret
3. Adicionar: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY

### 3. Build Falha

#### **Sintoma:**
- "npm install failed"
- "docker build failed"

#### **Diagnóstico:**
```yaml
# Adiciona debug no workflow
- name: Debug
  run: |
    pwd
    ls -la
    node --version
    npm --version
```

#### **Soluções:**
```yaml
# Corrige dependências Node.js
- name: Setup Node.js
  uses: actions/setup-node@v3
  with:
    node-version: '18'
    cache: 'npm'

- name: Install dependencies
  run: npm ci

- name: Build
  run: npm run build

# Corrige build Docker
- name: Build Docker image
  run: |
    docker build -t myapp:${{ github.sha }} .
    docker tag myapp:${{ github.sha }} myapp:latest
```

### 4. Deploy para AWS Falha

#### **Sintoma:**
- "AccessDenied" durante deploy
- "InvalidParameterValue" errors

#### **Soluções:**
```yaml
# Configura AWS credentials corretamente
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v2
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-east-1

# Deploy para S3
- name: Deploy to S3
  run: |
    aws s3 sync ./build s3://meu-bucket --delete

# Deploy para EC2
- name: Deploy to EC2
  run: |
    echo "${{ secrets.EC2_SSH_KEY }}" > private_key.pem
    chmod 600 private_key.pem
    scp -i private_key.pem -o StrictHostKeyChecking=no ./app.zip ec2-user@${{ secrets.EC2_HOST }}:/tmp/
    ssh -i private_key.pem -o StrictHostKeyChecking=no ec2-user@${{ secrets.EC2_HOST }} '
      cd /var/www/html
      sudo unzip -o /tmp/app.zip
      sudo systemctl restart nginx
    '
```

### 5. Testes Falham

#### **Sintoma:**
- Unit tests não passam
- Coverage insuficiente

#### **Soluções:**
```yaml
# Adiciona step de testes
- name: Run tests
  run: |
    npm test
    npm run test:coverage

# Configura ambiente de teste
- name: Setup test environment
  run: |
    cp .env.example .env.test
    npm run db:migrate:test

# Usa services para banco de dados
services:
  postgres:
    image: postgres:13
    env:
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: test_db
    options: >-
      --health-cmd pg_isready
      --health-interval 10s
      --health-timeout 5s
      --health-retries 5
```

### 6. Workflow Muito Lento

#### **Sintoma:**
- Build demora mais de 10 minutos
- Timeout em steps

#### **Soluções:**
```yaml
# Usa cache para dependências
- name: Cache node modules
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

# Paraleliza jobs
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Run tests
        run: npm test
  
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build
        run: npm run build
  
  deploy:
    needs: [test, build]
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: echo "Deploying..."
```

### 7. Rollback não Funciona

#### **Sintoma:**
- Deploy com erro não reverte
- Versão anterior não restaura

#### **Soluções:**
```yaml
# Implementa rollback automático
- name: Deploy with rollback
  run: |
    # Backup da versão atual
    aws s3 sync s3://meu-bucket s3://meu-bucket-backup/$(date +%Y%m%d-%H%M%S) --delete
    
    # Deploy nova versão
    aws s3 sync ./build s3://meu-bucket --delete
    
    # Testa se deploy funcionou
    if ! curl -f https://meusite.com/health; then
      echo "Deploy failed, rolling back..."
      aws s3 sync s3://meu-bucket-backup/latest s3://meu-bucket --delete
      exit 1
    fi

# Usa Blue-Green deployment
- name: Blue-Green Deploy
  run: |
    # Deploy para ambiente green
    aws s3 sync ./build s3://meu-bucket-green --delete
    
    # Testa ambiente green
    if curl -f https://green.meusite.com/health; then
      # Troca DNS para green
      aws route53 change-resource-record-sets --hosted-zone-id Z123 --change-batch file://switch-to-green.json
    else
      echo "Green environment failed health check"
      exit 1
    fi
```

### 8. Notificações não Funcionam

#### **Sintoma:**
- Não recebe notificação de falha
- Slack/Discord não notifica

#### **Soluções:**
```yaml
# Adiciona notificação Slack
- name: Notify Slack on failure
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: failure
    channel: '#deployments'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}

# Notificação por email
- name: Send email on failure
  if: failure()
  uses: dawidd6/action-send-mail@v3
  with:
    server_address: smtp.gmail.com
    server_port: 587
    username: ${{ secrets.EMAIL_USERNAME }}
    password: ${{ secrets.EMAIL_PASSWORD }}
    subject: "Deploy Failed: ${{ github.repository }}"
    body: "Deploy failed for commit ${{ github.sha }}"
    to: admin@meusite.com
```

---

**Desenvolvido por:** Professor Alexandre Tavares - UniFAAT