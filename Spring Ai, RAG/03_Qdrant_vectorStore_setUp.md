The best alternative is **Qdrant**. It runs in a single Docker command with zero password or user authentication hassles, has native Spring AI support, and includes a **built-in Web Dashboard** at `http://localhost:6333/dashboard` so you can visually inspect your vector collections without needing DBeaver.

1. **Run Qdrant Container:** Docker CMD.
Run Qdrant with a single line in your command prompt:

```cmd
docker run -d --name qdrant-db -p 6333:6333 -p 6334:6334 qdrant/qdrant

```

*Verification:* Open `http://localhost:6333/dashboard` in your browser. You will see the native Qdrant Web Dashboard interface.


2. **Add Qdrant Starter to Maven:** pom.xml.
Add the Qdrant vector store starter to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-qdrant-store-spring-boot-starter</artifactId>
</dependency>

```

*Verification:* Run `./mvnw dependency:resolve` or refresh Maven in your IDE.


3. **Configure Spring Properties:** application.properties.
Add the Qdrant connection parameters to `src/main/resources/application.properties`:

```properties
spring.ai.vectorstore.qdrant.host=localhost
spring.ai.vectorstore.qdrant.port=6334
spring.ai.vectorstore.qdrant.collection-name=vector_store
spring.ai.vectorstore.qdrant.initialize-schema=true

```

*Verification:
* Before Starting your application;
* Start Qdrant container 
* Spring AI will auto-connect to Qdrant on port 6334 and create the collection automatically.


### Viewing Data in Qdrant

Open `http://localhost:6333/dashboard` in your browser and click on **Collections** $\rightarrow$ **vector_store**. You will see all your document points, metadata payloads, and high-dimensional vector embeddings rendered in an interactive UI.



* The vector_store collection will not appear in the Qdrant Dashboard until your application actually inserts its first document vector. Qdrant lazily initializes the collection when the embedding model generates the first payload.

* create vector-store externally
```
 curl -X PUT "http://localhost:6333/collections/vector_store" -H "Content-Type: application.json" -d "{\"vectors\": {\"size\": 768, \"distance\": \"Cosine\"}}"
```
