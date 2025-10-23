<img width="800" height="200" alt="Image" src="https://github.com/user-attachments/assets/bc251934-6e76-47f8-99ef-af6788691611" />
<div align="center">
  
# 🚀 Projeto: GitOps na Prática com Kubernetes e ArgoCD

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)
![Rancher](https://img.shields.io/badge/Rancher_Desktop-0075A8?style=for-the-badge&logo=rancher&logoColor=white)
![GitOps](https://img.shields.io/badge/GitOps-FF6B6B?style=for-the-badge&logo=git&logoColor=white)

**Demonstração prática de GitOps utilizando ArgoCD para gerenciar uma aplicação de microserviços no Kubernetes**

</div>

---

## 📋 Sobre o Projeto

Este projeto, desenvolvido como parte do **Programa de Bolsas DevSecOps da Compass UOL**, demonstra a implementação de uma arquitetura completa de **GitOps** utilizando **Kubernetes** e **ArgoCD**.
A aplicação escolhida é a **Online Boutique** (Google Microservices Demo), uma loja de e-commerce composta por 11 microserviços, implantada em um cluster Kubernetes local através do Rancher Desktop.

### 🎯 Objetivo

Implementar um fluxo GitOps onde:
- O repositório Git é a **fonte única da verdade** (Single Source of Truth)
- O ArgoCD monitora continuamente o repositório
- Qualquer alteração nos manifestos é automaticamente aplicada no cluster
- O estado desejado é sempre sincronizado com o estado real

---

## 🛠️ Tecnologias Utilizadas

| Tecnologia | Descrição |
|------------|-----------|
| **Git / GitHub** | Controle de versão e fonte da verdade para os manifestos |
| **Rancher Desktop** | Cluster Kubernetes local e runtime de contêineres |
| **Kubernetes (K8s)** | Orquestrador de contêineres para gerenciar microserviços |
| **ArgoCD** | Ferramenta de GitOps para Sincronização Contínua (CD) |
| **WSL2** | Ambiente Linux no Windows para executar comandos `kubectl` |
| **Online Boutique** | Aplicação de demonstração com 11 microserviços |

---

## 📁 Estrutura do Repositório

```
gitops-kubernetes-argocd/
├── k8s/
│   └── online-boutique.yaml    # Manifestos Kubernetes da aplicação
├── README.md                     # Este arquivo
└── .gitignore
```

---

## 🚦 Pré-requisitos

Antes de iniciar, certifique-se de ter instalado:

- [x] **Rancher Desktop** (com Kubernetes habilitado)
- [x] **kubectl** (ferramenta de linha de comando do Kubernetes)
- [x] **Git** (para clonar o repositório)
- [x] **WSL2** (opcional, para ambiente Linux no Windows)

---

## 📖 Passo a Passo da Implementação

### 1️⃣ Configuração do Ambiente

O primeiro passo foi configurar o ambiente local com o Rancher Desktop, habilitando o Kubernetes e o Docker como runtime de contêineres.

Após a instalação, verificamos se o cluster estava funcionando corretamente:

```bash
kubectl get nodes
```

**Resultado Esperado:**
```
NAME               STATUS   ROLES                  AGE   VERSION
rancher-desktop    Ready    control-plane,master   2d3h    v1.33.5+k3s1
```

<img width="748" height="182" alt="Image" src="https://github.com/user-attachments/assets/029f8601-d5ea-48b2-9326-69bbf9b9b7cc" />


---

### 2️⃣ Preparação do Repositório GitOps (Etapa 1)

Este repositório foi criado para servir como a **"fonte da verdade"** do nosso cluster. 

**Passos executados:**

1. Criação do repositório no GitHub: `gitops-kubernetes-argocd`
2. Criação da estrutura de diretórios: `k8s/`
3. Download do manifesto `kubernetes-manifests.yaml` do projeto [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo)
4. Renomeação e organização do arquivo como `k8s/online-boutique.yaml`

**Estrutura criada:**

<img width="856" height="289" alt="Image" src="https://github.com/user-attachments/assets/e0fad75d-4af5-459a-b621-3403f8c13223" />

---

### 3️⃣ Instalação do ArgoCD (Etapa 2)

O ArgoCD foi instalado no cluster dentro de seu próprio namespace (`argocd`) utilizando o manifesto oficial:

```bash
# Criar namespace do ArgoCD
kubectl create namespace argocd

# Instalar o ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**Verificação da instalação:**

```bash
kubectl get pods -n argocd
```

Aguardamos todos os pods ficarem no estado `Running` (pode levar alguns minutos):

<img width="891" height="210" alt="Image" src="https://github.com/user-attachments/assets/f6f193eb-27fc-441b-b3fd-3b1051e41c93" />

**Acesso à interface do ArgoCD:**

```bash
# Port-forward para acessar a UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**Obter a senha inicial:**

```bash
# Senha padrão (usuário: admin)
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Acesso: `https://localhost:8080`

---

### 4️⃣ Configuração da Aplicação no ArgoCD (Etapa 3)

Com o ArgoCD em execução, configuramos nossa aplicação através da interface web:

**Configurações da Aplicação:**

- **Application Name:** `online-boutique`
- **Project:** `default`
- **Sync Policy:** `Manual` (pode ser `Automatic`)
- **Repository URL:** `https://github.com/Thiago-code-lab/gitops-kubernetes-argocd`
- **Revision:** `HEAD` (branch main/master)
- **Path:** `k8s`
- **Cluster URL:** `https://kubernetes.default.svc`
- **Namespace:** `default`

### Em seguida, verá um card semelhante à esse

<img width="1919" height="952" alt="Image" src="https://github.com/user-attachments/assets/a6aec5ce-9a31-4b13-9de3-c135c33cbcc6" />

### Clique sobre ele e prossiga para o próximo passo

---

### 5️⃣ Sincronização da Aplicação (Etapa 4)

Após criar a aplicação, ela aparecerá com status `OutOfSync`. Clicamos no botão **SYNC** para que o ArgoCD implante todos os recursos no cluster.

O ArgoCD irá:
1. Ler os manifestos do repositório
2. Comparar com o estado atual do cluster
3. Aplicar as mudanças necessárias
4. Monitorar a saúde dos recursos

**Status após sincronização:**

<img width="1919" height="951" alt="Image" src="https://github.com/user-attachments/assets/cc04823b-380b-4a94-8aec-28a9535ca273" />

✅ **Synced:** Estado do Git = Estado do Cluster  
💚 **Healthy:** Todos os pods estão rodando corretamente

---

### 6️⃣ Acesso à Aplicação "Online Boutique" (Etapa 5)

Com a aplicação implantada, fizemos um port-forward para acessar a loja localmente:

```bash
kubectl port-forward svc/frontend-external 9090:80
```

Acessando `http://localhost:9090` no navegador, a loja está funcionando:

<img width="1919" height="990" alt="Image" src="https://github.com/user-attachments/assets/bed18422-2d5c-4bd1-ae2e-e800695a3bbb" />

---

## 🎯 Desafio GitOps: Alterando Réplicas (Opcional)

Para validar o fluxo GitOps na prática, modificamos o número de réplicas do serviço `frontend` diretamente no código:

**Alteração no arquivo `k8s/online-boutique.yaml`:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3  # ← Alterado de 1 para 3
  selector:
    matchLabels:
      app: frontend
  # ... resto do manifesto
```

<img width="1593" height="1030" alt="Image" src="https://github.com/user-attachments/assets/dec686a4-7e69-451b-bb75-3a4be52929f6" />

**Comportamento do ArgoCD:**

1. ⚠️ Detecta que o repositório mudou → Status: `OutOfSync`
2. 🔄 Após clicar em SYNC (ou sync automático)
3. ✅ Aplica as mudanças no cluster
4. 🚀 Cria 2 novos pods do frontend automaticamente

### Mudança percebida por comando, após efetuar o commit com o acréscimo de réplicas do front-end 
<img width="1236" height="622" alt="Image" src="https://github.com/user-attachments/assets/f59465b5-03ba-40c2-b5db-9ff4c88b8bf5" />

### Refresh e Sync pelo próprio argocd, podemos perceber que temos agora 3 réplicas do front-end 
<img width="1919" height="770" alt="Image" src="https://github.com/user-attachments/assets/fb44cb91-5eda-4c4a-8342-97497d892c0f" />

---

## 📊 Arquitetura da Solução

```
┌─────────────────────────────────────────────────────────┐
│                     DESENVOLVEDOR                        │
│          (Commit & Push no GitHub)                      │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                 REPOSITÓRIO GIT                          │
│           (Single Source of Truth)                      │
│                                                          │
│    k8s/online-boutique.yaml                             │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ (ArgoCD monitora)
                     ▼
┌─────────────────────────────────────────────────────────┐
│                    ARGOCD                                │
│           (GitOps Controller)                           │
│                                                          │
│  • Detecta mudanças                                     │
│  • Compara estado desejado vs real                     │
│  • Aplica sincronização                                │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              KUBERNETES CLUSTER                          │
│           (Rancher Desktop)                             │
│                                                          │
│    ┌─────────────────────────────────────────┐         │
│    │      Online Boutique (11 services)      │         │
│    │  frontend | cartservice | productcatalog│         │
│    │  currencyservice | paymentservice | ...  │         │
│    └─────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────┘
```

---

## ✅ Benefícios do GitOps

| Benefício | Descrição |
|-----------|-----------|
| 🔍 **Auditoria** | Todo o histórico de mudanças fica registrado no Git |
| 🔄 **Reversão Fácil** | Basta reverter o commit para voltar ao estado anterior |
| 🤖 **Automação** | Deploy automático ao fazer push no repositório |
| 📦 **Versionamento** | Infraestrutura como código versionada |
| 🔒 **Segurança** | Controle de acesso via Git (PRs, aprovações) |
| 🎯 **Consistência** | Estado desejado sempre sincronizado com o real |

---

## 🧪 Comandos Úteis

### Kubernetes

```bash
# Ver todos os recursos no namespace default
kubectl get all

# Ver logs de um pod
kubectl logs <nome-do-pod>

# Descrever um recurso
kubectl describe pod <nome-do-pod>

# Deletar recursos
kubectl delete -f k8s/online-boutique.yaml
```

### ArgoCD

```bash
# Login via CLI
argocd login localhost:8080

# Listar aplicações
argocd app list

# Sincronizar aplicação
argocd app sync online-boutique

# Ver status da aplicação
argocd app get online-boutique
```

---

## 🐛 Troubleshooting

### Pods não iniciam (CrashLoopBackOff)

```bash
# Verificar logs do pod com problema
kubectl logs <nome-do-pod>

# Verificar eventos do cluster
kubectl get events --sort-by='.lastTimestamp'
```

### ArgoCD não detecta mudanças

- Verifique se o repositório está acessível
- Confirme o path correto (`k8s`)
- Force um refresh: `argocd app get online-boutique --refresh`

### Port-forward desconecta

- Normal após algum tempo de inatividade
- Execute o comando novamente quando necessário

---

## 📚 Referências

- [Documentação Oficial do ArgoCD](https://argo-cd.readthedocs.io/)
- [Online Boutique - Google Microservices Demo](https://github.com/GoogleCloudPlatform/microservices-demo)
- [Rancher Desktop](https://rancherdesktop.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [GitOps Principles](https://www.gitops.tech/)

---

## 👨‍💻 Autor

**Thiago Cardoso Davi**

[![LinkedIn](https://img.shields.io/badge/-LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/analyticsthiagocardoso)
[![GitHub](https://img.shields.io/badge/-GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com/Thiago-code-lab)
[![Email](https://img.shields.io/badge/-Email-D14836?style=flat&logo=gmail&logoColor=white)](mailto:analyticsdev.thiago@gmail.com)

**Programa de Bolsas DevSecOps - Compass UOL**

---

## 📄 Licença

Este projeto é de código aberto e está disponível para fins educacionais.

---

<div align="center">

**⭐ Se este projeto foi útil, considere dar uma estrela no repositório!**

**Feito com 💜 por Thiago Cardoso Davi**

</div>
