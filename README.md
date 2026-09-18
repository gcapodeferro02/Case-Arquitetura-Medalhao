# Case: construcao de uma arquitetura medalhao com Python

Este repositorio documenta a construcao de um pipeline de dados em camadas Bronze, Silver e Gold. O objetivo do case e mostrar as decisoes de engenharia, o fluxo de processamento e os cuidados necessarios para transformar dados operacionais em dados confiaveis para analise.

## Contexto

O ponto de partida era um conjunto de rotinas de extracao, tratamento e disponibilizacao executadas no mesmo projeto. A organizacao em camadas foi adotada para separar responsabilidades, facilitar a manutencao e permitir que cada etapa fosse validada de forma independente.

## Arquitetura

![Fluxo de atualização e arquitetura do pipeline](assets/fluxo-atualizacao-real.png)

## Como a estrutura foi criada

### 1. Bronze: preservar a origem

A primeira camada concentra as rotinas de extracao e carga inicial. Seu principio e manter rastreabilidade: os dados chegam da origem, recebem o minimo de tratamento necessario para persistencia e ficam disponiveis para reprocessamento historico.

Responsabilidades:

- conectar-se as fontes por configuracao externa;
- extrair dados atuais e historicos;
- persistir arquivos ou tabelas de entrada;
- registrar o periodo processado e eventuais falhas.

### 2. Silver: transformar em dados confiaveis

A segunda camada aplica padronizacao, limpeza, tipagem e regras de negocio. E aqui que os dados deixam de ser apenas uma copia da origem e passam a ter um contrato mais estavel para os consumidores.

Responsabilidades:

- normalizar nomes e tipos de colunas;
- tratar nulos e duplicidades;
- validar chaves e datas;
- consolidar conjuntos relacionados;
- gerar saídas consistentes para a camada Gold.

### 3. Gold: disponibilizar valor

A camada Gold organiza os dados tratados para consumo. Ela pode expor servicos, tabelas analiticas, endpoints ou rotinas de automacao, de acordo com a necessidade do negocio.

Responsabilidades:

- disponibilizar dados prontos para consumo;
- reduzir a complexidade para BI e automacoes;
- aplicar filtros e agregacoes de uso recorrente;
- monitorar a execução do fluxo.

## Ordem de processamento

```text
Fonte -> Bronze -> Silver -> Gold -> Consumidores
```

Cada etapa deve terminar com sucesso antes da seguinte começar. Em uma evolucao do projeto, essa dependência pode ser orquestrada por um scheduler, com logs, retries e alertas.

## Operacao observada

As capturas abaixo mostram a arquitetura em execucao: deployments organizados por camada, agendamento recorrente e historico de runs concluídas.

### Deployments por camada

![Deployments das camadas Bronze e Silver](assets/deployments.png)

### Runs do pipeline

![Execucoes recentes do pipeline](assets/runs.png)

## Decisoes de engenharia

| Decisao | Motivo |
| --- | --- |
| Separar o pipeline em tres camadas | Reduz acoplamento e facilita testes |
| Manter credenciais fora do codigo | Evita vazamento e permite ambientes diferentes |
| Preservar a Bronze | Permite auditoria e reprocessamento |
| Centralizar transformacoes na Silver | Evita regras duplicadas nos consumidores |
| Entregar dados orientados ao consumo na Gold | Simplifica BI, APIs e automacoes |

## Seguranca e reproducibilidade

O case publico nao contem credenciais, dados reais, arquivos `.env`, dumps, caminhos locais ou configuracoes de infraestrutura. Em um ambiente real, use um gerenciador de segredos, variaveis de ambiente e permissoes de menor privilegio.

Exemplo de configuracao local:

```env
SOURCE_HOST=seu-host
SOURCE_DATABASE=seu-banco
SOURCE_USER=seu-usuario
SOURCE_SECRET_REF=use-um-gerenciador-de-segredos
```

## Resultado

A arquitetura cria uma fronteira clara entre ingestao, qualidade e consumo. Isso torna o fluxo mais observavel, reduz o risco de alterar a origem ao criar uma nova regra e oferece uma base para evoluir de scripts locais para uma orquestracao produtiva.

## Proximos passos

- adicionar testes de qualidade por camada;
- versionar contratos de dados;
- incluir logs estruturados e metricas;
- automatizar a execução com CI/CD;
- substituir arquivos intermediarios por tabelas gerenciadas quando necessário.