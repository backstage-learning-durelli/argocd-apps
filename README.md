# ArgoCD Applications Repository

Repositório GitOps para gerenciar aplicações no ArgoCD usando o padrão **App of Apps** com ApplicationSet.

## 📋 Estrutura

```
argocd-apps/
├── applicationset.yaml  # ApplicationSet raiz que monitora apps/*
└── apps/                # Diretório com Applications individuais
    ├── app-1/
    │   └── application.yaml
    ├── app-2/
    │   └── application.yaml
    └── ...
```

## 🔄 Como Funciona

1. **ApplicationSet** monitora o diretório `apps/*` neste repositório
2. Para cada subdiretório encontrado em `apps/`, cria automaticamente uma **Application** no ArgoCD
3. Cada Application aponta para o repositório da aplicação específica (pasta `/k8s`)
4. ArgoCD faz sync automático quando detecta mudanças nos manifestos

## 🚀 Fluxo de Deploy

```
Backstage cria novo projeto
    ↓
Cria PR adicionando apps/my-app/application.yaml
    ↓
Merge do PR
    ↓
ApplicationSet detecta nova pasta
    ↓
Cria Application "my-app" no ArgoCD
    ↓
ArgoCD monitora rdurelli/my-app/k8s
    ↓
Deploy automático no Kubernetes
```

## 📦 Adicionar Nova Aplicação

### Via Backstage Template (Recomendado)
O template `java-basic` cria automaticamente um PR neste repo.

### Manualmente
1. Crie uma pasta em `apps/[nome-da-app]/`
2. Adicione `application.yaml`:
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: my-app
     namespace: argocd
   spec:
     project: default
     source:
       repoURL: https://github.com/rdurelli/my-app.git
       targetRevision: HEAD
       path: k8s
     destination:
       server: https://kubernetes.default.svc
       namespace: my-app
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
       syncOptions:
       - CreateNamespace=true
   ```
3. Commit e push

## 🔧 Setup Inicial

1. **Aplicar o ApplicationSet no cluster**:
   ```bash
   kubectl apply -f applicationset.yaml
   ```

2. **Verificar ApplicationSet**:
   ```bash
   kubectl get applicationset -n argocd
   kubectl describe applicationset apps -n argocd
   ```

3. **Ver Applications criadas**:
   ```bash
   kubectl get applications -n argocd
   argocd app list
   ```

## 📚 Documentação

- [ArgoCD ApplicationSet](https://argo-cd.readthedocs.io/en/stable/user-guide/application-set/)
- [App of Apps Pattern](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/)
- [Backstage Software Templates](https://backstage.io/docs/features/software-templates/)
