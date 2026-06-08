# HANDOVER - Projeto GS-DEVOPS-HERMES

Este arquivo resume o estado atual do projeto para continuidade em outro chat.

## 1. Contexto da entrega

A entrega pede uma solucao de cloud computing conectada a Industria Espacial, usando infraestrutura cloud, CI/CD, seguranca, monitoramento e documentacao tecnica.

O projeto escolhido foi o Projeto Hermes, uma plataforma digital simulada para suporte orbital, monitoramento de lixo espacial, classificacao de risco de detritos, simulacao de missoes de captura, taxiamento de satelites, reabastecimento orbital e desorbitagem controlada.

A ideia vem do arquivo Documentacao V2 do Projeto Hermes. O documento define o problema do lixo orbital, a solucao com Modulo de Servico Orbital Universal, Estacao de Abastecimento Orbital, Modulo de Transporte de Combustivel e uma plataforma digital de controle e monitoramento.

## 2. Repositorio GitHub

Repositorio:

```text
ronykenn/GS-DEVOPS-HERMES
```

Branch principal:

```text
main
```

Arquivos criados ate agora:

```text
index.html
styles.css
docs/RELATORIO.md
docs/HANDOVER.md
.github/workflows/deploy.yml
```

Observacao: a criacao do README.md falhou anteriormente por bloqueio da ferramenta, mas isso nao impede o deploy.

## 3. Aplicacao criada

Foi criada uma landing page estatica com dashboard simulado.

Arquivos principais:

```text
index.html
styles.css
```

Conteudo da aplicacao:

- Identidade do Projeto Hermes
- Problema espacial: lixo orbital e risco para satelites ativos
- ODS conectados: ODS 9, ODS 13 e ODS 11
- Explicacao da solucao Hermes
- Fluxo operacional da solucao
- Dashboard simulado com objetos monitorados, detritos criticos, missoes planejadas e combustivel orbital
- Arquitetura AWS proposta
- Equipe

## 4. Arquitetura AWS proposta

Arquitetura simples, barata e adequada para Free Tier:

```text
GitHub Repository
        -> push na main
GitHub Actions CI/CD
        -> OIDC
IAM Role
        -> deploy
S3 Bucket privado
        -> origem protegida
CloudFront
        -> URL publica HTTPS
```

Servicos AWS usados/configurados:

- Amazon S3
- Amazon CloudFront
- AWS IAM Role
- GitHub Actions
- AWS Systems Manager Parameter Store
- AWS KMS via chave gerenciada alias/aws/ssm
- AWS Budget recomendado
- CloudWatch recomendado para monitoramento

Importante: nao foram criadas chaves de acesso AWS do tipo Access Key e Secret Access Key. Isso foi evitado de proposito para reduzir risco de vazamento e seguir uma abordagem mais segura com OIDC.

## 5. Dados AWS conhecidos

Conta AWS:

```text
Account ID: 188776114680
```

Regiao usada:

```text
us-east-2
```

Bucket S3:

```text
gs-devops-hermes-ronykenn
```

CloudFront:

```text
Distribution name: hermes-cloudfront
Distribution ID: E205W33BAJ2N
Distribution domain name: d3i8d8x06no29f.cloudfront.net
Distribution ARN: arn:aws:cloudfront::188776114680:distribution/E205W33BAJ2N
```

IAM Role criada:

```text
Role name: HermesGithubActionsDeployRole
Role ARN: arn:aws:iam::188776114680:role/HermesGithubActionsDeployRole
```

OIDC Provider:

```text
Provider: token.actions.githubusercontent.com
Provider ARN: arn:aws:iam::188776114680:oidc-provider/token.actions.githubusercontent.com
Audience: sts.amazonaws.com
```

Parameter Store:

```text
Name: /hermes/prod/project-secret
Type: SecureString
KMS: alias/aws/ssm
Value usado: HermesSecureConfig2026
```

Observacao: o valor acima foi usado como segredo simulado academico. Nao e uma chave de acesso real. Se quiser melhorar seguranca, trocar depois por um valor novo.

## 6. Workflow GitHub Actions atual

Arquivo:

```text
.github/workflows/deploy.yml
```

Configuracao atual:

```yaml
name: Deploy Hermes Static App to AWS

on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

env:
  AWS_REGION: us-east-2
  S3_BUCKET: gs-devops-hermes-ronykenn
  CLOUDFRONT_DISTRIBUTION_ID: E205W33BAJ2N

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Debug OIDC token claims
        run: |
          python3 - <<'PY'
          import os, json, base64, urllib.request

          request_url = os.environ["ACTIONS_ID_TOKEN_REQUEST_URL"] + "&audience=sts.amazonaws.com"
          request_token = os.environ["ACTIONS_ID_TOKEN_REQUEST_TOKEN"]

          req = urllib.request.Request(request_url, headers={"Authorization": f"Bearer {request_token}"})
          with urllib.request.urlopen(req) as response:
              token = json.loads(response.read().decode())["value"]

          payload = token.split(".")[1]
          payload += "=" * ((4 - len(payload) % 4) % 4)
          claims = json.loads(base64.urlsafe_b64decode(payload.encode()).decode())

          print("OIDC iss:", claims.get("iss"))
          print("OIDC aud:", claims.get("aud"))
          print("OIDC sub:", claims.get("sub"))
          print("OIDC repository:", claims.get("repository"))
          print("OIDC ref:", claims.get("ref"))
          print("OIDC workflow_ref:", claims.get("job_workflow_ref"))
          PY

      - name: Configure AWS credentials using OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::188776114680:role/HermesGithubActionsDeployRole
          aws-region: ${{ env.AWS_REGION }}

      - name: Validate access to Parameter Store
        run: |
          aws ssm get-parameter \
            --name "/hermes/prod/project-secret" \
            --with-decryption \
            --region $AWS_REGION

      - name: Deploy files to S3
        run: |
          aws s3 sync . s3://$S3_BUCKET \
            --delete \
            --exclude ".git/*" \
            --exclude ".github/*" \
            --exclude "docs/*" \
            --exclude "README.md"

      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id $CLOUDFRONT_DISTRIBUTION_ID \
            --paths "/*"
```

Depois que o OIDC funcionar, pode remover a etapa `Debug OIDC token claims` para deixar o workflow mais limpo.

## 7. Claims reais do token OIDC do GitHub Actions

O debug do workflow mostrou:

```text
OIDC iss: https://token.actions.githubusercontent.com
OIDC aud: sts.amazonaws.com
OIDC sub: repo:ronykenn/GS-DEVOPS-HERMES:ref:refs/heads/main
OIDC repository: ronykenn/GS-DEVOPS-HERMES
OIDC ref: refs/heads/main
OIDC workflow_ref: ronykenn/GS-DEVOPS-HERMES/.github/workflows/deploy.yml@refs/heads/main
```

Essas informacoes devem ser usadas para configurar a trust policy da role.

## 8. Problema atual

O GitHub Actions ainda falha na etapa:

```text
Configure AWS credentials using OIDC
```

Erro:

```text
Error: Could not assume role with OIDC: Not authorized to perform sts:AssumeRoleWithWebIdentity
```

Isso significa que o token OIDC esta sendo gerado corretamente, mas a trust policy da role ainda nao esta autorizando a acao `sts:AssumeRoleWithWebIdentity`.

## 9. Trust Policy que deveria funcionar

A trust policy recomendada para a role `HermesGithubActionsDeployRole` e:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::188776114680:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:ronykenn/GS-DEVOPS-HERMES:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

Tambem foi testada uma versao com `job_workflow_ref`, mas o erro continuou.

Versao com `job_workflow_ref`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::188776114680:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:job_workflow_ref": "ronykenn/GS-DEVOPS-HERMES/.github/workflows/deploy.yml@refs/heads/main"
        }
      }
    }
  ]
}
```

## 10. Observacoes importantes sobre o erro

O provider OIDC esta correto conforme print enviado:

```text
Provider: token.actions.githubusercontent.com
Audience: sts.amazonaws.com
ARN: arn:aws:iam::188776114680:oidc-provider/token.actions.githubusercontent.com
```

O workflow tambem esta apontando para a role correta:

```text
arn:aws:iam::188776114680:role/HermesGithubActionsDeployRole
```

O token real do GitHub Actions tambem esta correto:

```text
sub: repo:ronykenn/GS-DEVOPS-HERMES:ref:refs/heads/main
```

Mesmo assim, `sts:AssumeRoleWithWebIdentity` continua negado.

## 11. Proximos passos recomendados

### Opcao A - Criar uma nova role limpa

Como a role atual foi criada pela tela de `Atribuir funcao` do provider, pode haver alguma configuracao gerada ou estado estranho. O proximo passo mais limpo e criar uma role nova manualmente.

Criar nova role:

```text
IAM > Roles > Create role
Trusted entity type: Custom trust policy
```

Nome sugerido:

```text
HermesDeployRole
```

Trust policy inicial:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::188776114680:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
          "token.actions.githubusercontent.com:sub": "repo:ronykenn/GS-DEVOPS-HERMES:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

Depois anexar permissao temporaria para simplificar:

```text
AmazonS3FullAccess
CloudFrontFullAccess
AmazonSSMReadOnlyAccess
```

Depois que funcionar, reduzir para uma policy minima.

Atualizar workflow:

```yaml
role-to-assume: arn:aws:iam::188776114680:role/HermesDeployRole
```

### Opcao B - Recriar provider OIDC

Se a nova role tambem falhar, excluir o provider OIDC e recriar exatamente assim:

```text
Provider URL: https://token.actions.githubusercontent.com
Audience/client ID: sts.amazonaws.com
```

Depois criar a role novamente.

### Opcao C - Alternativa menos ideal: usar Access Key temporaria

Se o prazo estiver apertado, criar um usuario IAM com Access Key e Secret Key, salvar no GitHub Secrets e usar no workflow. Esta opcao funciona, mas e menos segura e deve ser evitada se possivel.

Secrets no GitHub:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

Workflow alternativo:

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-east-2
```

Essa opcao deve ser usada apenas se OIDC nao for resolvido a tempo.

## 12. Policy minima recomendada depois que funcionar

Depois de resolver o OIDC, substituir as permissoes amplas por uma inline policy mais restrita:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3DeployAccess",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::gs-devops-hermes-ronykenn",
        "arn:aws:s3:::gs-devops-hermes-ronykenn/*"
      ]
    },
    {
      "Sid": "CloudFrontInvalidation",
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateInvalidation"
      ],
      "Resource": "*"
    },
    {
      "Sid": "ParameterStoreRead",
      "Effect": "Allow",
      "Action": [
        "ssm:GetParameter"
      ],
      "Resource": "arn:aws:ssm:us-east-2:188776114680:parameter/hermes/prod/project-secret"
    },
    {
      "Sid": "KMSDecryptForParameterStore",
      "Effect": "Allow",
      "Action": [
        "kms:Decrypt"
      ],
      "Resource": "*"
    }
  ]
}
```

## 13. Recursos que deram certo

- Conta AWS criada no Free Tier.
- Budget foi orientado como primeiro passo, mas precisa confirmar se foi criado.
- Bucket S3 criado com acesso publico bloqueado.
- CloudFront criado sem WAF para evitar custo.
- Parameter Store criado como SecureString com `alias/aws/ssm`.
- OIDC provider criado com `token.actions.githubusercontent.com` e audience `sts.amazonaws.com`.
- IAM Role criada: `HermesGithubActionsDeployRole`.
- Aplicacao criada no GitHub.
- Workflow criado e atualizado.
- Debug OIDC confirmou que o token do GitHub esta correto.

## 14. Recursos que ainda precisam ser finalizados

- Resolver `sts:AssumeRoleWithWebIdentity`.
- Fazer o deploy do S3 via GitHub Actions passar.
- Confirmar se o CloudFront abre a aplicacao.
- Criar ou confirmar AWS Budget.
- Criar CloudWatch Alarm simples ou justificar monitoramento com AWS Budget no relatorio.
- Remover o step de debug OIDC do workflow apos resolver.
- Se possivel, trocar permissoes amplas por policy minima.
- Tirar prints finais para a documentacao.

## 15. Checklist final para entrega

Prints recomendados:

```text
1. Repositorio GitHub com arquivos do projeto
2. GitHub Actions com workflow verde
3. Bucket S3 criado e privado
4. CloudFront Distribution criada
5. URL CloudFront abrindo a pagina Hermes
6. IAM Role com trust policy OIDC
7. Provider OIDC token.actions.githubusercontent.com
8. Parameter Store SecureString /hermes/prod/project-secret
9. KMS alias/aws/ssm ou evidenciar que SecureString usa KMS gerenciado
10. AWS Budget criado
11. CloudWatch Alarm ou tela de monitoramento CloudFront
12. Relatorio docs/RELATORIO.md
```

## 16. Nota de seguranca

Nao inserir chaves reais da AWS em arquivos Markdown, no repositorio ou no chat. O ideal e concluir com OIDC. Se usar Access Key por emergencia, armazenar apenas em GitHub Secrets e remover/desativar depois da entrega.
