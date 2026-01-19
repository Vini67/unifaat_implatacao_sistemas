# Lab 15 - Projeto Final: Sistema Completo de E-commerce na AWS
**Disciplina:** Implementação de Software  
**Curso:** Análise e Desenvolvimento de Sistemas - UniFAAT  
**Tipo:** Projeto Final Integrador  
**Valor:** 40% da nota final  
**Prazo:** Entrega na última aula do semestre



## 📋 Informações Gerais do Projeto

### Modalidade
- **Trabalho em grupo**: 2 a 5 pessoas
- **Commits individuais obrigatórios**: Cada membro deve contribuir
- **Acompanhamento**: Professor como membro do repositório

### Entregáveis
1. **Repositório GitHub** completo e funcional
2. **Documentação técnica** no Wiki do repositório
3. **Vídeo demonstrativo** (15-20 minutos)
4. **Vídeo individual** de cada membro (3-5 minutos cada)
5. **Apresentação final** na última aula



## 🎯 Objetivo do Projeto

Desenvolver um **sistema completo de e-commerce** utilizando todos os conceitos aprendidos na disciplina, demonstrando domínio de:

- Infraestrutura como Código
- Alta Disponibilidade e Resiliência
- Monitoramento e Observabilidade
- Segurança e Boas Práticas
- Automação e DevOps



## 🏗️ Arquitetura Obrigatória

### Componentes Mínimos

**Frontend (Obrigatório):**
- Site estático hospedado no **S3** com **CloudFront**
- Domínio personalizado com **Route 53** (opcional)
- Certificado SSL/TLS com **Certificate Manager**

**Backend (Obrigatório):**
- **Application Load Balancer** distribuindo tráfego
- **Auto Scaling Group** com mínimo 2 instâncias
- Instâncias em **múltiplas AZs** para alta disponibilidade
- **Launch Template** com automação via User Data

**Banco de Dados (Obrigatório):**
- **Amazon RDS** (MySQL ou PostgreSQL)
- **Multi-AZ** para alta disponibilidade
- **Backup automático** configurado
- **Security Groups** restritivos

**Monitoramento (Obrigatório):**
- **CloudWatch** com alarmes personalizados
- **SNS** para notificações
- **Dashboard** operacional
- **Logs centralizados**

**Segurança (Obrigatório):**
- **Security Groups** em camadas
- **IAM Roles** com menor privilégio
- **Secrets Manager** para credenciais
- **VPC** com subnets públicas e privadas



## 📝 Especificações Técnicas

### 1. Frontend - Loja Virtual

**Requisitos:**
- **Página inicial** com catálogo de produtos
- **Página de produto** individual
- **Carrinho de compras** (localStorage)
- **Formulário de contato**
- **Design responsivo** (mobile-friendly)

**Tecnologias permitidas:**
- HTML5, CSS3, JavaScript vanilla
- Bootstrap ou framework CSS similar
- Opcional: React, Vue.js, Angular

**Exemplo de estrutura:**
```
frontend/
├── index.html
├── produtos.html
├── produto-detalhes.html
├── carrinho.html
├── contato.html
├── css/
│   └── style.css
├── js/
│   └── app.js
└── images/
    └── produtos/
```

### 2. Backend - API REST

**Requisitos:**
- **API REST** para gerenciar produtos
- **CRUD completo** (Create, Read, Update, Delete)
- **Conexão com RDS** para persistência
- **Health check endpoint** (/health)
- **Logs estruturados**

**Tecnologias permitidas:**
- Node.js (Express)
- Python (Flask/FastAPI)
- Java (Spring Boot)
- PHP (Laravel/Slim)

**Endpoints obrigatórios:**
```
GET    /health           # Health check
GET    /api/produtos     # Listar produtos
GET    /api/produtos/:id # Produto específico
POST   /api/produtos     # Criar produto
PUT    /api/produtos/:id # Atualizar produto
DELETE /api/produtos/:id # Deletar produto
POST   /api/contato      # Formulário de contato
```

### 3. Banco de Dados

**Schema mínimo:**
```sql
-- Tabela de produtos
CREATE TABLE produtos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(255) NOT NULL,
    descricao TEXT,
    preco DECIMAL(10,2) NOT NULL,
    categoria VARCHAR(100),
    imagem_url VARCHAR(500),
    estoque INT DEFAULT 0,
    ativo BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabela de contatos
CREATE TABLE contatos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    assunto VARCHAR(255),
    mensagem TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Dados de exemplo
INSERT INTO produtos (nome, descricao, preco, categoria, estoque) VALUES
('Notebook Dell', 'Notebook Dell Inspiron 15 3000', 2499.99, 'Informática', 10),
('Mouse Logitech', 'Mouse óptico sem fio', 89.90, 'Periféricos', 25),
('Teclado Mecânico', 'Teclado mecânico RGB', 299.99, 'Periféricos', 15);
```



## 🚀 Configuração da Infraestrutura

### Passo 1: Preparação do Repositório

```bash
# Criar repositório no GitHub
# Nome sugerido: ecommerce-aws-unifaat

# Estrutura obrigatória:
ecommerce-aws-unifaat/
├── README.md
├── frontend/
├── backend/
├── database/
│   └── schema.sql
├── infrastructure/
│   ├── user-data.sh
│   └── cloudformation/ (opcional)
├── monitoring/
│   └── dashboard-config.json
├── docs/
│   └── architecture.md
└── scripts/
    ├── deploy.sh
    └── cleanup.sh
```

### Passo 2: Configuração de Membros

**Adicionar colaboradores:**
1. Settings → Manage access → Invite a collaborator
2. **Obrigatório**: Adicionar `AleTavares` como membro
3. Adicionar todos os membros do grupo

**Configurar proteção de branch:**
1. Settings → Branches → Add rule
2. Branch name pattern: `main`
3. ✅ Require pull request reviews before merging
4. ✅ Require status checks to pass before merging

### Passo 3: Infraestrutura AWS

**Ordem de criação:**
1. **VPC e Subnets** (opcional - pode usar default)
2. **Security Groups** em camadas
3. **RDS** com Multi-AZ
4. **Launch Template** com User Data
5. **Application Load Balancer**
6. **Auto Scaling Group**
7. **S3** para frontend
8. **CloudFront** para CDN
9. **CloudWatch** para monitoramento



## 📊 Critérios de Avaliação

### Infraestrutura (25 pontos)

**Configuração AWS (15 pontos):**
- [ ] RDS configurado e funcionando (3 pts)
- [ ] ALB + ASG com alta disponibilidade (4 pts)
- [ ] S3 + CloudFront para frontend (3 pts)
- [ ] Security Groups bem configurados (3 pts)
- [ ] Monitoramento com CloudWatch (2 pts)

**Automação (10 pontos):**
- [ ] Launch Template com User Data (3 pts)
- [ ] Scripts de deploy automatizado (3 pts)
- [ ] Health checks funcionando (2 pts)
- [ ] Backup automático configurado (2 pts)

### Desenvolvimento (20 pontos)

**Frontend (10 pontos):**
- [ ] Interface funcional e responsiva (4 pts)
- [ ] Integração com API backend (3 pts)
- [ ] Design profissional (3 pts)

**Backend (10 pontos):**
- [ ] API REST completa (4 pts)
- [ ] Conexão com banco de dados (3 pts)
- [ ] Tratamento de erros (3 pts)

### Documentação (15 pontos)

**README.md (8 pontos):**
- [ ] Descrição clara do projeto (2 pts)
- [ ] Instruções de instalação (2 pts)
- [ ] Guia de uso da API (2 pts)
- [ ] Informações dos membros (2 pts)

**Wiki do Repositório (7 pontos):**
- [ ] Arquitetura do sistema (2 pts)
- [ ] Guia de deploy (2 pts)
- [ ] Troubleshooting (2 pts)
- [ ] Lições aprendidas (1 pt)

### Apresentação (15 pontos)

**Vídeo Demonstrativo (10 pontos):**
- [ ] Demonstração completa do sistema (4 pts)
- [ ] Explicação da arquitetura (3 pts)
- [ ] Qualidade técnica do vídeo (3 pts)

**Vídeos Individuais (5 pontos):**
- [ ] Cada membro explica sua contribuição (2 pts)
- [ ] Aprendizados e dificuldades (2 pts)
- [ ] Qualidade da apresentação (1 pt)

### Colaboração (10 pontos)

**Git e GitHub (10 pontos):**
- [ ] Commits individuais de todos os membros (4 pts)
- [ ] Mensagens de commit descritivas (2 pts)
- [ ] Uso de branches e pull requests (2 pts)
- [ ] Issues e project management (2 pts)



## 📹 Especificações dos Vídeos

### Vídeo Demonstrativo (Grupo)

**Duração:** 15-20 minutos  
**Formato:** MP4, 1080p mínimo  
**Conteúdo obrigatório:**

1. **Introdução (2 min):**
   - Apresentação dos membros
   - Visão geral do projeto

2. **Demonstração do Sistema (8 min):**
   - Frontend funcionando
   - API sendo consumida
   - Banco de dados com dados
   - Monitoramento em ação

3. **Arquitetura AWS (5 min):**
   - Explicação dos componentes
   - Alta disponibilidade
   - Segurança implementada

4. **Desafios e Soluções (3 min):**
   - Principais dificuldades
   - Como foram superadas

5. **Conclusão (2 min):**
   - Aprendizados do grupo
   - Próximos passos

### Vídeos Individuais

**Duração:** 3-5 minutos cada  
**Formato:** MP4, 720p mínimo  
**Conteúdo obrigatório:**

1. **Apresentação pessoal (30s)**
2. **Sua contribuição específica (2-3 min)**
3. **Maior aprendizado (1 min)**
4. **Maior dificuldade e como superou (1 min)**
5. **Mensagem final (30s)**



## 📚 Documentação Obrigatória

### README.md

```markdown
# E-commerce AWS - UniFAAT ADS

## 👥 Equipe
- **Nome 1** - RA: 123456 - Responsabilidade: Frontend
- **Nome 2** - RA: 234567 - Responsabilidade: Backend
- **Nome 3** - RA: 345678 - Responsabilidade: Infraestrutura
- **Nome 4** - RA: 456789 - Responsabilidade: Banco de Dados
- **Nome 5** - RA: 567890 - Responsabilidade: Monitoramento

## 🏗️ Arquitetura

[Diagrama da arquitetura]

## 🚀 Como Executar

### Pré-requisitos
- AWS CLI configurado
- Conta AWS com Free Tier
- Git instalado

### Deploy da Infraestrutura
```bash
# Clone o repositório
git clone https://github.com/SEU-USUARIO/ecommerce-aws-unifaat
cd ecommerce-aws-unifaat

# Execute o script de deploy
./scripts/deploy.sh
```

### API Endpoints
- `GET /api/produtos` - Lista todos os produtos
- `POST /api/produtos` - Cria novo produto
- `GET /health` - Health check

## 🔗 Links Importantes
- **Frontend**: https://seu-cloudfront-url.com
- **API**: https://seu-alb-dns.com/api
- **Monitoramento**: [Link do Dashboard]

## 📊 Monitoramento
- CloudWatch Dashboard configurado
- Alarmes para CPU, memória e disponibilidade
- Logs centralizados

## 🛠️ Tecnologias Utilizadas
- **Frontend**: HTML5, CSS3, JavaScript
- **Backend**: Node.js/Python/Java/PHP
- **Banco**: Amazon RDS MySQL
- **Infraestrutura**: EC2, ALB, ASG, S3, CloudFront
- **Monitoramento**: CloudWatch, SNS
```

### Wiki do Repositório

**Páginas obrigatórias:**

1. **Home** - Visão geral do projeto
2. **Arquitetura** - Diagrama e explicação detalhada
3. **Guia de Deploy** - Passo a passo completo
4. **API Documentation** - Documentação da API
5. **Troubleshooting** - Problemas comuns e soluções
6. **Lições Aprendidas** - Reflexões do grupo



## 📅 Cronograma de Entrega

### Marco 1 - Semana 2
- [ ] Repositório criado e configurado
- [ ] Membros adicionados
- [ ] Estrutura básica do projeto
- [ ] README inicial

### Marco 2 - Semana 4
- [ ] Frontend básico funcionando
- [ ] Backend com endpoints principais
- [ ] RDS configurado e conectado
- [ ] Primeira versão no S3

### Marco 3 - Semana 6
- [ ] ALB + ASG configurados
- [ ] CloudFront funcionando
- [ ] Monitoramento básico
- [ ] Testes de alta disponibilidade

### Marco 4 - Semana 8 (Final)
- [ ] Sistema completo funcionando
- [ ] Documentação finalizada
- [ ] Vídeos gravados e editados
- [ ] Apresentação preparada



## 🚨 Alertas Importantes

### Custos AWS
- **Monitore constantemente** os custos
- **Use apenas recursos Free Tier** quando possível
- **Configure billing alarms** obrigatoriamente
- **Desligue recursos** quando não estiver trabalhando

### Segurança
- **Nunca commite credenciais** no Git
- **Use IAM roles** ao invés de access keys
- **Configure Security Groups** restritivos
- **Ative MFA** na conta root

### Colaboração
- **Commits frequentes** e descritivos
- **Code review** obrigatório via pull requests
- **Comunicação constante** entre membros
- **Divisão clara** de responsabilidades



## 🎯 Dicas de Sucesso

### Técnicas
1. **Comece simples** e evolua gradualmente
2. **Teste cada componente** isoladamente
3. **Documente conforme desenvolve**
4. **Use tags** para organizar recursos AWS
5. **Implemente logging** desde o início

### Colaboração
1. **Defina padrões** de código e commit
2. **Use issues** para rastrear tarefas
3. **Faça reuniões** regulares de alinhamento
4. **Compartilhe conhecimento** entre membros
5. **Ajude colegas** com dificuldades

### Apresentação
1. **Pratique** a demonstração várias vezes
2. **Prepare** para possíveis falhas técnicas
3. **Seja claro** e objetivo nas explicações
4. **Mostre** o valor técnico do projeto
5. **Demonstre** aprendizado real



## 📞 Suporte

### Canais de Comunicação
- **Issues do GitHub**: Para dúvidas técnicas
- **E-mail do professor**: Para questões administrativas
- **Discord da turma**: Para discussões rápidas

### Office Hours
- **Terças e quintas**: 14h-16h
- **Sábados**: 9h-11h (agendamento prévio)



## 🏆 Critério de Excelência

Para nota máxima, o projeto deve:

- **Funcionar perfeitamente** em demonstração ao vivo
- **Implementar todos** os requisitos obrigatórios
- **Demonstrar domínio** dos conceitos da disciplina
- **Ter documentação** profissional e completa
- **Mostrar colaboração** efetiva entre membros
- **Apresentar soluções** criativas para desafios
- **Seguir boas práticas** de desenvolvimento e DevOps



**Boa sorte! Este projeto é sua oportunidade de demonstrar todo o conhecimento adquirido na disciplina e se preparar para o mercado de trabalho! 🚀**



**Desenvolvido por:** Professor Alexandre Tavares - UniFAAT  
**Versão:** 1.0 - Semestre 2026.1  
**Última atualização:** [Data]