# Lab 7 - Prova Teórico-Prática: Implementação de Software
**Disciplina:** Implementação de Software  
**Curso:** Análise e Desenvolvimento de Sistemas - UniFAAT  
**Duração:** 3 horas (1h teoria + 2h prática)  
**Valor:** 10 pontos (3 pontos teoria + 7 pontos prática)



## Instruções Gerais

### Parte Teórica (3 pontos - 60 minutos)
- 6 questões de múltipla escolha (0,5 pontos cada)
- Marque apenas UMA alternativa por questão
- Não é permitido consulta durante a parte teórica

### Parte Prática (7 pontos - 120 minutos)
- Desafio técnico completo
- Entrega via GitHub
- Envio do link pelo formulário Google Forms
- Consulta aos labs anteriores permitida



## PARTE TEÓRICA - Questões de Múltipla Escolha

### Questão 1 (0,5 pontos)
**Qual a principal diferença entre `apt update` e `apt upgrade` no Ubuntu?**

a) `apt update` instala novos pacotes, `apt upgrade` atualiza os existentes  
b) `apt update` atualiza a lista de pacotes disponíveis, `apt upgrade` instala as atualizações  
c) `apt update` é mais rápido que `apt upgrade`  
d) Não há diferença, são comandos equivalentes  
e) `apt update` requer sudo, `apt upgrade` não



### Questão 2 (0,5 pontos)
**No Docker Compose, qual a diferença entre `depends_on` simples e `depends_on` com `condition: service_healthy`?**

a) Não há diferença prática entre as duas formas  
b) `depends_on` simples aguarda o container estar healthy, `condition` apenas a ordem  
c) `depends_on` simples apenas garante ordem de inicialização, `condition` aguarda saúde  
d) `condition: service_healthy` é mais rápido que `depends_on` simples  
e) `depends_on` simples funciona apenas com bancos de dados



### Questão 3 (0,5 pontos)
**Em um healthcheck do Docker, qual o propósito do parâmetro `start_period`?**

a) Define o intervalo entre execuções do teste de saúde  
b) Estabelece o tempo máximo para o comando executar  
c) Determina quantas tentativas antes de marcar como unhealthy  
d) Define um período de carência onde falhas não contam para retries  
e) Especifica quando o healthcheck deve começar a executar



### Questão 4 (0,5 pontos)
**No Nginx, qual a função do bloco `upstream` em um cenário de load balancing?**

a) Define as configurações de SSL/TLS  
b) Configura os logs de acesso  
c) Agrupa servidores backend para distribuição de carga  
d) Estabelece regras de firewall  
e) Configura cache de conteúdo estático



### Questão 5 (0,5 pontos)
**Qual comando Docker permite monitorar eventos de containers em tempo real?**

a) `docker logs -f`  
b) `docker ps --watch`  
c) `docker events`  
d) `docker inspect --follow`  
e) `docker monitor`



### Questão 6 (0,5 pontos)
**Por que usar `nginx:alpine` ao invés de `nginx:latest` em produção?**

a) Alpine é mais estável que a versão latest  
b) Alpine tem melhor performance de CPU  
c) Alpine é menor, mais seguro e consome menos recursos  
d) Alpine suporta mais módulos do Nginx  
e) Alpine é obrigatório para usar com Docker Compose



## 🔧 PARTE PRÁTICA - Desafio Técnico (7 pontos)

### Cenário: Sistema de E-commerce Resiliente

Você foi contratado para implementar a infraestrutura de um e-commerce que precisa ser altamente disponível. O sistema deve ter:

- **Frontend**: Site de vendas (Nginx customizado)
- **API**: Serviço de produtos (2 instâncias para alta disponibilidade)
- **Database**: Banco de dados MySQL com persistência
- **Proxy**: Load balancer inteligente



### Requisitos Técnicos

#### 1. Estrutura do Projeto (1 ponto)
Crie um repositório GitHub com a seguinte estrutura:
```
ecommerce-infrastructure/
├── README.md
├── Dockerfile
├── docker-compose.yml
├── nginx.conf
├── frontend/
│   └── index.html
└── scripts/
    └── health_monitor.sh
```

#### 2. Frontend Personalizado (1,5 pontos)
**Dockerfile requirements:**
- Base: `nginx:alpine`
- Instalar `curl` e `jq` para healthchecks
- Copiar arquivos do frontend
- Expor porta 80

**index.html requirements:**
- Título: "TechCommerce - Sua Loja Online"
- Mostrar: Nome do aluno, RA, timestamp de build
- CSS responsivo básico
- Seção com informações do container (hostname)

#### 3. Orquestração Completa (2 pontos)
**docker-compose.yml deve conter:**

**Serviço `database`:**
- MySQL 8.0
- Volume persistente `mysql_data`
- Variáveis: `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`
- Healthcheck com `mysqladmin ping`

**Serviços `api1` e `api2`:**
- Usar imagem `nginxdemos/hello` (simula API)
- Portas internas diferentes
- Dependência: database healthy
- Healthcheck próprio

**Serviço `loadbalancer`:**
- Sua imagem customizada
- Porta externa: 8080
- Volume: nginx.conf personalizado
- Dependência: api1 e api2 healthy

#### 4. Load Balancer Inteligente (1,5 pontos)
**nginx.conf requirements:**
- Upstream com api1 e api2
- Algoritmo: least_conn
- Headers de proxy corretos
- Endpoint `/health` para monitoramento
- Log format personalizado com upstream_addr

#### 5. Monitoramento Automatizado (1 ponto)
**Script health_monitor.sh:**
- Verificar status de todos os containers
- Exibir apenas containers unhealthy
- Usar cores (verde=healthy, vermelho=unhealthy)
- Timestamp da verificação
- Executável e com tratamento de erro



### Critérios de Avaliação

#### Funcionalidade (4 pontos)
- **Stack completa funcionando** (1,5 pts)
- **Load balancing operacional** (1 pt)
- **Healthchecks configurados** (1 pt)
- **Persistência de dados** (0,5 pts)

#### Qualidade Técnica (2 pontos)
- **Boas práticas Docker** (0,5 pts)
- **Configuração Nginx correta** (0,5 pts)
- **Scripts funcionais** (0,5 pts)
- **Tratamento de erros** (0,5 pts)

#### Documentação (1 ponto)
- **README completo** (0,5 pts)
- **Comentários no código** (0,5 pts)



### README.md Obrigatório

Seu README deve conter:

```markdown
# E-commerce Infrastructure

## Aluno
- **Nome:** [Seu Nome Completo]
- **RA:** [Seu RA]
- **Curso:** Análise e Desenvolvimento de Sistemas

## Descrição
Sistema de e-commerce com alta disponibilidade usando Docker Compose.

## Arquitetura
- Frontend: Nginx customizado
- API: 2 instâncias com load balancing
- Database: MySQL com persistência
- Proxy: Nginx load balancer

## Como Executar
```bash
# Clone o repositório
git clone [seu-repo-url]
cd ecommerce-infrastructure

# Subir o stack
docker compose up -d --build

# Verificar status
docker compose ps

# Testar load balancer
curl http://localhost:8080
```

## Monitoramento
```bash
# Executar script de monitoramento
./scripts/health_monitor.sh
```

## Testes Realizados
- [ ] Stack sobe sem erros
- [ ] Load balancer distribui tráfego
- [ ] Healthchecks funcionando
- [ ] Persistência de dados
- [ ] Script de monitoramento

## Problemas Encontrados
[Descreva problemas encontrados e como resolveu]
```



### Comandos de Validação

Antes de entregar, execute:

```bash
# 1. Build e start
docker compose up -d --build

# 2. Verificar todos healthy
docker compose ps

# 3. Testar load balancer (deve alternar)
for i in {1..6}; do 
  curl -s http://localhost:8080 | grep "Server name"
done

# 4. Testar persistência
docker compose restart database
sleep 10
docker compose ps

# 5. Script de monitoramento
./scripts/health_monitor.sh

# 6. Logs sem erros críticos
docker compose logs --tail=50
```



### Entrega

1. **Criar repositório público no GitHub**
2. **Fazer commit de todos os arquivos**
3. **Testar que funciona com `git clone`**
4. **Enviar link via formulário Google Forms**

**Link do formulário:** [Será fornecido pelo professor]

**Prazo:** Até 23:59 do dia da prova



## 📚 GABARITO - Parte Teórica

### Questão 1: **Resposta B**
**Justificativa:** `apt update` atualiza apenas a lista de pacotes disponíveis nos repositórios, enquanto `apt upgrade` efetivamente instala as atualizações dos pacotes já instalados. É como "verificar se há atualizações" vs "instalar as atualizações".

### Questão 2: **Resposta C**
**Justificativa:** `depends_on` simples apenas garante a ordem de inicialização dos containers, mas não verifica se estão funcionais. `condition: service_healthy` aguarda até que o serviço dependente esteja marcado como "healthy" pelo healthcheck.

### Questão 3: **Resposta D**
**Justificativa:** `start_period` define um período de carência após a inicialização do container onde falhas no healthcheck não contam para o número de retries. Isso é importante para aplicações que demoram para inicializar completamente.

### Questão 4: **Resposta C**
**Justificativa:** O bloco `upstream` no Nginx agrupa múltiplos servidores backend, permitindo que o Nginx distribua as requisições entre eles usando algoritmos como round-robin, least_conn, etc.

### Questão 5: **Resposta C**
**Justificativa:** `docker events` é o comando que permite monitorar eventos de containers em tempo real, incluindo mudanças de estado, healthcheck status, start, stop, etc.

### Questão 6: **Resposta C**
**Justificativa:** Alpine Linux é uma distribuição minimalista (~5MB vs ~100MB+), tem menor superfície de ataque (mais seguro), consome menos recursos e é ideal para containers em produção.



**Boa prova! Demonstre todo o conhecimento adquirido durante as aulas.**



**Desenvolvido por:** Professor [Nome] - UniFAAT  
**Versão:** 1.0 - Semestre 2026.1  
**Última atualização:** [Data]