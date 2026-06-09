# Projeto Hermes

## Equipe
- Aksel Viktor Caminha Rae (RM: 99011)
- Ian Xavier Kuraoka (RM: 98860)
- Rony Ken Nagai (RM: 551549)
- Tomáz Versolato Carballo (RM: 551417)

## Descrição do Projeto
O **Projeto Hermes** é uma solução de infraestrutura digital voltada para suporte orbital e manutenção de satélites. O objetivo principal é lidar com o **lixo orbital**, oferecer **manutenção e reabastecimento de satélites**, e garantir segurança e sustentabilidade na órbita terrestre.

O projeto integra **Engenharia de Software**, **cloud computing** e arquitetura em AWS, com uma interface web publicada via **S3** e distribuída pelo **CloudFront**.

O sistema é composto por três módulos principais:
1. **Módulo de Serviço Orbital Universal** – execução de inspeção, captura e redirecionamento de detritos e satélites.
2. **Estação de Abastecimento Orbital** – armazenamento de combustível e apoio logístico.
3. **Módulo de Transporte de Combustível** – transporte seguro entre Terra e a estação orbital.

## Tecnologias e Arquitetura
- **Frontend:** HTML, CSS, JS (simples, responsivo)
- **Deploy:** AWS S3 + CloudFront
- **Autenticação de deploy:** IAM Role com OIDC via GitHub Actions
- **Pipeline CI/CD:** GitHub Actions para upload automático de arquivos e invalidação de cache do CloudFront
- **Segurança:** Bucket S3 privado, política de acesso via IAM Role limitado
- **URL Pública:** Distribuição via CloudFront HTTPS
- **Monitoramento:** CloudWatch, AWS Budget (opcional)

### Fluxo de Deploy
1. Commit no repositório GitHub (`main`)
2. GitHub Actions executa workflow:
   - Configuração de credenciais AWS via OIDC
   - Upload dos arquivos para S3
   - Invalidação de cache CloudFront
3. Site publicado em URL pública: `https://d3i8d8x06no29f.cloudfront.net`
4. Atualização automática do cache via workflow

> **Observação:** Durante testes, certifique-se de que o bucket e a distribuição CloudFront estejam corretos e habilitados. Use `Ctrl+F5` para forçar refresh.

## Estrutura do Repositório

/assets
hermes-logo.png
styles.css
index.html
.github/workflows/deploy.yaml
README.md



## Funcionalidades do Site
- Página única com apresentação do projeto
- Exibição da logo e título
- Seções de:
  - Problema
  - Solução Hermes
  - Fluxo operacional
  - Dashboard simulado
  - Arquitetura AWS
  - Equipe
- Responsivo e pronto para deploy via S3 + CloudFront

## Conexão com a Indústria Espacial
O Projeto Hermes se conecta com:
- **ODS 9** – Indústria, Inovação e Infraestrutura
- **ODS 11** – Cidades Sustentáveis
- **ODS 13** – Ação Climática

O projeto apresenta um problema real e aplicável: **redução do risco orbital e suporte a satélites ativos**  .

## Integrações e Deploy AWS
- **S3 Bucket** – armazenamento dos arquivos web (privado)
- **CloudFront** – distribuição global com HTTPS
- **IAM Role HermesDeployRole** – permissão mínima para GitHub Actions
- **GitHub Actions** – CI/CD para deploy automático
- **OIDC** – autenticação sem chave AWS exposta
- **Cache Invalidation** – atualização do site após cada deploy
- **Segurança e Custo:** Free Tier AWS suficiente para a demonstração, sem custos altos, monitorável via AWS Budget

## Considerações Finais
O Projeto Hermes oferece uma **infraestrutura orbital digital** funcional, segura e sustentável, com deploy completo em AWS, integração GitHub Actions e interface web responsiva. Ideal para apresentações acadêmicas ou demonstração prática da Global Solution 2026    .
