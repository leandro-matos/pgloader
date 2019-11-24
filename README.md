
# Migração de banco de dados MYSQL para PostgreSQL

## Ambiente Ruby On Rails

***Esse documento servirá de base para a sua migração. Peço que o leia inteiro antes de começar as atividades. Importe salientar a diversidade de ambiente e que a sua migração será diferente da que eu realizei. Você deve analisar a estrutura do seu projeto e do seu banco de dados e, depois de ler o documento, analisar se servirá para o seu ambiente.*** 

Esse repositorio foi criado com o objeto de passar uma experiencia de migração em ambientes corporativos e facilitar a migração. Desde uma migração simples, onde a base origem não possui muitos dados, até uma grande migração, onde a base origem possui uma grande quantidade de dados. Peço que considere a possibilidade de migrar apenas as tabelas do banco que são essencial para a base do seu projeto, excluindo por exemplo `rails_logs` e entre outras tabelas não importantes.

Após algumas tentativas falhas de migração utilizando o `pgloader`, optamos por utilizar a ferramenta `mysql-to-postgresql`. Os criadores da ferramenta não prestam mais suporte e, por causa disso, é necessário realizar alguns ajustes pontuais no projeto.

## Envoiroment

Necessario ter o ambiente abaixo: 

![Imgur](https://i.imgur.com/rtUuXOl.jpg)

Database MYSQL que contém a sua base de produção. Fique a vontade para criar outras databases para testes.

Database PostgreSQL que receberá a migração de dados. Fique a vontade para criar outras dabases para testes.

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

Crie um arquivo a partir dele

Nesse arquivo você seta a base origem (que será migrada) e a base destrino.

```
├── config
│   └── default.database.yml
```

---

## Clonagem do repositório mysql2postgres

A ferramenta opera apenas com o `Ruby` versão `>= 2.0`. Você pode ter um ambiente com verão antiga, como `1.9.3`, mas terá que instalar, através do seu gerenciador de versões do ruby (`rvm` ou `rbenv`), uma versão 2.0 ou superior para utilizar-la.

#### Apos baixar, passos iniciais:

#### repositório:

`git clone https://github.com/maxlapshin/mysql2postgres`

#### Gemas basicas necessárias do seu projeto:

`"mysql2" >= "0.3.20"`

`"pg" >= "0.18.4"`

#### Dentro do repositorio do mysql2postgres, baixe as dependencias que ele utiliza:

`bundle install`

#### Sete o enconding da base do MYSQL no arquivo `packet.rb` do projeto mysql2postgres para `ascii-8bit`. O arquivo se encontra na estrutura de diretórios do seu gerenciador de versões do Ruby (rvm ou rbenv). Essa configuração serve para que na hora da conversão dos dados, não dê incompatibilidade de enconding das bases.

`find / -name mysql-pr`

`ex: /xpto/etc/rbenv/versions/2.6.1/lib/ruby/gems/2.6.0/gems/mysql-pr-2.9.11/lib/mysql-pr/packet.rb`

#### No inicio do arquivo:

`#encoding: ascii-8bit`

***Possível erro se não setar essa opção:***

`Mysql2psql: Conversion failed: packet is not EOF`