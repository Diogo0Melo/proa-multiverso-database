<h1 align="center">🌌 Desafio MongoDB: Limpeza do Multiverso 🦸‍♂️</h1>

<p align="center">
  <a href="https://www.mongodb.com/"><img src="https://img.shields.io/badge/Database-MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white" alt="MongoDB"></a>
  <a href="#"><img src="https://img.shields.io/badge/Status-Concluído-success?style=for-the-badge" alt="Status"></a>
</p>

Este repositório contém a resolução do desafio de higienização e normalização de dados do **Multiverso Nerd**. O objetivo foi restaurar a ordem no banco de dados multiverso, limpando registros inconsistentes e estruturando as informações em coleções relacionais.

## 🛠️ Tecnologias e Ferramentas
- **Banco de Dados:** MongoDB (NoSQL)
- **Linguagem:** Python (para limpeza e normalização)

## 📜 Schema de Dados Normalizado

A estrutura dos dados foi normalizada em coleções relacionais, utilizando IDs de referência:

### `characters`
| Campo | Tipo | Descrição |
|---|---|---|
| `name` | String | Nome principal do personagem. |
| `alias` | Array de Strings | Apelidos ou nomes alternativos. |
| `universe_id` | String (Ref) | Referência ao universo de origem. |
| `species_id` | String (Ref) | Referência à espécie do personagem. |
| `first_appearance` | String ou Null | Primeira aparição registrada. |
| `power_level` | Número, String ou Null | Nível de poder. |
| `equipment_ids` | Array de Strings (Ref) | Lista de IDs de equipamentos. |
| `movie_ids` | Array de Strings (Ref) | Lista de IDs de filmes/mídias. |
| `debut_year` | Número | Ano de estreia ou criação. |

### `universes`
| Campo | Tipo | Descrição |
|---|---|---|
| `_id` | String | Identificador único. |
| `name` | String | Nome do universo (ex: Marvel, Star Wars). |
| `type` | String | Tipo de mídia (ex: mixed, HQs, games). |
| `origin` | String | Origem do universo. |

### `species`
| Campo | Tipo | Descrição |
|---|---|---|
| `_id` | String | Identificador único. |
| `name` | String | Nome da espécie (ex: Human, Saiyan). |
| `description` | String | Descrição da espécie. |

### `equipment`
| Campo | Tipo | Descrição |
|---|---|---|
| `_id` | String | Identificador único. |
| `name` | String | Nome do equipamento. |
| `description` | String | Descrição do equipamento. |

### `movies`
| Campo | Tipo | Descrição |
|---|---|---|
| `_id` | String | Identificador único. |
| `title` | String | Título da obra. |
| `year` | Número | Ano de lançamento. |

## 🚀 Consultas Heroicas (MongoDB Aggregation)

### 1. O personagem mais poderoso de cada universo
```javascript
db.characters.aggregate([
  { $match: { power_level: { $type: "number" } } },
  { $sort: { power_level: -1 } },
  { $group: {
      _id: "$universe_id",
      max_power: { $first: "$power_level" },
      character_name: { $first: "$name" }
  }},
  { $lookup: {
      from: "universes",
      localField: "_id",
      foreignField: "_id",
      as: "universe_info"
  }},
  { $project: {
      _id: 0,
      universe: { $arrayElemAt: ["$universe_info.name", 0] },
      character: "$character_name",
      power_level: "$max_power"
  }}
])
```

### 2. O universo com mais personagens registrados
```javascript
db.characters.aggregate([
  { $group: { _id: "$universe_id", count: { $sum: 1 } } },
  { $sort: { count: -1 } },
  { $limit: 1 },
  { $lookup: {
      from: "universes",
      localField: "_id",
      foreignField: "_id",
      as: "universe_info"
  }},
  { $project: {
      _id: 0,
      universe: { $arrayElemAt: ["$universe_info.name", 0] },
      total_characters: "$count"
  }}
])
```

### 3. Quantos personagens possuem mais de 3 equipamentos
```javascript
db.characters.aggregate([
  { $match: { equipment_ids: { $exists: true, $type: "array" } } },
  { $match: { $expr: { $gt: [{ $size: "$equipment_ids" }, 3] } } },
  { $count: "personagens_com_mais_de_3_equipamentos" }
])
```