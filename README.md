<h1 align="center">🌌 Desafio MongoDB: Multiverso Nerd Edition 🦸‍♂️</h1>

<p align="center">
  <a href="https://www.mongodb.com/"><img src="https://img.shields.io/badge/Database-MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white" alt="MongoDB"></a>
  <a href="#"><img src="https://img.shields.io/badge/Status-Concluído-success?style=for-the-badge" alt="Status"></a>
</p>

Este repositório contém a resolução do desafio de higienização e normalização de dados do **Multiverso Nerd**. O objetivo foi restaurar a ordem no banco de dados multiverso, limpando registros inconsistentes e estruturando as informações em coleções relacionais.

## 🛠️ Tecnologias e Ferramentas
- **Banco de Dados:** MongoDB (NoSQL)

## 📜 Schema de Dados

A estrutura dos dados foi normalizada e higienizada, seguindo o seguinte schema para cada personagem/entidade:

| Campo | Tipo | Descrição |
|---|---|---|
| `name` | String | Nome principal do personagem. |
| `alias` | Array de Strings | Apelidos ou nomes alternativos (pode ser vazio). |
| `universe` | String | Universo de origem (ex: Marvel, Star Wars, DC Comics). |
| `first_appearance` | String ou Null | Primeira aparição registrada (ex: quadrinho, episódio). |
| `power_level` | Número, String ou Null | Nível de poder (numérico, descritivo ou nulo se não aplicável). |
| `equipment` | Array de Strings | Equipamentos, armas ou habilidades tecnológicas/mágicas. |
| `species` | String | Espécie ou raça do personagem. |
| `movies` | Array de Strings | Lista de filmes ou mídias principais onde aparece. |
| `debut_year` | Número | Ano de estreia ou criação do personagem. |
