# Estrutura do Curso: Implementação de Software

A carga horária será dividida em exposição técnica, laboratórios práticos (mãos na massa) e projeto integrador.  

**Módulo 1: Fundamentos e Ambiente Local (Aulas 1 a 5)**  

O foco aqui é garantir que o aluno entenda o que é um servidor e como empacotar sua aplicação.

  - **Aula 01: Introdução ao Ciclo de Vida de Implantação.**
      - Diferença entre desenvolvimento e operação. Conceitos de On-premise vs. Cloud.
  - **Aula 02: Virtualização e Containers (Docker).**
      - Por que não usamos mais apenas VMs? Criando o primeiro Dockerfile.
  - **Aula 03: Orquestração Local e Persistência.**
      - Docker Compose para multi-serviços (App + Banco de Dados). Redes e Volumes.
  - **Aula 04: Servidores Web e Configuração de Ambiente.**
      - Configurando Nginx como Proxy Reverso e variáveis de ambiente (.env).
  - **Aula 05: Automação de Build Local.**
      - Scripts de automação e verificação de saúde da aplicação (Healthchecks).

**Módulo 2: Avaliação Intermediária (Aulas 6 e 7)**

  - **Aula 06: Revisão Prática Individual.**
      - Checkpoint: Garantir que todos os alunos conseguem rodar o stack completo localmente com Docker.
  - **Aula 07: Prova Teórico-Prática 1.**
      - Objetivo: Realizar o deploy de uma aplicação multicamadas em um ambiente local simulado.

**Módulo 3: Migração e AWS Cloud (Aulas 8 a 13)**  

Aqui entramos no ecossistema AWS, focando na escalabilidade e segurança.

  - **Aula 08: Introdução à AWS e Infraestrutura Global.**
      - Regiões, Zonas de Disponibilidade e o Console AWS. Primeiros passos com IAM (Segurança).
  - **Aula 09: Computação com EC2 e Segurança de Rede.**
      - Provisionamento de instâncias Linux, Security Groups e chaves SSH.
  - **Aula 10: Banco de Dados Gerenciado (RDS) vs. Local.**
      - Migrando o banco de dados do container local para o Amazon RDS.
  - **Aula 11: Armazenamento e Static Hosting (S3).**
      - Hospedagem de Front-end e armazenamento de arquivos (Buckets).
  - **Aula 12: Introdução a Containers na Nuvem (ECS ou Elastic Beanstalk).**
      - Como levar a imagem Docker criada no Módulo 1 para um serviço gerenciado.
  - **Aula 13: CI/CD - O Caminho da Automação.**
      - Conceitos de GitHub Actions ou AWS CodePipeline para deploy automatizado.

**Módulo 4: Consolidação e Encerramento (Aulas 14 e 15)**

  - **Aula 14: Revisão Geral e Workshop de Migração.**
      - Revisão dos conceitos de Cloud, custos (Billing) e segurança. Plantão de dúvidas para o projeto final.
  - **Aula 15: Prova Final / Apresentação de Projeto.**
      - Objetivo: Demonstrar a aplicação rodando na AWS, integrada e automatizada.

**Considerações Pedagógicas:**

1.  **Pré-requisitos:** É fundamental que os alunos já tenham uma aplicação base (em Python, Node ou Java) pronta para ser "deployada".
2.  **Laboratórios:** Todas as aulas (exceto a 1 e as de prova) devem ter 50% de tempo dedicado à prática guiada.
3.  **Custo Zero:** Utilizaremos o *AWS Free Tier* para garantir que os alunos não tenham custos durante o aprendizado, ** mas NÂO podemos garantir, pois o aluno precisa ter uma conta com creditos iniciais e seguir rigorosamente os passos para não ter surpress e acompanhar o relatorio de gastos da AWS**.