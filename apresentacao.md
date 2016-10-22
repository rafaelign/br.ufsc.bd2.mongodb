### Sharding

```js
// Configurando a fragmentação
// Diretórios
mkdir data && mkdir data/configserver && mkdir data/sh1 && mkdir data/sh2 && mkdir data/sh3

// Inicializa o Config Server (Metadados)
mongod --configsvr --dbpath data/configserver --port 27010

// Inicializa o Router (Mediador)
mongos --configdb localhost:27010 --port 27020

// Iniciando instâncias de Sharding e pré-configurando para receber replica
mongod --port 27031 --dbpath data/sh1
mongod --port 27032 --dbpath data/sh2
mongod --port 27033 --dbpath data/sh3

// Registrando no Router
mongo --port 27020;

sh.addShard("localhost:27031");
sh.addShard("localhost:27032");
sh.addShard("localhost:27033");

// Definindo collection
sh.enableSharding("apresentacao");

sh.shardCollection("apresentacao.carros", {"_id": 1});

// Insert inicial
var marcas = ["Volkswagen", "Fiat", "Honda", "Toyota"];
var potencias = ["1.0", "1.6", "1.8", "2.0"];
var cores = ["Preto", "Grafite", "Branco"];
var p;
for ( i = 1; i < 50000; i++ ) {
    p = potencias[ Math.floor(Math.random() * 4) ];
    db.carros.insert({
        key: 'carro-' + i,
        descricao : 'Carro ' + i + ' ' + p,
        valor: Math.random() * 10000 * p,
        status: 1,
        cor: cores[ Math.floor(Math.random() * 3) ],
        date_created: Date.now(),
        marca: marcas[ Math.floor(Math.random() * 4) ]
    });
}

var marcas = ["Volkswagen", "Fiat", "Honda", "Toyota"];
var potencias = ["1.0", "1.6", "1.8", "2.0"];
var cores = ["Branco", "Azul", "Prata"];
var p;
for ( i = 50000; i < 100000; i++ ) {
    p = potencias[ Math.floor(Math.random() * 4) ];
    db.carros.insert({
        key: 'carro-' + i,
        descricao : 'Carro ' + i + ' ' + p,
        valor: Math.random() * 10000 * p,
        status: 1,
        cor: cores[ Math.floor(Math.random() * 3) ],
        date_created: Date.now(),
        marca: marcas[ Math.floor(Math.random() * 4) ]
    });
}

var marcas = ["Volkswagen", "Fiat", "Honda", "Toyota"];
var potencias = ["1.0", "1.6", "1.8", "2.0"];
var cores = ["Prata", "Verde", "Preto"];
var p;
for ( i = 100000; i < 150000; i++ ) {
    p = potencias[ Math.floor(Math.random() * 4) ];
    db.carros.insert({
        key: 'carro-' + i,
        descricao : 'Carro ' + i + ' ' + p,
        valor: Math.random() * 10000 * p,
        status: 1,
        cor: cores[ Math.floor(Math.random() * 3) ],
        date_created: Date.now(),
        marca: marcas[ Math.floor(Math.random() * 4) ]
    });
}
```

### CRUD

#### insert()

```js
db.carros.count();

db.carros.insert([{
    key: 'carro-' + 150000,
    descricao : "Deville/Eldorado 4.9",
    valor: Math.random() * 100000,
    status: 1,
    cor: "Prata",
    date_created: Date.now(),
    marca: "Cadillac"
}, {
    key: 'carro-' + 150001,
    descricao : "Seville 4.6",
    valor: Math.random() * 104000,
    status: 1,
    cor: "Prata",
    date_created: Date.now(),
    marca: "Cadillac"
}]);

db.carros.count();
```

#### update()

```js
// Normal
var carro = db.carros.findOne({ marca: "Cadillac" });
    carro.descricao = carro.descricao + " Atualizado";

db.carros.update({ _id: carro._id }, carro);

db.carros.find({ _id: carro._id });

// Atualização sem o parâmetro multi - Vai gerar erro pois a base esta usando sharding (proposital)
db.carros.count({ marca: "Volkswagen" });
db.carros.update({ marca: "Volkswagen" }, {
    $set: {
        marca: "Volks"
    }
});

// Atualização com o parâmetro multi
db.carros.count({ marca: "Volkswagen" });
db.carros.update({ marca: "Volkswagen" }, {
    $set: {
        marca: "Volks"
    }
},{ multi: 1 });
db.carros.count({ marca: "Volkswagen" });

// Reajustando
db.carros.update({ marca: "Volks" }, {
    $set: {
        marca: "Volkswagen"
    }
},{ multi: 1 });
```

#### remove()

```js
db.carros.count();

var id = db.carros.findOne({ marca: "Cadillac" })._id;
db.carros.remove({ _id: id }, 1);

db.carros.count();
```

#### find()

```js
// Algumas formas de busca
db.carros.count({
    $or: [
        { marca: "Cadillac" },
        { marca: "Volkswagen"}
    ]
});

db.carros.count({
    $and: [
        { cor: "Preto" },
        { marca: "Volkswagen"}
    ]
});

db.carros.count({
    cor: {
        $in: [
            "Preto",
            "Prata"
        ]
    }
});

db.carros.count({
    valor: {
        $gt: 17000
    }
});

db.carros.count({
    valor: {
        $lt: 17000
    }
});
```

### Aggregate, Group e Distinct

```js
// Distinct
db.carros.distinct('marca');

// Group - Vai gerar erro pois a base esta usando sharding (proposital)
db.carros.group({
    initial: { total: 0 },
    reduce: function(current, result) {
        result.total++;
        if (result[current.marca]) {
            result[current.marca]++;
        } else {
             result[current.marca] = 1;
        }
    }
});

// Aggregate - mesmo resultado que seria obtido pelo Group
db.carros.aggregate({
    $group: {
        _id: '$marca',
        total: { $sum: 1 }
    }
});

```

### Índices

```js
// Busca sem índice
db.carros.find({ key: "carro-150" }).explain('executionStats').executionStats.totalDocsExamined
db.carros.find({ key: "carro-150" }).explain('executionStats').executionStats.executionTimeMillis

// Criação de índice
db.carros.createIndex({ key: 1 });

// Busca com índice
db.carros.find({ key: "carro-150" }).explain('executionStats').executionStats.totalDocsExamined
db.carros.find({ key: "carro-150" }).explain('executionStats').executionStats.executionTimeMillis
```

### Gerenciamento de Usuários
```js
// Criação de usuários
db.runCommand({usersInfo: 1});
db.createUser({
    user: "SomenteLeitura",
    pwd: "leitura",
    roles: [ { role: "readAnyDatabase", db: "admin" }, "read" ]
});

// Adicionando permissões
db.runCommand( {
    grantRolesToUser: "SomenteLeitura",
    roles: [
        "readWrite",
    ],
    writeConcern: { w: "majority" , wtimeout: 2000 }
});

// Removendo permissões
db.revokeRolesFromUser( "SomenteLeitura", [ "read" ] );

db.runCommand({usersInfo: 1});
```
