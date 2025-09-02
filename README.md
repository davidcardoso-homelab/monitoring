# monitoring

Projeto de monitoramento do cluster Kubernetes utilizando o kube-prometheus-stack gerenciado pelo ArgoCD.

## Estrutura

- `setup/`
  - `application.yaml`: Configuração do ArgoCD Application para o kube-prometheus-stack.
  - `project.yaml`: Projeto do ArgoCD com permissões e repositórios.
  - `repository.yaml`: Secret com chave SSH para acesso ao repositório Git.
  - `persistent-volumes.yaml`: Definição dos volumes persistentes locais para Prometheus e Grafana.
- `values/values.yaml`: Configurações customizadas do Helm Chart, incluindo senha e URL do Grafana.

## Passos para instalação

1. **Edite as configurações conforme necessário:**
	- `persistent-volumes.yaml`: Defina os diretórios locais para armazenamento dos dados do Prometheus e Grafana.
	- `application.yaml`: Ajuste o `repoURL` se necessário.
	- `project.yaml`: Ajuste o `sourceRepos` conforme seu repositório.
	- `repository.yaml`: Insira sua chave SSH privada para acesso ao Git.
	- `values.yaml`: Defina a senha do Grafana (`adminPassword`) e a URL do Grafana (`root_url`).

2. **Aplique os manifests:**
	```sh
	kubectl apply -f setup/
	```

3. **Atenção com finalizers e hooks:**
	- Devido a um problema de compatibilidade entre o ArgoCD e os hooks do kube-prometheus-stack, pode ser necessário remover manualmente o finalizer do ClusterRoleBinding após a instalação:
	  ```sh
	  kubectl patch clusterrolebinding prometheus-kube-prometheus-admission -p '{"metadata":{"finalizers":[]}}' --type=merge
	  ```
	- O ArgoCD está configurado para ignorar diferenças nesses recursos, mas pode travar na deleção. Se necessário, adicione exclusões no `argocd-cm` para ignorar completamente esses recursos.

4. **Acesse o Grafana:**
	- URL: conforme definido em `values.yaml` (exemplo: http://grafana.homelab)
	- Usuário: admin
	- Senha: conforme definido em `values.yaml`

## Observações

- Não desative os admission webhooks do Prometheus Operator, pois são necessários para o funcionamento correto do stack.
- Sempre garanta que o Prometheus Operator está rodando para evitar travamentos na deleção de recursos.
- Para upgrades do stack, siga as recomendações da documentação oficial do kube-prometheus-stack.