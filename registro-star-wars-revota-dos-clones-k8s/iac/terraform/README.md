# Terraform

Infraestrutura base do projeto em AWS para o trabalho final.

Arquivos principais:

- `main.tf`: provider, rede, security group e instâncias EC2.
- `variables.tf`: parâmetros do ambiente.
- `outputs.tf`: IPs e IDs das máquinas criadas.
- `terraform.tfvars.example`: exemplo de variáveis para execução local.

Topologia criada:

- 1 control plane
- 3 workers

Objetivo:

- Provisionar a base para instalar o cluster Kubernetes com Ansible.
- Manter a configuração pronta para o fluxo GitOps com ArgoCD.
