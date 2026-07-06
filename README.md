# 🛰️ ARGO-DEVOPS-3 — GitOps

Repositório **GitOps** do projeto [`registro-star-wars-revolta-dos-clones-devops`](https://github.com/tom-locatelli47/registro-star-wars-revolta-dos-clones-devops), desenvolvido para a disciplina **Fundamentos de DevOps**.

Aqui ficam a **infraestrutura como código (IaC)** usada para provisionar o cluster e os **manifests Kubernetes** sincronizados automaticamente pelo **ArgoCD**.

---

## 📁 Estrutura

```
registro-star-wars-revota-dos-clones-k8s/
  iac/
    terraform/
      main.tf                     # provider AWS, security group e instâncias EC2
      variables.tf                # parâmetros do ambiente (região, AMI, chave SSH...)
      outputs.tf                  # IPs e IDs das instâncias criadas
      terraform.tfvars.example    # modelo de variáveis para execução local
    ansible/
      inventory/hosts.ini         # inventário dos nós do cluster
      group_vars/all.yml          # variáveis (pacotes, usuário admin, chave pública)
      playbooks/prepare-serves.yml# prepara os servidores para o Kubernetes
  k8s/
    namespace.yaml                # namespace registro-atividades-2
    secret.yaml.example           # modelo — copie para secret.yaml local
    postgres-pvc.yaml
    postgres-deployment.yaml
    postgres-service.yaml
    api-deployment.yaml
    api-service.yaml
    argocd-repo-server-network-policy.yaml
    kustomization.yaml            # o CI/CD atualiza a tag da imagem aqui
```

---

## ☁️ Provisionamento da Infraestrutura (Terraform)

Cria a topologia de base na AWS:

- 1 instância **control plane**
- 3 instâncias **worker**

```bash
cd registro-star-wars-revota-dos-clones-k8s/iac/terraform
cp terraform.tfvars.example terraform.tfvars
# edite terraform.tfvars com AMI, key_name e allowed_ssh_cidr

terraform init
terraform apply
```

## ⚙️ Configuração dos Nós (Ansible)

Depois que as instâncias existem, o Ansible prepara cada nó (usuário administrativo, chave SSH, hardening básico e dependências) antes da instalação do Kubernetes:

```bash
cd ../ansible
ansible-playbook -i inventory/hosts.ini playbooks/prepare-serves.yml
```

## 🐳 Cluster K3s + ArgoCD

Com os nós preparados, instale o **K3s** (1 control plane + 3 workers) e, em seguida, o **ArgoCD** no control plane:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

## 🔐 Criar o Secret no Cluster (uma vez, fora do Git)

As credenciais do banco não vão para o repositório — apenas um modelo (`secret.yaml.example`):

```bash
cp k8s/secret.yaml.example k8s/secret.yaml
# edite os valores reais (usuário, senha, banco e SECRET_KEY)

kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/secret.yaml
```

## ✅ Validar os Manifests Localmente

```bash
kubectl apply -k k8s
kubectl get pods -n registro-atividades-2
```

---

## 🔄 Registrar a Aplicação no ArgoCD

Pela CLI/`kubectl`:

```bash
argocd app create registro-atividades \
  --repo https://github.com/tom-locatelli47/ARGO-DEVOPS-3.git \
  --path registro-star-wars-revota-dos-clones-k8s/k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace registro-atividades-2 \
  --sync-policy automated --auto-prune --self-heal
```

Ou pela UI do ArgoCD, informando:

- **Repository URL:** URL deste repositório
- **Path:** `registro-star-wars-revota-dos-clones-k8s/k8s`
- **Namespace:** `registro-atividades-2`
- **Sync Policy:** Automatic + Auto-Prune + Self-Heal

---

## 🌐 Acesso à Aplicação

Os manifests deste repositório expõem a API (`registro-atividades-api-service`) e o PostgreSQL como **ClusterIP** internos ao cluster. Para acessar a API a partir da sua máquina durante testes:

```bash
kubectl port-forward -n registro-atividades-2 svc/registro-atividades-api-service 8001:8001
```

O frontend (`frontend/index.html`, no repositório principal) consome essa API a partir da URL configurada.

---

## 🔁 Fluxo CD Ponta-a-Ponta

1. Push na `main` do repositório principal → GitHub Actions roda os testes.
2. Workflow builda e publica a imagem `tomaslocatelli/registro-star-wars-revolta-dos-clones-devops:<sha>` no Docker Hub.
3. Workflow faz commit **neste** repositório, atualizando `newTag` em `k8s/kustomization.yaml` via Kustomize.
4. **ArgoCD** detecta o commit e sincroniza automaticamente o cluster K3s.

```
git push (app) → GitHub Actions → Docker Hub → bump de tag aqui (GitOps) → ArgoCD → K3s
```

---

## 📦 Organização dos Repositórios

| Repositório | Função |
| --- | --- |
| [`registro-star-wars-revolta-dos-clones-devops`](https://github.com/tom-locatelli47/registro-star-wars-revolta-dos-clones-devops) | Código da aplicação (backend + frontend) e pipeline de CI |
| [`ARGO-DEVOPS-3`](https://github.com/tom-locatelli47/ARGO-DEVOPS-3) | Manifests Kubernetes/Kustomize, IaC (Terraform + Ansible) e ArgoCD (GitOps) |

---

## 👤 Autor

**Tomas Locatelli**
Disciplina: Fundamentos de DevOps
