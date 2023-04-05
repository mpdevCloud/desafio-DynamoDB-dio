# dio-desafio boas práticas com dynamodb
Repositório para o desafio Boas práticas com DynamoDB

### Serviço utilizado
  - Amazon DynamoDB
  - Amazon CLI para execução em linha de comando

### Comandos para execução do experimento:


- Criar uma tabela

```
aws dynamodb create-table --table-name Livro 
    --attribute-definitions 
        AttributeName=Title,AttributeType=S 
        AttributeName=Author,AttributeType=S 
    --key-schema 
        AttributeName=Title,KeyType=HASH 
        AttributeName=Author,KeyType=RANGE 
    --provisioned-throughput 
        ReadCapacityUnits=10,WriteCapacityUnits=5
```

- Inserir um item

```
aws dynamodb put-item\
	--table-name Livro\
    --item file://bookitem.json \
```

- Inserir múltiplos itens

```
aws dynamodb put-item\
	--table-name Livro --item file://batchbookitems.json
```

- Criar um index global secundário baeado no título do álbum

```
aws dynamodb update-table \
    --table-name Livro \
    --attribute-definitions AttributeName=BookPublisher,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"BookPublisher-index\",\"KeySchema\":[{\"AttributeName\":\"BookPublisher\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"

```

- Criar um index global secundário baseado no nome do artista e no título do álbum

```
aws dynamodb update-table \
    --table-name Livro \
    --attribute-definitions\
        AttributeName=Author,AttributeType=S \
        AttributeName=BookPublisher,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"AuthorBookPublisher-index\",\"KeySchema\":[{\"AttributeName\":\"Author\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"BookPublisher\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"

```

- Criar um index global secundário baseado no título da música e no ano

```
aws dynamodb update-table \
    --table-name Livro \
    --attribute-definitions\
        AttributeName=Title,AttributeType=S \
        AttributeName=NumberPagination,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"TitleNumberPagination-index\",\"KeySchema\":[{\"AttributeName\":\"Title\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"NumberPagination\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"

```

- Pesquisar item por artista

```
aws dynamodb query \
    --table-name Livro \
    --key-condition-expression "Author = :author" \
    --expression-attribute-values  '{":author":{"S":"Machado de Assis"}}'
```
- Pesquisar item por artista e título da música

```
aws dynamodb query \
    --table-name Livro \
    --key-condition-expression "Title = :title and Author = :author" \
    --expression-attribute-values file://keymaps.json
```

- Pesquisa pelo index secundário baseado no título do álbum

```
aws dynamodb query \
    --table-name Livro \
    --index-name BookPublisher-index \
    --key-condition-expression "BookPublisher = :bookpublisher" \
    --expression-attribute-values  '{":bookpublisher":{"S":"Tipografia Nacional"}}'    
```

- Pesquisa pelo index secundário baseado no nome do artista e no título do álbum

```
aws dynamodb query \
    --table-name Livro \
    --index-name AuthorBookPublisher-index \
    --key-condition-expression "Author = :author and BookPublisher = :bookpublisher" \
    --expression-attribute-values  '{":author":{"S":"Machado de Assis"},":bookpublisher":{"S":"Tipografia Nacional"} }'
```

- Pesquisa pelo index secundário baseado no título da música e no ano

```
aws dynamodb query \
    --table-name Livro \
    --index-name TitleNumberPagination-index \
    --key-condition-expression "Title = :v_title and NumberPagination = :v_number" \
    --expression-attribute-values  '{":v_title":{"S":"Memórias Póstumas de Brás Cubas"},":v_number":{"S":"500"} }'    
```