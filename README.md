# 🎵 Music Graph Recommender (Sistema de Recomendação Baseado em Grafos)

Este é um projeto que implementa um sistema de recomendação de músicas utilizando a arquitetura de **Grafos** com **Neo4j** e a linguagem de consulta **Cypher**.

O principal objetivo é demonstrar como modelar dados contextuais (usuários, músicas, artistas, gêneros) e suas interações como um grafo, permitindo consultas complexas para gerar recomendações altamente personalizadas.

## 🚀 Arquitetura e Tecnologia

* **Banco de Dados:** Neo4j (Graph Database)
* **Linguagem de Consulta:** Cypher
* **Back-end/Conexão:** Python 3.x
* **Driver:** `neo4j` Python Driver

### Modelo de Grafo (Schema)

O sistema utiliza um modelo de grafo que representa as entidades e suas relações:

| Entidade (Node) | Relação (Relationship) |
| :--- | :--- |
| `User`, `Song`, `Artist`, `Genre` | `LISTENS_TO`, `LIKES`, `FOLLOWS`, `PERFORMED_BY`, `HAS_GENRE` |



## 🛠️ Configuração e Instalação

### Pré-requisitos

1.  **Neo4j Instance:** Você precisa ter uma instância do Neo4j rodando (localmente ou na nuvem). As configurações padrão assumem `bolt://localhost:7687` com as credenciais `neo4j` / `your_neo4j_password`.
2.  **Python 3.x**

### 1. Instalação das Dependências Python

```bash
pip install -r requirements.txt
