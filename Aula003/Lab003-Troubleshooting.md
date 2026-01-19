# Troubleshooting Lab 3 - Docker Básico

**Lab:** 003 - Docker Básico  
**Foco:** Containerização, Dockerfile, imagens

---

## 🚨 Problemas Mais Comuns

### 1. Docker não Instala

#### **Sintoma:**
```bash
docker --version
# Command not found
```

#### **Soluções:**
```bash
# Instala Docker no Ubuntu
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Adiciona usuário ao grupo docker
sudo usermod -aG docker $USER
newgrp docker
```

### 2. Permission Denied no Docker

#### **Sintoma:**
```bash
docker run hello-world
# permission denied while trying to connect to Docker daemon
```

#### **Soluções:**
```bash
# Verifica se está no grupo docker
groups $USER

# Reinicia serviço Docker
sudo systemctl restart docker

# Relogar ou usar newgrp
newgrp docker
```

### 3. Build Falha

#### **Sintoma:**
- "No such file or directory" durante build
- "COPY failed"

#### **Soluções:**
```bash
# Verifica contexto do build
ls -la

# Build com contexto correto
docker build -t minha-app .

# Verifica Dockerfile
cat Dockerfile
```

### 4. Container não Inicia

#### **Sintoma:**
```bash
docker ps
# Container não aparece na lista
```

#### **Diagnóstico:**
```bash
# Verifica containers parados
docker ps -a

# Verifica logs
docker logs nome-container

# Inicia container interativo para debug
docker run -it minha-imagem /bin/bash
```

---

**Desenvolvido por:** Professor Alexandre Tavares - UniFAAT