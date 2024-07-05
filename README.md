# Relatório Desafio de Engenharia de Dados
## Introdução
Este documento detalha a execução e implementação da solução para o
Indicium Tech Code Challenge (https://github.com/techindicium/code-challenge).

Autor: Ian Miranda Gomes de Souza - ian.mgsouza@gmail.com

GitHub do Desafio - https://github.com/poxaIan/Indicium-Tech-Code-Challenge

### Tecnologias Usadas:
- Scheduler: **Airflow**

- Data Loader: **Meltano (Baseado em Python)**

- Database: **PostgreSQL**

- Python 3.9 ou inferior

## Configuração do Ambiente

### Aviso Importante
Dependendo da versão do Docker Compose instalada em seu sistema, 
a chamada ao comando pode variar. Certifique-se de verificar se a 
sua versão requer o uso de um hífen ou não.

- Versões mais antigas: O comando é docker-compose (com hífen).
- Versões mais recentes: O comando é docker compose (sem hífen).

Por favor, ajuste seus comandos conforme a versão do Docker Compose que você está utilizando para evitar erros.

### Execute os seguinte comandos para preparar o ambiente:
**Instalação do Meltano:** Ferramenta que facilita o gerenciamento de pipelines de dados.
```
pip install meltano
```

**Construção do Projeto:** Executado para **configurar o ambiente** e instalar dependências adicionais necessárias para o projeto.
```
bash ./build.sh
```

**Inicialização da Aplicação:** Utilizado para inicializar o projeto, configurando os plugins e verificando as dependências.
```
bash ./init.sh
```

**Pipeline executa qualquer dia anterior.** Modifique a data para escolher qualquer dia de interesse.
```
bash ./init.sh 2024-10-06
```

![image](https://github.com/poxaIan/Desafio_Engenharia_Dados/blob/main/Docs/Fluxograma.png)

Fluxograma de como funciona o projeto no geral - Fonte: Autor do Projeto


## Executando cada trabalho individualmente

Será usado a data definida no arquivo .env, não é possível passar um argumento de data aqui

Step 1:
```
meltano run details-to-csv
meltano run postgres-to-csv
```

Step 2:
```
meltano run disk-to-postgres
```


# Resultados das Consultas

## Step 1

Verifique os resultados salvos localmente em: 

```
/data/postgres/{table}/{Data_Escolhida}/{table}.csv
/data/csv/{Data_Escolhida}/order_details.csv
```
Cada tabela existente foi criada uma saída salva local, 
onde foram **extraidas todas as tabelas do banco de dados da origem.**

![image](https://github.com/poxaIan/Desafio_Engenharia_Dados/blob/main/Docs/resultados.png)

Print do Autor do Projeto dos arquivos de saídas salvas localmente.

Verifique os Jobs:

```
meltano run details-to-csv 
meltano run postgres-to-csv
```

O formato .csv foi escolhido pois:

- Simplicidade: Formato de texto fácil de criar, ler e editar.
- Compatibilidade: Suportado por quase todos os softwares de planilhas e bancos de dados.
- Eficiência: Leve, compacto e rápido para transferir e carregar.
- Flexibilidade: Fácil de integrar e personalizar com scripts e ferramentas.
- Popularidade: Amplamente usado em data science e análise de dados.

## Step 2


Dados são carregados dos arquivos locais da etapa 1
para o banco de dados final **dbdata_target**. A execução do job foi:

```
meltano run disk-to-postgres
```

O banco de dados finaliza com todas as tabelas necessárias para a consulta que 
detalha os pedidos. Esta consulta está em `queries/final_goal.sql`, com os resultados 
em `queries/{data_escolhida}/final_goal.csv`. Consulte também outras consultas de exemplo no 
mesmo diretório.

![image](https://github.com/poxaIan/Desafio_Engenharia_Dados/blob/main/Docs/step2.png)

Arquivo final_goal.csv - Fonte: Autor do Projeto

## Considerações Importantes


- **Delimitadores CSV:**
Alguns arquivos têm vírgulas no conteúdo, foram alterados para ponto e vírgula.


- **Configuração do tap-postgres e target-csv-tables:**
Utilizados na etapa 1 para carregar o banco de dados Northwind no disco. 
O extrator recupera apenas o esquema público (`'select: - public-*.*`), 
ignorando information_schema. Tabelas vazias, 
como customer_customer_demo e customer_demographics, não são criadas no disco.


- **Manipulação do Tipo de Dados Real:**
Os campos orders.freight e products.unit_price no banco Northwind, que usam o tipo Real, 
foram mapeados para float usando o meltano-map-transformer. O tipo bpchar é tratado 
como string pelo tap.

### Verificação se ocorrer falha no pipeline
Navegue até o diretório `queries/arquivos` para analisar onde erros podem ocorre na conversão dos arquivos.

![image](https://github.com/poxaIan/Desafio_Engenharia_Dados/blob/main/Docs/queries.png)

Imagem mostrando onde encontrar os arquivos sql convertidos em csv para análise.

A seguir prints dos arquivos salvos localmente e com eles no Postgres:

![image](https://github.com/poxaIan/Desafio_Engenharia_Dados/blob/main/Docs/orders_csv.png) 
Arquivo orders_csv salvo localmente.

![image](https://github.com/poxaIan/Desafio_Engenharia_Dados/blob/main/Docs/orders_south.png)
Tabela orders do Postgres do **segundo banco de dados criado.**






## Airflow

Verificação se o airflow está rodando em segundo plano
```
meltano invoke airflow scheduler
```

Verificação se os agendamentos estão configurados corretamente:

```
meltano invoke airflow dags list
```

Verificação dos próximos 10 tempos de execução para cada dag:

```
meltano invoke airflow dags next-execution -n 10 meltano_load-details-csv_details-to-csv
meltano invoke airflow dags next-execution -n 10 meltano_load-postgres-csv_postgres-to-csv 
meltano invoke airflow dags next-execution -n 10 meltano_load-disk-postgres_disk-to-postgres
```

## Desafios Enfrentados
- **Configuração Inicial:** A configuração inicial do ambiente, incluindo a 
instalação do Meltano e a configuração do Docker, apresentou alguns desafios, 
especialmente em relação à compatibilidade de versões.


- **Compatibilidade do Docker Compose:** Foi necessário ajustar os arquivos de script 
para garantir a compatibilidade com a versão do Docker Compose utilizada.


- **Conflitos de Porta:** Houve a necessidade de gerenciar conflitos de porta com outros 
serviços em execução no sistema.


- **Padrões de Nome de Arquivo:** O script organiza_details.py apresentou dificuldades ao lidar com padrões de nome de arquivo, exigindo ajustes para garantir que os arquivos fossem processados corretamente.

## Conclusão
Apesar dos desafios enfrentados, a solução foi **implementada com sucesso.** 
Os dados foram extraídos de arquivos locais, transformados conforme necessário 
e carregados no banco de dados PostgreSQL conforme os requisitos do desafio. 
Este projeto proporcionou uma experiência valiosa no uso de ferramentas de ETL 
e orquestração de dados, como Meltano e Docker, e demonstrou a eficácia dessas 
ferramentas na automação de processos de engenharia de dados.