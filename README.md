# Player Service Architecture Diagram

```mermaid
flowchart LR
    %% Clients
    subgraph "Clients"
        Clients[(API Consumers / Postman)]:::external
    end

    %% API Test Collection
    subgraph "API Test Collection"
        GetAll["GetAllPlayers.http"]:::external
        GetBy["GetPlayerById.http"]:::external
        ChatReq["chat_requests.txt"]:::external
    end

    %% Java Application
    subgraph "Player Service (Spring Boot Java App)"
        direction TB
        AppEntry["PlayerServiceJavaApplication"]:::java
        subgraph "Configuration Layer"
            Config["ChatClientConfiguration"]:::java
        end
        subgraph "Controller Layer"
            PC["PlayerController"]:::java
            CC["ChatController"]:::java
        end
        subgraph "Service Layer"
            PS["PlayerService"]:::java
            CCS["ChatClientService"]:::java
        end
        subgraph "Repository Layer"
            PR["PlayerRepository"]:::java
        end
        subgraph "Model Layer"
            PlayerModel["Player.java"]:::java
            PlayersModel["Players.java"]:::java
        end

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
        Schema["schema.sql"]:::db
        Yml["application.yml"]:::java
    end

    %% Runtime Components
    H2["H2 Database\n(in-memory)"]:::db
    Ollama["Ollama LLM Container\ntinyllama\n(port 11434)"]:::external

    %% Python Model Trainer (Build-time)
    subgraph "Python Model Trainer"
        direction TB
        PM["player-service-model/"]:::build
        ModelCode["model.py"]:::build
        Notebook["train.ipynb"]:::build
        JobLib["team_model.joblib"]:::build
        DockerTrainer["Dockerfile"]:::build
        PM --> ModelCode
        PM --> Notebook
        ModelCode --> JobLib
        PM --> DockerTrainer
    end

    %% Connections
    Clients --> GetAll
    Clients --> GetBy
    Clients --> ChatReq
    Clients -- "GET /v1/players":::api --> PC
    Clients -- "POST /v1/chat/generate":::api --> CC

    PR -- "JDBC (Spring Data JPA)":::dbEdge --> H2
    Schema -- "init schema":::resource --> H2
    Yml -- "app config":::resource --> AppEntry

    CCS -- "HTTP POST /api/generate":::llm --> Ollama
    JobLib -. "Volume mount":::buildEdge .-> Ollama

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
