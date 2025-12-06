# 🎵 Music Graph Recommender (Sistema de Recomendação Baseado em Grafos)

Este projeto demonstra um sistema de recomendação de músicas utilizando a arquitetura de **Grafos** com **Neo4j** e a linguagem de consulta **Cypher**.

O objetivo é modelar a complexidade das relações entre **Usuários**, **Músicas**, **Artistas** e **Gêneros** em um grafo, permitindo consultas avançadas para gerar recomendações altamente personalizadas. A solução é entregue em um único arquivo Python que gerencia a inicialização, a população e a execução das consultas.

## 🚀 Arquitetura e Configuração

| Componente | Tecnologia | Detalhe |
| :--- | :--- | :--- |
| **Banco de Dados** | Neo4j | Modelo de grafo nativo para alto desempenho em consultas relacionais. |
| **Linguagem de Consulta** | Cypher | Linguagem declarativa e expressiva para travessia de grafos. |
| **Back-end/Script** | Python 3.x | Usado para conectar, inicializar e executar as consultas via `neo4j` driver. |

---

## 🏗️ Modelo de Grafo (Schema)

A estrutura do grafo é projetada para capturar as interações e metadados musicais de forma eficiente.

### Entidades (Nós)

| Nó (Label) | Propriedades Chave |
| :--- | :--- |
| `User` | `userId`, `name` |
| `Song` | `songId`, `title` |
| `Artist` | `artistId`, `name` |
| `Genre` | `name` |

### Relações

| Relação (Type) | De | Para | Propriedades Chave | Descrição |
| :--- | :--- | :--- | :--- | :--- |
| `LISTENS_TO` | `User` | `Song` | `playCount` | Interação de escuta, crucial para peso de recomendação. |
| `LIKES` | `User` | `Song` | `timestamp` | Preferência forte. |
| `FOLLOWS` | `User` | `Artist` | `timestamp` | Interesse explícito no artista. |

O diagrama conceitual reflete essa estrutura: 

---

## 🛠️ Configuração e Execução

### Pré-requisitos

1.  **Instância Neo4j:** Uma instância do Neo4j deve estar rodando (URI padrão: `bolt://localhost:7687`).
2.  **Python 3.x**

### Passos de Execução

1.  **Instalação da Dependência:**
    ```bash
    pip install neo4j
    ```

2.  **Criação do Arquivo:** Crie um arquivo chamado **`music_recommender.py`** na sua máquina e cole o código Python da seção seguinte.

3.  **AJUSTE CRÍTICO:** **Edite** o arquivo `music_recommender.py` e **substitua** a variável `PASSWORD` com sua senha real do Neo4j.

4.  **Execução:** Execute o script no terminal. Ele irá inicializar o DB, popular os dados e imprimir os resultados das 3 estratégias de recomendação:
    ```bash
    python music_recommender.py
    ```

---

## 🐍 Código Python: `music_recommender.py`

Este script contém toda a lógica necessária para o projeto. Salve-o na raiz do seu diretório.

```python
from neo4j import GraphDatabase

# --- CONFIGURAÇÕES DE CONEXÃO (MUDAR SUA SENHA AQUI) ---
URI = "bolt://localhost:7687"
USERNAME = "neo4j"
PASSWORD = "SUA_SENHA_DO_NEO4J_AQUI" # <-- ATUALIZE ESTA LINHA COM SUA SENHA REAL

class MusicRecommender:
    """Gerencia a conexão, inicialização do grafo e execução de consultas Cypher."""
    
    def __init__(self, uri, user, password):
        self.driver = GraphDatabase.driver(uri, auth=(user, password))

    def close(self):
        print("\nFechando conexão com o Neo4j...")
        self.driver.close()

    def _execute_write(self, query, parameters=None):
        """Método auxiliar para executar operações de escrita e garantir integridade."""
        with self.driver.session() as session:
            session.execute_write(lambda tx: tx.run(query, parameters))

    def _execute_read(self, query, parameters=None):
        """Método auxiliar para executar operações de leitura performáticas."""
        with self.driver.session() as session:
            return session.execute_read(lambda tx: list(tx.run(query, parameters)))

    # --- INICIALIZAÇÃO E POPULAÇÃO ---

    def create_constraints_and_indexes(self):
        """Cria índices e restrições para garantir a unicidade e performance (Melhor Prática)."""
        print("Criando índices e restrições...")
        self._execute_write("CREATE CONSTRAINT IF NOT EXISTS FOR (u:User) REQUIRE u.userId IS UNIQUE")
        self._execute_write("CREATE CONSTRAINT IF NOT EXISTS FOR (s:Song) REQUIRE s.songId IS UNIQUE")
        self._execute_write("CREATE CONSTRAINT IF NOT EXISTS FOR (a:Artist) REQUIRE a.artistId IS UNIQUE")
        self._execute_write("CREATE CONSTRAINT IF NOT EXISTS FOR (g:Genre) REQUIRE g.name IS UNIQUE")
        print("Índices e restrições criadas.")

    def populate_graph(self):
        """Popula o grafo com dados de exemplo."""
        print("Populando o grafo...")
        
        # 1. Criação de Artistas, Músicas e Gêneros
        artist_song_data = [
            ("The Beatles", "Hey Jude", "Pop"),
            ("Iron Maiden", "The Trooper", "Metal"),
            ("Miles Davis", "So What", "Jazz"),
            ("Queen", "Bohemian Rhapsody", "Rock"),
            ("Iron Maiden", "Aces High", "Metal"),
            ("Queen", "We Will Rock You", "Rock"),
        ]
        
        query_structure = """
        UNWIND $data AS item
        MERGE (a:Artist {artistId: item.artist, name: item.artist})
        MERGE (s:Song {songId: item.song, title: item.song})
        MERGE (g:Genre {name: item.genre})
        MERGE (s)-[:PERFORMED_BY]->(a)
        MERGE (s)-[:HAS_GENRE]->(g)
        """
        data_list = [{"artist": a, "song": s, "genre": g} for a, s, g in artist_song_data]
        self._execute_write(query_structure, {"data": data_list})

        # 2. Criação de Usuários e Interações (LISTENS_TO, LIKES, FOLLOWS)
        user_interactions = [
            ("u101", "Alice", "Hey Jude", "LISTENS_TO", 3),
            ("u101", "Alice", "Bohemian Rhapsody", "LISTENS_TO", 2),
            ("u101", "Alice", "The Trooper", "LIKES", 0), 
            ("u102", "Bob", "The Trooper", "LISTENS_TO", 5),
            ("u102", "Bob", "Aces High", "LISTENS_TO", 4),
            ("u102", "Bob", "So What", "LISTENS_TO", 1),
            ("u103", "Charlie", "We Will Rock You", "LISTENS_TO", 8),
            ("u103", "Charlie", "Bohemian Rhapsody", "LISTENS_TO", 7),
        ]
        
        # Relações FOLLOWS
        self._execute_write("MERGE (u:User {userId: 'u101', name: 'Alice'})-[:FOLLOWS {timestamp: datetime()}]->(a:Artist {artistId: 'Iron Maiden'})")
        self._execute_write("MERGE (u:User {userId: 'u102', name: 'Bob'})-[:FOLLOWS {timestamp: datetime()}]->(a:Artist {artistId: 'Miles Davis'})")
        self._execute_write("MERGE (u:User {userId: 'u103', name: 'Charlie'})-[:FOLLOWS {timestamp: datetime()}]->(a:Artist {artistId: 'Queen'})")


        interaction_query = """
        MERGE (u:User {userId: $userId, name: $userName})
        MERGE (s:Song {songId: $songTitle})
        WITH u, s

        FOREACH (i IN CASE WHEN $type = 'LISTENS_TO' THEN [1] ELSE [] END |
            MERGE (u)-[r:LISTENS_TO]->(s)
            ON CREATE SET r.playCount = $count, r.timestamp = datetime()
            ON MATCH SET r.playCount = r.playCount + $count, r.timestamp = datetime()
        )

        FOREACH (i IN CASE WHEN $type = 'LIKES' THEN [1] ELSE [] END |
            MERGE (u)-[r:LIKES]->(s)
            ON CREATE SET r.timestamp = datetime()
        )
        """
        
        for userId, userName, songTitle, relType, count in user_interactions:
            self._execute_write(
                interaction_query, 
                {"userId": userId, "userName": userName, "songTitle": songTitle, "type": relType, "count": count}
            )
        print("População concluída.")
        
    def initialize_and_populate_db(self):
        """Limpa o DB (detaching e deletando todos os nós) e o popula."""
        print("--- Inicialização do Banco de Dados ---")
        self._execute_write("MATCH (n) DETACH DELETE n") # Limpa o DB (Cuidado em Produção!)
        self.create_constraints_and_indexes()
        self.populate_graph()
        print("--- Inicialização e População Concluídas ---")

    # --- CONSULTAS DE RECOMENDAÇÃO ---

    def recommend_item_to_item(self, user_id: str, limit: int = 5):
        """Recomendação 1: Item-to-Item (Gênero Mais Escutado)."""
        print(f"\n--- Recomendação 1: Gênero Mais Escutado para o Usuário {user_id} ---")
        query = """
        MATCH (u:User {userId: $userId})-[:LISTENS_TO]->(s1:Song)-[:HAS_GENRE]->(g:Genre)
        WITH u, g, sum(s1.playCount) AS genreScore
        ORDER BY genreScore DESC
        LIMIT 1
        
        MATCH (g)<-[:HAS_GENRE]-(s2:Song)
        WHERE NOT EXISTS { (u)-[:LISTENS_TO|LIKES]->(s2) }
        
        OPTIONAL MATCH (s2)<-[r:LISTENS_TO]-(:User)
        WITH s2, g.name AS genreName, count(r) AS popularity
        ORDER BY popularity DESC
        LIMIT $limit
        RETURN s2.title AS RecommendedSong, genreName, popularity
        """
        return self._execute_read(query, {"userId": user_id, "limit": limit})

    def recommend_user_to_user(self, user_id: str, limit: int = 5):
        """Recomendação 2: Colaborativa (Usuários Vizinhos por Gosto Comum)."""
        print(f"\n--- Recomendação 2: Usuários Vizinhos (Gostos Comuns) para {user_id} ---")
        query = """
        MATCH (u1:User {userId: $userId})-[r1:FOLLOWS|LISTENS_TO]->(target)
        MATCH (target)<-[r2:FOLLOWS|LISTENS_TO]-(u2:User)
        WHERE u1 <> u2 
        
        MATCH (u2)-[r3:LISTENS_TO]->(s:Song)
        WHERE NOT EXISTS { (u1)-[:LISTENS_TO|LIKES]->(s) }
        
        WITH s, sum(r3.playCount) AS recommendationScore
        ORDER BY recommendationScore DESC
        LIMIT $limit
        
        MATCH (s)-[:PERFORMED_BY]->(a:Artist)
        RETURN s.title AS RecommendedSong, collect(a.name) AS Artists, recommendationScore
        """
        return self._execute_read(query, {"userId": user_id, "limit": limit})

    def recommend_followed_artist(self, user_id: str, limit: int = 3):
        """Recomendação 3: Direta (Músicas de Artistas Seguidos)."""
        print(f"\n--- Recomendação 3: Artistas Seguidos para {user_id} ---")
        query = """
        MATCH (u:User {userId: $userId})-[:FOLLOWS]->(a:Artist)<-[:PERFORMED_BY]-(s:Song)
        WHERE NOT EXISTS {
            (u)-[:LISTENS_TO|LIKES]->(s)
        }
        
        OPTIONAL MATCH (s)<-[r:LISTENS_TO]-(:User)
        WITH s, a.name AS ArtistName, count(r) AS popularity
        ORDER BY popularity DESC
        LIMIT $limit
        RETURN s.title AS RecommendedSong, ArtistName, popularity
        """
        return self._execute_read(query, {"userId": user_id, "limit": limit})

    @staticmethod
    def print_results(results):
        """Imprime os resultados da consulta em um formato legível (Tabela CLI)."""
        if not results:
            print("Nenhuma recomendação encontrada.")
            return

        header = list(results[0].keys())
        print(" | ".join(header))
        print("-" * (sum(len(str(h)) for h in header) + 3 * len(header)))
        for record in results:
            values = [str(record[key]) for key in header]
            print(" | ".join(values))


if __name__ == "__main__":
    recommender = MusicRecommender(URI, USERNAME, PASSWORD)
    
    try:
        recommender.initialize_and_populate_db()
        
        # Execução das recomendações para usuários de exemplo
        alice_id = 'u101'
        print(f"\n======== EXECUTANDO RECOMENDAÇÕES PARA USUÁRIO: {alice_id} (Alice) ========")
        
        MusicRecommender.print_results(recommender.recommend_item_to_item(alice_id))
        MusicRecommender.print_results(recommender.recommend_user_to_user(alice_id))
        MusicRecommender.print_results(recommender.recommend_followed_artist(alice_id))

        bob_id = 'u102'
        print(f"\n======== EXECUTANDO RECOMENDAÇÕES PARA USUÁRIO: {bob_id} (Bob) ========")
        
        MusicRecommender.print_results(recommender.recommend_item_to_item(bob_id))
        MusicRecommender.print_results(recommender.recommend_user_to_user(bob_id))
        MusicRecommender.print_results(recommender.recommend_followed_artist(bob_id))
        
    except Exception as e:
        print(f"\n[ERRO FATAL] Falha na execução. Verifique se o Neo4j está ativo na porta 7687 e as credenciais.")
        print(f"Detalhe do Erro: {e}")
        
    finally:
        recommender.close()
