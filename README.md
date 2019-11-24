
# Migração de banco de dados MYSQL para PostgreSQL

## Ambiente Ruby On Rails

***Esse documento servirá de base para a sua migração, aplicável apenas em ambiente Ruby On Rails. Peço que o leia inteiro antes de começar as atividades. Importe salientar a diversidade de ambiente e que a sua migração será diferente da que eu realizei. Você deve analisar a estrutura do seu projeto e do seu banco de dados e, depois de ler o documento, analisar se servirá para o seu ambiente.***

Esse repositorio foi criado com o objeto de passar uma experiencia de migração de dados em ambientes corporativos. Desde uma migração simples, onde a base de dados origem não é muito grande, até uma grande migração, onde a base de dados origem possui uma quantidade de dados relevante. Peço que considere a possibilidade de migrar apenas as tabelas do banco que são essenciais para a base do seu projeto. Se puder excluir por exemplo `rails_logs` e entre outras tabelas não importantes, o tempo de migração cairá.

Após algumas tentativas falhas de migração utilizando o `pgloader`, optamos por utilizar a ferramenta `mysql-to-postgresql`. Os criadores da ferramenta não prestam mais suporte e, por causa disso, é necessário realizar alguns ajustes pontuais no projeto.

## Envoiroment

Necessario ter o ambiente abaixo. Extramamente interessante ter um novo ambiente para migrar o seu banco de dados: 

![Imgur](https://i.imgur.com/rtUuXOl.jpg)

1 - Database MYSQL que contém a sua base de produção. Fique a vontade para criar outras databases para testes.

2 - Database PostgreSQL que receberá a migração de dados. Fique a vontade para criar outras dabases para testes.

#### Estrutura do projeto:

```
.
├── Gemfile
├── MIT-LICENSE
├── README.md
├── Rakefile
├── bin
│   └── mysqltopostgres
├── config
│   └── default.database.yml
├── lib
│   ├── mysql2psql
│   │   ├── config.rb
│   │   ├── config_base.rb
│   │   ├── connection.rb
│   │   ├── converter.rb
│   │   ├── errors.rb
│   │   ├── mysql_reader.rb
│   │   ├── postgres_db_writer.rb
│   │   ├── postgres_file_writer.rb
│   │   ├── postgres_writer.rb
│   │   ├── version.rb
│   │   └── writer.rb
│   └── mysqltopostgres.rb
├── mysqltopostgres.gemspec
└── test
    ├── fixtures
    │   ├── config_all_options.yml
    │   └── seed_integration_tests.sql
    ├── integration
    │   ├── convert_to_db_test.rb
    │   ├── convert_to_file_test.rb
    │   ├── converter_test.rb
    │   ├── mysql_reader_base_test.rb
    │   ├── mysql_reader_test.rb
    │   └── postgres_db_writer_base_test.rb
    ├── lib
    │   ├── ext_test_unit.rb
    │   └── test_helper.rb
    └── units
        ├── config_base_test.rb
        ├── config_test.rb
        └── postgres_file_writer_test.rb
```

#### Binário:

```
├── bin
│   └── mysqltopostgres
```

***Na hora da operação, o binário gera um arquivo de log `.sql` com todas as operações que ele está realizando.***

#### Database YAML:

***Crie um arquivo a partir dele***

Nesse arquivo você seta a base origem (que será migrada) e a base destrino.

```
├── config
│   └── default.database.yml
```

Indicação de `database.yml`:

```
mysql2psql:
  mysql:
    hostname: localhost
    port: 3306
    username: root
    socket: /var/lib/mysql/mysql.sock
    password: YYXXNN
    database: XPTO # DATABASE_TO_MIGRATE

  destination:
    production:
      encoding: utf8
      hostname: localhost
      port: 5432
      username: postgres
      password: YYXXNN
      database: XPTO # MIGRATED_DATABASE

    preserve_order: true
    remove_dump_file: true
    dump_file_directory: /tmp
    report_status: json
```

---

## Clonagem do repositório mysql2postgres

A ferramenta opera apenas com o `Ruby` versão `>= 2.0`. Você pode ter um ambiente com verão antiga, como `1.9.3`, mas terá que instalar, através do seu gerenciador de versões do ruby (`rvm` ou `rbenv`), uma versão 2.0 ou superior para utilizar-la.

#### Apos baixar, passos iniciais:

#### repositório:

`git clone https://github.com/maxlapshin/mysql2postgres`

#### Gemas necessárias seu projeto rails:

`"mysql2" >= "0.3.20"`

`"pg" >= "0.18.4"`

#### Dentro do repositorio do mysql2postgres, baixe as dependencias que ele utiliza:

`bundle install`

`gem build mysqltopostgres.gemspec`

`gem install mysqltopostgres-0.3.0.gem`

#### Sete o enconding da base do MYSQL no arquivo `packet.rb` do projeto mysql2postgres para `ascii-8bit`. O arquivo se encontra na estrutura de diretórios do seu gerenciador de versões do Ruby (rvm ou rbenv). Essa configuração serve para que na hora da conversão dos dados, não dê incompatibilidade de enconding das bases.

`find / -name mysql-pr`

`ex: /xpto/etc/rbenv/versions/2.6.1/lib/ruby/gems/2.6.0/gems/mysql-pr-2.9.11/lib/mysql-pr/packet.rb`

#### No inicio do arquivo:

`#encoding: ascii-8bit`

***Possível erro na hora da migração se não setar essa opção:***

`Mysql2psql: Conversion failed: packet is not EOF`

## Atividades

A partir daqui você tem duas opções. Caso a sua base seja pequena:

***Obs: Sempre alterne o conector do banco no Gemfile para interagir com o `MYSQL` ou com o `PostgreSQL`. Caso esteja utilizando duas versões de `Ruby`, também fique atento em alternar entre o antigo e o novo.***

#### Small Bases:

**1 - Crie uma base no `PostgreSQL` para receber os dados da base de produção do `MYSQL`.**

**2 - No banco `PostgreSQL`:**

`RAILS_ENV=production bundle exec rake db:create`

***Não rode migration***: 

`RAILS_ENV=production bundle exec rake db:migrate`

***Deixe que a migração crie as tabelas, relcionamentos entre tabelas e as keys. Pode rodar a migration no banco MYSQL, caso esteja trabalhando com dump e tal***

***Possível erro se rodar a migration:*** 

`Mysql2psql: Conversion failed: Error: relation "xpto_key" already exists`

***Quer dizer que o mysql2postgres está tentando criar novamente uma relação que já existe.***

**3 - Entre no diretório do binário e o execute:**

`mysqltopostgres ../config/your_database.yml`

**4 - Aguarde a migração terminar**