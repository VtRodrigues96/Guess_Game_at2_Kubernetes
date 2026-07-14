Guess Game - Kubernetes (k3d)
Descrição

Este projeto consiste na reimplementação da aplicação Guess Game, originalmente desenvolvida utilizando Docker Compose, agora utilizando conceitos de Kubernetes através do k3d (K3s dentro de Docker).

A aplicação é composta por:

Backend Flask (Python)
Banco de dados PostgreSQL
Frontend React
NGINX como servidor web e proxy reverso

Nesta versão, a orquestração dos containers foi migrada do Docker Compose para Kubernetes, utilizando:

Deployments
Services
PersistentVolumeClaim (PVC)
Horizontal Pod Autoscaler (HPA)
NodePort
Imagens hospedadas no Docker Hub

O objetivo desta atividade foi aplicar conceitos de Kubernetes para disponibilização, escalabilidade e gerenciamento de uma aplicação distribuída.

Arquitetura da Solução

A arquitetura Kubernetes é composta pelos seguintes componentes:

                 Usuário
                    |
                    |
              NodePort 30080
                    |
                    |
              Frontend React
              (NGINX)
                    |
                    |
              Service Backend
                    |
          ---------------------
          |                   |
      Backend Pod        Backend Pod
          |                   |
          |                   |
          ---- HPA controla ---
                    |
                    |
            PostgreSQL Service
                    |
                    |
             PostgreSQL Pod
                    |
                    |
                 PVC
Componentes Kubernetes Utilizados
Cluster Kubernetes

O ambiente Kubernetes foi criado utilizando:

k3d
k3s
Docker Desktop

O k3d executa um cluster Kubernetes leve utilizando containers Docker.

Versões utilizadas:

k3d version

k3d version v5.9.0

k3s version v1.35.5-k3s1
Estrutura do Projeto
Guess_Game_at2_Kubernetes

├── backend/
├── frontend/
├── nginx/
├── k8s/
│
│── postgres/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── pvc.yaml
│
│── backend/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── hpa.yaml
│
│── frontend/
│   ├── deployment.yaml
│   └── service.yaml
│
├── Dockerfile
├── docker-compose.yml
├── README.md
└── requirements.txt
Imagens Docker

As imagens utilizadas pelo Kubernetes foram previamente construídas e publicadas no Docker Hub.

Dessa forma, não é necessário reconstruir nenhuma imagem durante a execução.

Backend

Imagem:

vitorrodrigues3794/guess-game-backend:latest

Docker Hub:

https://hub.docker.com/r/vitorrodrigues3794/guess-game-backend

Frontend

Imagem:

vitorrodrigues3794/guess-game-frontend:latest

Docker Hub:

https://hub.docker.com/r/vitorrodrigues3794/guess-game-frontend

Pré-requisitos

É necessário possuir instalado:

Docker Desktop
kubectl
k3d

Verificar instalações:

docker --version

kubectl version --client

k3d version
Criando o Cluster Kubernetes

Criar o cluster:

k3d cluster create guess-game

Validar:

kubectl cluster-info

Verificar nodes:

kubectl get nodes

Resultado esperado:

NAME                       STATUS
k3d-guess-game-server-0    Ready
Implantação da Aplicação

Todos os manifestos Kubernetes estão localizados no diretório:

/k8s
1 - Banco PostgreSQL

Aplicar os manifestos:

kubectl apply -f k8s/postgres/

Verificar:

kubectl get pods

Resultado esperado:

postgres-xxxx   Running
Persistência do Banco

O PostgreSQL utiliza um PersistentVolumeClaim:

Arquivo:

k8s/postgres/pvc.yaml

Configuração:

storage: 5Gi

Verificar:

kubectl get pvc

Resultado esperado:

postgres-pvc   Bound
2 - Backend Flask

Aplicação do backend:

kubectl apply -f k8s/backend/

Verificar:

kubectl get pods

Exemplo:

backend-xxxx   Running
backend-yyyy   Running
Configuração do Banco

O backend utiliza as variáveis:

FLASK_DB_TYPE=postgres

FLASK_DB_USER=postgres

FLASK_DB_PASSWORD=secretpass

FLASK_DB_NAME=postgres

FLASK_DB_HOST=postgres

FLASK_DB_PORT=5432

O endereço do banco é resolvido pelo Service Kubernetes:

postgres:5432
Escalabilidade Automática do Backend (HPA)

Foi implementado Horizontal Pod Autoscaler.

Arquivo:

k8s/backend/hpa.yaml

Configuração:

Mínimo de pods: 2

Máximo de pods: 5

CPU alvo: 70%

Verificar:

kubectl get hpa

Exemplo:

NAME          TARGETS      MINPODS MAXPODS
backend-hpa   13%/70%       2       5
Teste de Escalabilidade

Durante teste de carga:

CPU: 253%

O Kubernetes aumentou automaticamente:

2 pods → 5 pods

Após redução da carga:

5 pods → 2 pods

Demonstrando funcionamento do HPA.

3 - Frontend React

Aplicação:

kubectl apply -f k8s/frontend/

Verificar:

kubectl get pods

Resultado esperado:

frontend-xxxx Running
Exposição do Frontend

O frontend foi disponibilizado utilizando:

NodePort

Service:

frontend

Porta:

30080

Verificar:

kubectl get svc

Resultado:

frontend NodePort 80:30080
Acesso à Aplicação
Opção 1 - Port Forward

Executar:

kubectl port-forward service/frontend 8080:80

Acessar:

http://localhost:8080
Opção 2 - NodePort

Acessar:

http://localhost:30080
Utilização do Sistema
Criar jogo

Acesse:

http://localhost:8080

Clique:

Create Game

Informe uma senha.

O sistema retornará:

Game ID
Realizar tentativa

Utilize o Game ID gerado para tentar descobrir a senha.

Consulta ao Banco PostgreSQL

Entrar no container:

kubectl exec -it deployment/postgres -- psql -U postgres -d postgres

Listar tabelas:

\dt

Consultar jogos:

SELECT * FROM game;

Sair:

\q
Monitoramento
Pods
kubectl get pods
Serviços
kubectl get svc
Recursos
kubectl top pods
Logs

Backend:

kubectl logs deployment/backend

Frontend:

kubectl logs deployment/frontend
Benefícios da Migração para Kubernetes
Escalabilidade

O backend pode aumentar ou reduzir automaticamente conforme utilização.

Alta disponibilidade

Múltiplas réplicas do backend permitem continuidade do serviço.

Gerenciamento declarativo

Toda infraestrutura está descrita em YAML.

Portabilidade

O sistema pode ser executado em qualquer cluster Kubernetes compatível.
