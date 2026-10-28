# Auditoria de Viés Étnico-Gênero em Fomento à Pesquisa Científica

## 1. Objetivo e escopo desta etapa

Esta etapa prepara, trata e armazena os dados de bolsas da Universidade Aberta do Brasil (UAB) para análises posteriores sobre a distribuição dos registros e dos valores registrados no campo de pagamento.

O arquivo utilizado tem o nome `br-capes-bolsistas-uab-2013-2021-12-01.xlsx`, mas a inspeção identificou registros referentes ao ano de 2013. Portanto, o escopo da base tratada nesta etapa é **o ano de 2013**, com 270.252 registros.

A base não possui campos explícitos de gênero nem de raça/cor. Assim, a base principal, por si só, não permite concluir a existência de viés étnico ou confirmar a identidade de gênero das pessoas. Qualquer análise baseada em gênero inferido deverá ser descrita como exploratória e acompanhada de suas limitações metodológicas.

## 2. Origem e identificação da versão original

- **Fonte informada:** Portal de Dados Abertos da CAPES.
- **Arquivo original:** `br-capes-bolsistas-uab-2013-2021-12-01.xlsx`.
- **Formato:** XLSX.
- **Tamanho do arquivo:** 12.251.318 bytes.
- **Colunas originais:** 11.
- **Registros lidos pelo pandas:** 270.252.
- **Ano encontrado e selecionado:** 2013.
- **Data de obtenção do arquivo:** não registrada no material disponível.
- **SHA-256 do arquivo incluído no pacote analisado:** `535eea511a9ada0744f08a8ad21f8a1d1d3b32932ea0e3511b902b39f955234b`.

O nome do arquivo e seu hash permitem identificar a versão exata utilizada. Para completar a referência da fonte, registre no relatório ou nesta seção a URL específica da página de download da CAPES, caso ela esteja disponível no material da equipe. O arquivo original está em `dados/original/`.

## 3. Estrutura do projeto

```text
projeto-big-data/
├── dados/
│   ├── original/
│   └── tratados/
├── etl/
│   ├── inspecionar_dados.py
│   ├── selecionar_amostra.py
│   ├── tratar_dados.py
│   ├── testar_conexao.py
│   └── carregar_postgres.py
├── sql/
│   └── schema.sql
├── requirements.txt
└── README.md
```

## 4. Processo de tratamento e justificativas

O tratamento é implementado em `etl/tratar_dados.py`, com Python e pandas. O arquivo resultante é `dados/tratados/bolsistas_2013_tratado.csv`.

| Decisão técnica | Justificativa |
|---|---|
| Ler a planilha XLSX original | Preservar a fonte recebida e permitir a repetição do processo. |
| Verificar a presença das 11 colunas esperadas | Interromper o processamento se o formato da fonte mudar de forma incompatível. |
| Manter apenas registros com `AN_REFERENCIA = 2013` | Delimitar o escopo temporal da análise àquele ano, conforme os dados encontrados no arquivo. |
| Remover espaços nas extremidades e reduzir sequências de espaços nos campos textuais | Padronizar textos e reduzir diferenças causadas apenas por espaços excedentes. |
| Converter mês, ano e código de modalidade para tipos inteiros anuláveis | Representar campos numéricos com tipos adequados e permitir identificar valores que não possam ser convertidos. |
| Validar mês entre 1 e 12 e ano igual a 2013 | Verificar a consistência temporal dos registros incluídos. |
| Verificar campos essenciais ausentes e valores de pagamento ausentes ou negativos | Identificar problemas que podem prejudicar análises posteriores. O script informa essas ocorrências; não deve ser interpretado como prova de que todos os problemas possíveis foram eliminados. |
| Contar duplicatas exatas sem removê-las automaticamente | Evitar excluir registros sem investigar se representam duplicidade indevida ou ocorrências legítimas. |
| Remover `NM_BOLSISTA` e `CPF_BOLSISTA` da base analítica exportada | Reduzir a exposição de dados pessoais que não são necessários para as análises atuais. |
| Exportar o resultado para CSV | Disponibilizar uma saída simples, portátil e reproduzível para a carga no banco. |

O script não altera a unidade ou a periodicidade de `VL_BOLSISTA_PAGAMENTO`. A interpretação desse campo deverá ser confirmada na documentação da fonte antes de afirmar que representa salário, valor mensal efetivamente recebido ou outra medida específica.

## 5. Validações registradas

Na execução documentada do ETL e da carga no PostgreSQL, foram registrados os seguintes resultados:

- Registros lidos: 270.252.
- Registros selecionados para 2013: 270.252.
- Registros no CSV tratado: 270.252.
- Colunas no CSV tratado: 9.
- Meses inválidos: 0.
- Registros de outros anos na saída: 0.
- Pagamentos ausentes ou negativos: 0.
- Duplicatas exatas identificadas: 0.
- Registros carregados no PostgreSQL: 270.252.
- IDs distintos no PostgreSQL: 270.252.

Esses resultados correspondem às validações implementadas nos scripts e às consultas executadas. Eles não substituem a confirmação da semântica dos campos na documentação original da CAPES.

## 6. Dicionário de dados da base tratada

Arquivo: `dados/tratados/bolsistas_2013_tratado.csv`.

| Campo no CSV | Tipo no PostgreSQL | Descrição documentada |
|---|---|---|
| `NM_ENTIDADE_ENSINO` | `TEXT NOT NULL` | Nome da instituição de ensino. |
| `ME_REFERENCIA` | `SMALLINT NOT NULL` | Mês de referência; restrição de 1 a 12. |
| `AN_REFERENCIA` | `SMALLINT NOT NULL` | Ano de referência; restrição igual a 2013 nesta tabela. |
| `SG_DIRETORIA` | `VARCHAR(20)` | Sigla da diretoria. |
| `SG_SISTEMA_ORIGEM` | `VARCHAR(20)` | Sigla do sistema de origem. |
| `CD_MODALIDADE_SGB` | `INTEGER NOT NULL` | Código da modalidade no sistema de origem. |
| `DS_MODALIDADE_PAGAMENTO` | `TEXT NOT NULL` | Descrição da modalidade de pagamento. |
| `CD_MOEDA` | `VARCHAR(10) NOT NULL` | Código ou representação da moeda fornecida pela fonte. |
| `VL_BOLSISTA_PAGAMENTO` | `NUMERIC(12,2) NOT NULL` | Valor registrado no campo de pagamento; unidade e periodicidade ainda precisam de confirmação documental. |

O banco também possui o campo técnico `id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY`, criado no PostgreSQL para identificar unicamente cada linha armazenada. Esse campo não faz parte das nove colunas do CSV tratado.

## 7. Modelagem e armazenamento

- **SGBD:** PostgreSQL 18.
- **Provedor de hospedagem:** Aiven.
- **Banco:** `defaultdb`.
- **Tabela:** `public.bolsistas_2013`.
- **Arquivo de definição do esquema:** `sql/schema.sql`.
- **Quantidade carregada e validada:** 270.252 registros.

A modelagem atual utiliza uma tabela analítica única porque os dados tratados são armazenados como registros de bolsas com atributos de instituição, referência temporal, sistema de origem, modalidade, moeda e valor. O campo `id` é a chave primária técnica. Não há relacionamentos entre tabelas nesta versão, pois o esquema definido contém apenas `bolsistas_2013`.

O esquema define restrições para o mês, o ano e valores de pagamento não negativos. Também cria índices para instituição, código de modalidade e mês, campos que podem ser úteis em consultas analíticas. A adequação desses índices poderá ser reavaliada conforme as consultas utilizadas nas próximas etapas.

A carga é executada por `etl/carregar_postgres.py`. O script verifica a quantidade de linhas e as colunas esperadas, cancela a carga se a tabela já contiver registros e confere a quantidade ao final. As credenciais são lidas de variáveis de ambiente; não devem ser gravadas no código nem no repositório.

## 8. Como reproduzir o processo localmente

Os comandos abaixo são para PowerShell, a partir da pasta raiz do projeto. É necessário ter Python instalado e configurar as variáveis de ambiente de conexão ao PostgreSQL antes de testar a conexão ou carregar os dados.

Instalar as dependências:

```powershell
python -m pip install -r requirements.txt
```

Inspecionar a fonte:

```powershell
python .\etl\inspecionar_dados.py
```

Executar o tratamento:

```powershell
python .\etl\tratar_dados.py
```

Testar a conexão ao PostgreSQL, depois de configurar as variáveis de ambiente:

```powershell
python .\etl\testar_conexao.py
```

Criar a tabela executando `sql/schema.sql` no cliente SQL conectado ao banco e, em seguida, carregar os dados:

```powershell
python .\etl\carregar_postgres.py
```

O script de carga foi projetado para cancelar a importação se a tabela já tiver registros. Não o execute novamente sobre uma tabela populada sem antes definir e validar um procedimento de recarga apropriado.

## 9. Privacidade e limitações

As colunas `NM_BOLSISTA` e `CPF_BOLSISTA` são removidas da base analítica final. O arquivo original e a amostra que contém nomes continuam exigindo controle de acesso e não devem ser publicados sem avaliação de privacidade e das regras da atividade.

A base atual não contém campos explícitos de gênero ou raça/cor. A inferência de gênero a partir de nomes não confirma identidade de gênero e pode gerar classificações incorretas. Não é metodologicamente adequado apresentar essa inferência como atributo confirmado. Sem dados adequados para a dimensão étnica, a base atual não sustenta conclusões sobre viés étnico.

Também permanecem pendentes a confirmação documental do significado e da periodicidade do campo de pagamento, a investigação de combinações entre códigos e descrições de modalidade e o registro da URL exata de origem e da data de obtenção do arquivo, se essas informações puderem ser recuperadas.

## 10. Segurança

- Não incluir senhas, tokens ou strings de conexão no repositório.
- Utilizar variáveis de ambiente para credenciais.
- Não compartilhar credenciais em capturas de tela, logs ou mensagens.
- Restringir o acesso ao arquivo original e a arquivos que contenham nomes ou outros dados pessoais.
- Não publicar arquivos identificáveis sem verificar as condições de uso e a necessidade de divulgação.
