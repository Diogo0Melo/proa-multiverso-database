<h1 align="center">🌌 Desafio MongoDB: Multiverso Nerd Edition 🦸‍♂️</h1>

<p align="center">
  <a href="https://www.mongodb.com/"><img src="https://img.shields.io/badge/Database-MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white" alt="MongoDB"></a>
  <a href="#"><img src="https://img.shields.io/badge/Status-Concluído-success?style=for-the-badge" alt="Status"></a>
</p>

Este repositório contém a resolução do desafio de higienização e normalização de dados do **Multiverso Nerd**. O objetivo foi restaurar a ordem no banco de dados interdimensional, limpando registros inconsistentes e estruturando as informações em coleções relacionais.

## 🛠️ Tecnologias e Ferramentas
- **Banco de Dados:** MongoDB (NoSQL)
- **Interface:** MongoDB Compass / Mongosh
- **Pré-processamento:** Python (para higienização inicial e geração do JSON limpo)
- **Dataset:** 75 registros de personagens, criaturas e heróis do mundo nerd

## 🗺️ Guia de Soluções

<details>
<summary><h3 style="display: inline-block">🧹 Nível 1: Higienização de Dados (Etapa 1)</h3></summary>
<br>

A higienização foi realizada combinando um script Python (`clean_and_normalize.py`) para padronização robusta de textos e tipos, gerando o arquivo `nerd_universe_clean.json`. 

Caso deseje realizar a higienização diretamente no MongoDB, aqui estão as pipelines de agregação equivalentes:

<details>
<summary><strong>🔧 1.1 Padronizar nomes de campos e capitalização</strong></summary>
<br>

💻 **Query:**
```javascript
db.multiverso_raw.aggregate([
  {
    $project: {
      name: {
        $trim: {
          input: {
            $toTitleCase: {
              $replaceAll: { input: { $ifNull: ["$Name", "$nome", "$char_name", "$name", "$Nome"] }, find: "  ", replacement: " " }
            }
          }
        }
      },
      universe: { $toLower: { $trim: { input: { $ifNull: ["$Universe", "$universe"] } } } },
      powerLevel: { $toInt: { $ifNull: ["$powerLevel", "$PowerLevel", "$power_level", "$powerlevel", 0] } },
      equipment: {
        $map: {
          input: { $split: [{ $ifNull: ["$equipment", ""] }, ", "] },
          as: "eq",
          in: { $trim: { input: { $toTitleCase: "$$eq" } } }
        }
      }
    }
  }
])
```
</details>
<br>

<details>
<summary><strong>🔧 1.2 Unificar universos semelhantes e tratar nulos</strong></summary>
<br>

💻 **Query:**
```javascript
db.multiverso_raw.aggregate([
  {
    $addFields: {
      universe_clean: {
        $switch: {
          branches: [
            { case: { $in: ["$universe", ["marvel", "marvel studios", "marvel comics"]] }, then: "Marvel" },
            { case: { $in: ["$universe", ["starwars", "star wars"]] }, then: "Star Wars" },
            { case: { $in: ["$universe", ["dc", "dc comics"]] }, then: "DC Comics" }
          ],
          default: { $toTitleCase: { $trim: { input: "$universe" } } }
        }
      }
    }
  },
  { $match: { name: { $ne: null, $ne: "" } } }
])
```
</details>
<br>

<details>
<summary><strong>🔧 1.3 Deduplicação e Limpeza de Arrays</strong></summary>
<br>

💻 **Query:**
```javascript
db.multiverso_raw.aggregate([
  {
    $addFields: {
      alias: {
        $filter: {
          input: {
            $cond: [
              { $isArray: "$alias" },
              "$alias",
              { $split: [{ $ifNull: ["$alias", ""] }, ","] }
            ]
          },
          as: "item",
          cond: { $and: [
            { $ne: [{ $trim: { input: "$$item" } }, ""] },
            { $ne: [{ $toLower: { $trim: { input: "$$item" } } }, "null"] }
          ]}
        }
      }
    }
  },
  {
    $group: {
      _id: { 
        $toLower: { 
          $replaceAll: { 
            input: { $trim: { input: { $ifNull: ["$name", ""] } } }, 
            find: " ", 
            replacement: "" 
          } 
        } 
      },
      doc: { $first: "$$ROOT" }
    }
  },
  { $replaceRoot: { newRoot: "$doc" } }
])
```
</details>
<br>
</details>

<details>
<summary><h3 style="display: inline-block">🗄️ Nível 2: Normalização (Etapa 2)</h3></summary>
<br>

Os dados higienizados foram normalizados em coleções separadas para garantir a integridade referencial, utilizando IDs de referência (ObjectId) em vez de strings.

<details>
<summary><strong>🏗️ 2.1 Criar coleções de referência (Universos, Espécies, Equipamentos, Filmes)</strong></summary>
<br>

💻 **Query:**
```javascript
// 1. Universos
db.nerd_universe_clean.aggregate([
  { $group: { _id: "$universe" } },
  { $project: { _id: 0, name: "$_id" } },
  { $out: "universes" }
]);

// 2. Espécies
db.nerd_universe_clean.aggregate([
  { $group: { _id: "$species" } },
  { $project: { _id: 0, name: "$_id" } },
  { $out: "species" }
]);

// 3. Equipamentos (desnormalizando o array)
db.nerd_universe_clean.aggregate([
  { $unwind: "$equipment" },
  { $group: { _id: "$equipment" } },
  { $project: { _id: 0, name: "$_id" } },
  { $out: "equipment" }
]);

// 4. Filmes/Obras
db.nerd_universe_clean.aggregate([
  { $unwind: "$movies" },
  { $group: { _id: "$movies" } },
  { $project: { _id: 0, title: "$_id" } },
  { $out: "movies" }
]);
```
</details>
<br>

<details>
<summary><strong>🔗 2.2 Atualizar coleção `characters` com IDs de referência</strong></summary>
<br>

💻 **Script (Mongosh):**
```javascript
// Substituir strings por IDs de referência
db.universes.find().forEach(u => {
  db.characters.updateMany(
    { universe: u.name }, 
    { $set: { universe_id: u._id }, $unset: { universe: "" } }
  );
});

db.species.find().forEach(s => {
  db.characters.updateMany(
    { species: s.name }, 
    { $set: { species_id: s._id }, $unset: { species: "" } }
  );
});

db.characters.find().forEach(char => {
  let equipIds = char.equipment.map(eq => {
    let found = db.equipment.findOne({ name: eq });
    return found ? found._id : null;
  }).filter(id => id !== null);

  let movieIds = char.movies.map(mv => {
    let found = db.movies.findOne({ title: mv });
    return found ? found._id : null;
  }).filter(id => id !== null);
  
  db.characters.updateOne(
    { _id: char._id },
    { 
      $set: { equipment_ids: equipIds, movie_ids: movieIds }, 
      $unset: { equipment: "", movies: "" } 
    }
  );
});
```
</details>
<br>

<details>
<summary><strong>🔗 2.3 Relacionar Personagens com Universos usando $lookup por ID</strong></summary>
<br>

💻 **Query:**
```javascript
db.characters.aggregate([
  {
    $lookup: {
      from: "universes",
      localField: "universe_id",
      foreignField: "_id",
      as: "universe_details"
    }
  },
  { $unwind: "$universe_details" },
  { 
    $project: { 
      name: 1, 
      universe: "$universe_details.name", 
      power_level: 1,
      universe_id: 1
    } 
  }
])
```
</details>
<br>
</details>

<details>
<summary><h3 style="display: inline-block">🚀 Nível 3: Desafio Extra (Consultas Heroicas)</h3></summary>
<br>

<details>
<summary><strong>🦸 3.1 O personagem mais poderoso de cada universo</strong></summary>
<br>

💻 **Query:**
```javascript
db.nerd_universe_clean.aggregate([
  { $match: { power_level: { $type: "number" } } },
  { $sort: { universe: 1, power_level: -1 } },
  {
    $group: {
      _id: "$universe",
      most_powerful: { $first: "$name" },
      max_power: { $first: "$power_level" }
    }
  },
  { $project: { _id: 0, universe: "$_id", character: "$most_powerful", power_level: "$max_power" } }
])
```
</details>
<br>

<details>
<summary><strong>🌍 3.2 O universo com mais personagens registrados</strong></summary>
<br>

💻 **Query:**
```javascript
db.nerd_universe_clean.aggregate([
  { $group: { _id: "$universe", total_characters: { $sum: 1 } } },
  { $sort: { total_characters: -1 } },
  { $limit: 1 },
  { $project: { _id: 0, universe: "$_id", total: "$total_characters" } }
])
```
</details>
<br>

<details>
<summary><strong>🛡️ 3.3 Quantos personagens possuem mais de 3 equipamentos</strong></summary>
<br>

💻 **Query:**
```javascript
db.nerd_universe_clean.aggregate([
  { $match: { $expr: { $gt: [{ $size: { $ifNull: ["$equipment", []] } }, 3] } } },
  { $count: "personagens_com_mais_de_3_equipamentos" }
])
```
</details>
<br>
</details>