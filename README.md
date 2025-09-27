# Player Service Architecture Diagram (Pro-Level)

```mermaid
flowchart LR
    %% Clients
    subgraph "Clients"
        APIClients[(API Consumers / Postman<br>- Sends REST API requests<br>- Testing & Consumption)]:::external
    end

    %% API Test Collection
    subgraph "API Test Collection"
        GetAll["GetAllPlayers.http<br>- Example GET request"]:::external
        GetBy["GetPlayerById.http<br>- Example GET by ID"]:::external
        ChatReq["chat_requests.txt<br>- Example POST chat requests"]:::external
    end

    %% Java Application
    subgraph "Player Service (Spring Boot Java App)"
        direction TB
        AppEntry["PlayerServiceJavaApplication<br>- Spring Boot main class<br>- Starts app"]:::java

        subgraph "Configuration Layer"
            Config["ChatClientConfiguration<br>- Configures HTTP client for Ollama LLM"]:::java
        end

        subgraph "Controller Layer"
            PC["PlayerController<br>- Handles /v1/players endpoints<br>- Maps requests to PlayerService"]:::java
            CC["ChatController<br>- Handles /v1/chat/generate endpoint<br>- Maps requests to ChatClientService"]:::java
        end

        subgraph "Service Layer"
            PS["PlayerService<br>- Implements business logic<br>- CRUD operations for Player<br>- Calls PlayerRepository"]:::java
            CCS["ChatClientService<br>- Sends requests to Ollama LLM<br>- Handles responses"]:::java
        end

        subgraph "Repository Layer"
            PR["PlayerRepository<br>- Spring Data JPA<br>- CRUD on Player table<br>- Connects to H2 DB"]:::java
        end

        subgraph "Model Layer"
            PlayerModel["Player.java<br>- Entity representing a Player<br>- Maps DB columns"]:::java
            PlayersModel["Players.java<br>- Wrapper / DTO for multiple players"]:::java
        end

        %% Layer connections
        AppEntry --> PC
        AppEntry --> CC
        PC --> PS
        PS --> PR
        PS --> PlayerModel
        PS --> PlayersModel
        CC --> CCS
        Config --> CCS
    end

    %% Resources
    subgraph "Resources"
        Schema["schema.sql<br>- Initializes H2 database"]:::db
        Yml["application.yml<br>- Spring Boot config (DB, server, properties)"]:::java
    end

    %% Runtime Components
    H2["H2 Database<br>- In-memory database<br>- Stores Player data"]:::db
    Ollama["Ollama LLM Container<br>- Handles AI chat generation<br>- Port 11434"]:::external

    %% Python Model Trainer (Build-time)
    subgraph "Python Model Trainer"
        direction TB
        PM["player-service-model/<br>- Model code & training scripts"]:::build
        ModelCode["model.py<br>- AI model definition"]:::build
        Notebook["train.ipynb<br>- Jupyter Notebook to train model"]:::build
        JobLib["team_model.joblib<br>- Trained model artifact"]:::build
        DockerTrainer["Dockerfile<br>- Build container for training"]:::build

        PM --> ModelCode
        PM --> Notebook
        ModelCode --> JobLib
        PM --> DockerTrainer
    end

    %% Connections
    APIClients --> GetAll
    APIClients --> GetBy
    APIClients --> ChatReq
    APIClients -- "GET /v1/players":::api --> PC
    APIClients -- "POST /v1/chat/generate":::api --> CC

    PR -- "JDBC (Spring Data JPA)":::dbEdge --> H2
    Schema -- "Init DB schema":::resource --> H2
    Yml -- "App configuration":::resource --> AppEntry

    CCS -- "HTTP POST /api/generate":::llm --> Ollama
    JobLib -. "Volume mount (build-time)":::buildEdge .-> Ollama

    %% Styles
    classDef java fill:#AED6F1,stroke:#1F618D,color:#1F618D
    classDef db fill:#ABEBC6,stroke:#196F3D,color:#196F3D
    classDef external fill:#F9E79F,stroke:#B9770E,color:#B9770E
    classDef build fill:#D5D8DC,stroke:#566573,color:#566573

    %% Edge styles
    classDef api stroke:#1F618D,stroke-width:2px,color:#1F618D
    classDef dbEdge stroke:#196F3D,stroke-width:2px,color:#196F3D
    classDef llm stroke:#B9770E,stroke-width:2px,color:#B9770E
    classDef resource stroke:#7B7D7D,stroke-dasharray: 3 3,stroke-width:2px,color:#7B7D7D
    classDef buildEdge stroke:#566573,stroke-dasharray: 5 5,stroke-width:2px,color:#566573
