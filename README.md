
# Migração de banco de dados MYSQL para PostgreSQL

Esse repositorio foi criado com o objeto de passar uma experiencia de migração de dados em ambientes corporativos. Desde uma migração simples, onde a base de dados origem não é muito grande, até uma grande migração, onde a base de dados origem possui uma quantidade de dados relevante. Iremos dar exemplo de utilização em um ambiente `Ruby On Rails`. Peço que considere a possibilidade de migrar apenas as tabelas do banco que são essenciais para a base do seu projeto. Se puder excluir por exemplo `rails_logs`, `Eventos` e entre outras tabelas que julga não importantes, o tempo de migração cairá ainda mais.

Utilizamos uma ferramenta chamada ['Pgloader'](https://github.com/dimitri/pgloader).

## Envoiroment

Recomendamos o ambiente abaixo em um hosts apartado do de produção:

***Necessary Ruby libs***

![Imgur](https://i.imgur.com/rtUuXOl.jpg)

1 - Database MYSQL que contém a sua base de produção. Fique a vontade para criar outras databases para testes.

2 - Database PostgreSQL que receberá a migração de dados. Fique a vontade para criar outras dabases para testes.

## Instalação

***Ubuntu:***
apt-get install pgloader

***CentOS:***
baixe o source code do projeto e realize o build do programa. Boa sote

## Uso

`$ pgloader mysql://${user}@${server}/${data_base} postgresql://${user}@${server}/${datbase}`

***Com senha:***

`$ pgloader mysql://${user}:${password}@${server}/${data_base} postgresql://${user}:${password}@${server}/${datbase}`

#### Exemplo de migração com sucesso:

```
2019-11-29T20:48:48.253000Z LOG report summary reset
                                               table name       read   imported     errors      total time
---------------------------------------------------------  ---------  ---------  ---------  --------------
                                          fetch meta data        310        310          0          1.024s
                                           Create Schemas          0          0          0          0.007s
                                         Create SQL Types          0          0          0          0.008s
                                            Create tables        252        252          0          0.640s
                                           Set Table OIDs        126        126          0          0.007s
---------------------------------------------------------  ---------  ---------  ---------  --------------
                            exemple_database.exemple_table    1308497    1308497          0         20.458s
                                exemple_database.agr_texts   20380642   20380642          0        5m6.366s
                                  exemple_database.alertas          2          2          0          1.782s
---------------------------------------------------------  ---------  ---------  ---------  --------------
                                  COPY Threads Completion          4          4          0      21m30.869s
                                           Create Indexes        183        183          0       3m16.090s
                                   Index Build Completion        183        183          0          0.021s
                                          Reset Sequences         81         81          0          0.835s
                                             Primary Keys         81         81          0          0.064s
								      Create Foreign Keys          1          1          0          0.016s
                                          Create Triggers          0          0          0          0.000s
                                         Install Comments          0          0          0          0.000s
---------------------------------------------------------  ---------  ---------  ---------  --------------
                                        Total import time  128176782  128176782          0      21m35.266s

```

## Alter Role

postgres=# `ALTER ROLE ${user} SET search_path = '${database}';`



## Validando de registros:

Criamos um script de contagem de registros utilizando o `ActviveRecord` do `Rails ` em todas as tabelas nos dois bancos de dados.

***Ruby mysqlgem:***

```
@hash = {}
connection = ActiveRecord::Base.connection; nil
	connection.tables.each do |table_name|
	
	@hash[table_name] = connection.execute("select count(*) from #{table_name}").first.first

end
```

***Ruby postgres gem***

```
@hash = {}
connection = ActiveRecord::Base.connection; nil
	connection.tables.each do |table_name|
	
	@hash[table_name] = connection.execute("select count(*) from #{table_name}").first['count'].to_i

end
```

***Com o script abaixo, saberá qual tabela está com registro diferente:***
```
tabelas = []
@hash.each_key do |key|
tabelas << key if @hash[key] != @hash_prod[key]
end
```
