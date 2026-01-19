# Troubleshooting Lab 13 - Lambda e Serverless

**Lab:** 013 - Lambda e Serverless  
**Foco:** Funções Lambda, triggers, API Gateway

---

## 🚨 Problemas Mais Comuns

### 1. Função Lambda não Executa

#### **Sintoma:**
- Função não é invocada
- "Function not found" error

#### **Diagnóstico:**
```bash
# Verifica se função existe
aws lambda list-functions

# Testa invocação manual
aws lambda invoke \
  --function-name minha-funcao \
  --payload '{"test": "data"}' \
  response.json

# Verifica logs
aws logs describe-log-groups --log-group-name-prefix /aws/lambda/minha-funcao
```

#### **Soluções:**
```bash
# Verifica configuração da função
aws lambda get-function --function-name minha-funcao

# Atualiza código da função
zip function.zip index.js
aws lambda update-function-code \
  --function-name minha-funcao \
  --zip-file fileb://function.zip
```

### 2. Timeout da Função

#### **Sintoma:**
- "Task timed out after X seconds"
- Função para no meio da execução

#### **Soluções:**
```bash
# Aumenta timeout (máximo 15 minutos)
aws lambda update-function-configuration \
  --function-name minha-funcao \
  --timeout 300

# Otimiza código para ser mais rápido
# Usa conexões persistentes
# Evita operações síncronas demoradas
```

### 3. Erro de Permissões

#### **Sintoma:**
- "AccessDenied" em recursos AWS
- "User is not authorized to perform"

#### **Soluções:**
```bash
# Verifica role da função
aws lambda get-function --function-name minha-funcao \
  --query 'Configuration.Role'

# Adiciona permissões ao role
aws iam attach-role-policy \
  --role-name lambda-execution-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Cria policy customizada
aws iam create-policy \
  --policy-name LambdaS3Access \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::meu-bucket/*"
    }]
  }'
```

### 4. API Gateway não Funciona

#### **Sintoma:**
- "Internal Server Error" (500)
- "Missing Authentication Token" (403)

#### **Diagnóstico:**
```bash
# Verifica API Gateway
aws apigateway get-rest-apis

# Testa endpoint
curl -X GET https://api-id.execute-api.region.amazonaws.com/stage/resource

# Verifica logs do API Gateway
aws logs describe-log-groups --log-group-name-prefix API-Gateway-Execution-Logs
```

#### **Soluções:**
```bash
# Habilita logs no API Gateway
aws apigateway update-stage \
  --rest-api-id api-id \
  --stage-name prod \
  --patch-ops op=replace,path=/accessLogSettings/destinationArn,value=arn:aws:logs:region:account:log-group:api-gateway-logs

# Redeploy da API
aws apigateway create-deployment \
  --rest-api-id api-id \
  --stage-name prod

# Verifica integração com Lambda
aws apigateway get-integration \
  --rest-api-id api-id \
  --resource-id resource-id \
  --http-method GET
```

### 5. Variáveis de Ambiente não Funcionam

#### **Sintoma:**
- process.env.VARIABLE retorna undefined
- Configuração não é lida

#### **Soluções:**
```bash
# Define variáveis de ambiente
aws lambda update-function-configuration \
  --function-name minha-funcao \
  --environment Variables='{
    "DB_HOST": "database.amazonaws.com",
    "DB_NAME": "mydb",
    "API_KEY": "secret-key"
  }'

# Verifica variáveis definidas
aws lambda get-function-configuration \
  --function-name minha-funcao \
  --query 'Environment'
```

### 6. Cold Start Lento

#### **Sintoma:**
- Primeira invocação demora muito
- Timeout em funções raramente usadas

#### **Soluções:**
```bash
# Aumenta memória (melhora CPU também)
aws lambda update-function-configuration \
  --function-name minha-funcao \
  --memory-size 512

# Configura Provisioned Concurrency
aws lambda put-provisioned-concurrency-config \
  --function-name minha-funcao \
  --qualifier $LATEST \
  --provisioned-concurrency-config ProvisionedConcurrencyConfigs=10

# Otimiza código:
# - Inicializa conexões fora do handler
# - Use linguagens com cold start mais rápido (Python, Node.js)
# - Minimize dependências
```

### 7. Erro de Payload

#### **Sintoma:**
- "Invalid request body"
- JSON parsing errors

#### **Soluções:**
```bash
# Testa payload manualmente
aws lambda invoke \
  --function-name minha-funcao \
  --payload '{"key": "value"}' \
  --cli-binary-format raw-in-base64-out \
  response.json

# Verifica formato esperado no código
# Exemplo Node.js:
cat << EOF > index.js
exports.handler = async (event) => {
    console.log('Event:', JSON.stringify(event, null, 2));
    
    try {
        // Processa event aqui
        const response = {
            statusCode: 200,
            body: JSON.stringify({
                message: 'Success',
                input: event
            })
        };
        return response;
    } catch (error) {
        console.error('Error:', error);
        return {
            statusCode: 500,
            body: JSON.stringify({
                error: error.message
            })
        };
    }
};
EOF
```

### 8. Logs não Aparecem

#### **Sintoma:**
- CloudWatch Logs vazio
- console.log não aparece

#### **Soluções:**
```bash
# Verifica se log group existe
aws logs describe-log-groups --log-group-name-prefix /aws/lambda/minha-funcao

# Cria log group se necessário
aws logs create-log-group --log-group-name /aws/lambda/minha-funcao

# Verifica permissões do role
aws iam get-role-policy \
  --role-name lambda-execution-role \
  --policy-name lambda-logs-policy

# Adiciona permissões de logs
aws iam put-role-policy \
  --role-name lambda-execution-role \
  --policy-name lambda-logs-policy \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    }]
  }'
```

---

**Desenvolvido por:** Professor Alexandre Tavares - UniFAAT