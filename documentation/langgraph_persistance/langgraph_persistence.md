# LangGraph Persistence with Docker

## LangGraph & Docker

### Development Mode (No Docker)

- Running with `langgraph dev`
- Imports `langgraph-cli[inmem]` framework
- Stores state of the agent in local cache

### Deployment-ready Mode (w/ Docker)

- Persistence of the agent state on actual database
- Uses `langgraph-cli` framework (not `[inmem]`)
- Builds 3 core images:
  - Langgraph Server
  - Postgres
  - Redis

`DATABASE_URI` and `REDIS_URI` are official variables declared inside the compose file out of the box — keep environment-specific values out of the compose file and manage them via `.env` or a secrets manager.

#### What is Postgres actually doing?

Enables Agent Data Persistence — [Persistence docs by LangChain](https://langchain-ai.github.io/langgraph/concepts/persistence/)

- Remembers conversations so they don't disappear when the server restarts
- Essential for any production deployment where you want conversations to survive restarts or need horizontal scaling

Database Schema of LangGraph framework in UML:

<img width="3840" height="1790" alt="UML_Langgraph" src="https://github.com/user-attachments/assets/e87f83ea-1319-444d-8dc2-93ec00d6a5df" />

#### What is Redis actually doing?

Improves client-server communication:

- Used for pub/sub messaging — a message broker that coordinates streaming events between the server and clients
- Critical when you have multiple server replicas that need to coordinate streaming responses to the same client
- If running a single server instance without streaming, Redis is optional (though the default setup includes it)

---

## Try it yourself

1. Clone a template repo from [langgraph-template · GitHub Topics](https://github.com/topics/langgraph-template): `git clone <repo_url>`
2. Create and activate a virtual environment: `uv venv`
3. Install langgraph-cli: `uv add langgraph-cli`
4. Install dependencies: `uv sync`
5. Create Dockerfile using the LangGraph official command: `langgraph dockerfile Dockerfile`
6. Create `docker-compose.yml` from the [official docs](https://docs.langchain.com/langsmith/deploy-standalone-server)
7. Build the image: `docker build -t <IMAGE_NAME> .`
8. Copy env example: `cp .env.example .env`
9. Populate `.env` with:
   - `IMAGE_NAME` — the name used when building the image, so Compose knows which container to create
   - `LANGSMITH_API_KEY`
10. Start services: `docker compose up -d`
11. Verify 3 containers are healthy (agent, postgres, redis): `docker ps`
12. Open LangGraph Studio: `https://smith.langchain.com/studio/?baseUrl=http://0.0.0.0:8123`

## Resources

- [Docker Cheat Sheet](https://docs.docker.com/get-started/docker_cheatsheet.pdf)
- [Self-hosted LangSmith - Docs by LangChain](https://docs.langchain.com/langsmith/self-hosted#standalone-server)
