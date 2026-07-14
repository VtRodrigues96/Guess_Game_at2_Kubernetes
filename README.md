# Guess Game - Kubernetes (k3d)

## Descrição

Este projeto consiste na implementação do jogo de adivinhação disponibilizado em:

https://github.com/fams/guess_game

Este projeto consiste na migração da aplicação **Guess Game** desenvolvida na atividade anterior utilizando Docker Compose para uma arquitetura baseada em **Kubernetes utilizando k3d**.

A aplicação é composta por:

- Backend Flask (Python)
- Frontend React servido pelo NGINX
- Banco PostgreSQL
- Kubernetes para gerenciamento dos containers
- HPA para escalabilidade automática do backend

As imagens Docker estão disponíveis no Docker Hub, portanto não é necessário realizar o build das imagens para executar o projeto.

---

# Arquitetura

```text
Usuário
   |
   |
NodePort / Port-forward
   |
   |
Frontend React + NGINX
   |
   |
Service Backend
   |
   |
Backend Flask (Pods)
   |
   |
PostgreSQL
   |
   |
Persistent Volume
```

---

# Tecnologias Utilizadas

- Kubernetes
- k3d
- Docker
- Docker Hub
- Flask
- React
- NGINX
- PostgreSQL

---

# Imagens Docker

As imagens utilizadas pelo Kubernetes estão publicadas no Docker Hub:

Backend:

```text
vitorrodrigues3794/guess-game-backend:latest
```

Frontend:

```text
vitorrodrigues3794/guess-game-frontend:latest
```

---

# Estrutura Kubernetes

Os manifestos Kubernetes estão disponíveis no diretório:

```text
/k8s
```

Estrutura:

```text
k8s
├── backend
│   ├── deployment.yaml
│   ├── service.yaml
│   └── hpa.yaml
│
├── frontend
│   ├── deployment.yaml
│   └── service.yaml
│
└── postgres
    ├── deployment.yaml
    ├── service.yaml
    └── pvc.yaml
```

---

# Pré-requisitos

Instalar:

- Docker
- kubectl
- k3d


Verificar:

```bash
docker --version

kubectl version --client

k3d version
```

---

# Criar Cluster Kubernetes

Criar o cluster k3d:

```bash
k3d cluster create guess-game
```

Validar:

```bash
kubectl get nodes
```

Resultado esperado:

```text
k3d-guess-game-server-0   Ready
```

---

# Deploy da Aplicação

## PostgreSQL

Aplicar os manifestos:

```bash
kubectl apply -f k8s/postgres/
```

Validar:

```bash
kubectl get pods
kubectl get pvc
```

O PVC deve estar:

```text
STATUS: Bound
```

---

## Backend

Aplicar:

```bash
kubectl apply -f k8s/backend/
```

Validar:

```bash
kubectl get pods
```

O backend deve possuir inicialmente 2 Pods.

---

## Frontend

Aplicar:

```bash
kubectl apply -f k8s/frontend/
```

Validar:

```bash
kubectl get pods
kubectl get svc
```

---

# Acesso ao Sistema

O frontend está configurado utilizando NodePort.

Verificar:

```bash
kubectl get svc frontend
```

Exemplo:

```text
frontend   NodePort   80:30080/TCP
```

Caso esteja utilizando k3d, acessar utilizando:

```bash
kubectl port-forward service/frontend 8080:80
```

Abrir no navegador:

```
http://localhost:8080
```

---

# Teste da Aplicação

1. Acessar o frontend.
2. Criar um novo jogo.
3. Informar uma senha.
4. Utilizar o Game ID gerado para realizar tentativas.

---

# Banco de Dados

Acessar o PostgreSQL:

```bash
kubectl exec -it deployment/postgres -- \
psql -U postgres -d postgres
```

Listar tabelas:

```sql
\dt
```

Consultar jogos:

```sql
SELECT * FROM game;
```

Sair:

```sql
\q
```

---

# Auto Scaling (HPA)

O backend possui Horizontal Pod Autoscaler configurado.

Configuração:

```text
Mínimo de Pods: 2
Máximo de Pods: 5
CPU alvo: 70%
```

Verificar:

```bash
kubectl get hpa
```

Exemplo:

```text
NAME          REFERENCE            TARGETS
backend-hpa   Deployment/backend   cpu: 20%/70%
```

Durante aumento de carga, o Kubernetes cria novos Pods automaticamente.

---

# Comandos Úteis

Ver todos os Pods:

```bash
kubectl get pods
```

Ver serviços:

```bash
kubectl get svc
```

Ver consumo:

```bash
kubectl top pods
```

Remover aplicação:

```bash
kubectl delete -f k8s/
```

Remover cluster:

```bash
k3d cluster delete guess-game
```
---

# Requisitos Atendidos

| Requisito | Status |
|-|-|
| Kubernetes utilizando k3d | ✔ |
| Aplicação migrada do Docker para Kubernetes | ✔ |
| Imagens disponíveis no Docker Hub | ✔ |
| Frontend acessível diretamente | ✔ |
| NodePort implementado | ✔ |
| Backend com HPA | ✔ |
| PostgreSQL com persistência | ✔ |
| Manifestos dentro da pasta /k8s | ✔ |

---

# Autor

Vitor Rodrigues
