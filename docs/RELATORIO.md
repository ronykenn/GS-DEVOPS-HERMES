# Relatorio Tecnico - Projeto Hermes

## 1. Visao geral

O Projeto Hermes e uma solucao cloud conectada a Industria Espacial. A aplicacao representa uma plataforma digital para monitoramento de objetos orbitais, classificacao de risco de detritos espaciais, simulacao de missoes e apoio a desorbitagem controlada.

## 2. Problema

A orbita terrestre esta cada vez mais congestionada por satelites desativados, fragmentos de foguetes e residuos de colisoes. Esses objetos podem comprometer satelites ativos usados em comunicacao, navegacao, monitoramento climatico e sensoriamento remoto.

## 3. Solucao

A solucao Hermes simula uma rede de servicos orbitais composta por tres partes principais: modulo de servico orbital universal, estacao de abastecimento orbital e modulo de transporte de combustivel.

## 4. Aplicacao

Foi criada uma pagina web unica contendo identidade do produto, problema espacial, ODS conectados, funcionamento da solucao, dashboard simulado e arquitetura AWS proposta.

## 5. Arquitetura AWS

A arquitetura proposta utiliza GitHub Actions para deploy automatizado, IAM Role com OIDC para evitar credenciais fixas, S3 privado para armazenamento, CloudFront para entrega publica via HTTPS, Parameter Store para parametros sensiveis, KMS para criptografia, AWS Budget para controle de custos e CloudWatch para monitoramento.

## 6. Seguranca

A seguranca foi pensada com S3 privado, IAM Role com menor privilegio, OIDC no GitHub Actions, Parameter Store para evitar hardcode de configuracoes sensiveis e KMS para criptografia.

## 7. Monitoramento

O monitoramento sera feito com AWS Budget para controle financeiro e CloudWatch Alarm para acompanhar erros na distribuicao CloudFront.

## 8. Prints necessarios

- Repositorio GitHub
- GitHub Actions com sucesso
- Bucket S3 privado
- CloudFront criado
- URL publica funcionando
- IAM Role
- Parameter Store
- KMS Key
- AWS Budget
- CloudWatch Alarm
