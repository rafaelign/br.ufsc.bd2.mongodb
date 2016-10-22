### CRUD

```js
// Tab 1
mongod --dbpath data/db

// Tab 2
mongo

use apresentacao

mongoimport --db apresentacao --collection carros --drop --file carros.json --jsonArray
```

#### insert()

```js
db.carros.find({}).count();
db.carros.insert({
    fipe_marca: "VW - VolksWagen",
    name: "Novo Carro Volks",
    marca: "VOLKSWAGEN",
    key: "novo-carro-5585",
    id: "9999",
    fipe_name: "Novo Carro Volks",
    parecido_com: [
        "Gol", "Golf", "Fox", "Polo"
    ]
});
db.carros.find({}).count();
```

#### update()

```js
db.candidatos.find({}).count();
db.candidatos.insert([
    {
        nome: "João",
        numero: "55",
        idade: 40,
    },
    {
        nome: "Maria",
        numero: "99",
        idade: 55,
    },
    {
        nome: "José",
        numero: "22",
        idade: 30,
    }
]);
db.candidatos.find({}).count();
```

#### remove()

```js
```

#### find()

```js
// Importação de uma coleção

mongoimport --db be-mean-instagram --collection carros --drop --file carros.json

// Algumas formas de busca
```

### Índices

```js
// Busca sem índice
// Criação de índice
// Busca com índice
```

### Gerenciamento de Usuários
```js
// Criação de usuários
// Adicionando permissões
// Removendo permissões
```

### Sharding

```js
// Configurando uma fragmentação
```

### Réplicas

```js
//Configurando uma replicação de dados
```
