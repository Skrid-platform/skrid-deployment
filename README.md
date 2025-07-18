# SKRID Deployment

This repository provides Docker-based orchestration for the SKRID platform.

> **Note**: This repository only handles setup and deployment. For details on functionality and usage, refer to the individual repositories:
>
> * [Backend README](https://gitlab.inria.fr/skrid/backend/-/blob/main/README.md)
> * [Frontend README](https://gitlab.inria.fr/skrid/frontend/-/blob/main/README.md)
> * [Client README](https://gitlab.inria.fr/skrid/client/-/blob/main/README.md)

---

## Requirements
* [Docker](https://docs.docker.com/get-docker/)
* [Docker Compose](https://docs.docker.com/compose/)

---

## Quick Setup (all with containers)
1. **Clone this repository and the frontend/backend repositories** side by side:

   ```bash
   git clone --depth=1 https://gitlab.inria.fr/skrid/skrid-deployment.git
   cd skrid-deployment/
   git clone --depth=1 https://gitlab.inria.fr/skrid/frontend.git
   git clone --depth=1 https://gitlab.inria.fr/skrid/backend.git
   ```

   Note that it is not needed to clone the client for production, as it will be cloned and built inside the frontend docker container.

2. **Create the `.env` file** by copying the example and adjusting values if needed:

   ```bash
   cp .env.example .env
   ```

3. **Initialize the Neo4j volume** (this will create `neo4j/neo4j_data/`):

   ```bash
   docker compose up neo4j --build
   ```

   Then stop the container with `Ctrl+C` once it's ready.

4. **Load the graph dump into the database** (once Neo4j is initialized):

   ```bash
   docker run --rm --name neo4j-admin-loader  \
      -v "$(pwd)/neo4j/neo4j_data":/data  \
      -v "$(pwd)/neo4j/neo4j_import":/import  \
      -v "$(pwd)/neo4j/neo4j_config/neo4j.conf":/conf/neo4j.conf  \
      -e NEO4J_AUTH=none   neo4j:4.2.19-community  \
      neo4j-admin load --from=/import/graph.dump --database=neo4j --force
   ```

5. **Build the containers** (backend, frontend):

   ```bash
   docker compose up --build
   ```

   This will build the backend and the frontend.

   For the frontend, it will take a while, because the following tasks will be realised:
   - cloning and building the [client](https://gitlab.inria.fr/skrid/client) ;
   - cloning and building verovio (needed to generate the other data formats, at next step) ;
   - cloning and generating other formats of [data](https://gitlab.inria.fr/skrid/data).

   Then stop the containers with `Ctrl+C` once it's ready.

6. **Launch the full platform** (Neo4j, backend, frontend):

   ```bash
   docker compose up
   ```

   Access the application at [http://localhost:3000](http://localhost:3000)

---

## Development setup
For development and debugging, it is advised to run only the database in a container, and the rest without.

- **Database**:
```
docker compose up neo4j
```

- **Backend** (`./backend`, see the [readme](https://gitlab.inria.fr/skrid/backend/-/blob/main/README.md) there for more details):
```
pip install -r requirements.txt
python3 api.py
```

- **Frontend server** (`./frontend`, see the [readme](https://gitlab.inria.fr/skrid/frontend/-/blob/main/README.md) there for more details):
```
npm install
npm run nodemon
```

- **Client (vueJS)** (`./client`, see the [readme](https://gitlab.inria.fr/skrid/client/-/blob/main/README.md) there for more details):
```
npm install
npm run dev
```

---

## Environment Variables
Defined in `.env` (not versioned):

```env
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password
```

Defaults for `API_BASE_URL` and `BASE_PATH` are handled by `docker-compose.yml` and each component's code logic.

---

## Manually add cypher files
To manually add a `.cypher` file, you can run the command:
```
docker exec -i skrid-neo4j bin/cypher-shell -u [NEO4J_USER] -p [NEO4J_PASSWORD] < path/to/file.cypher
```

---

## Notes
* The initial setup may take a few minutes on first run (image builds, network setup, etc.).
* Neo4j data is stored persistently in `neo4j/neo4j_data`.
* Each part of the project (client, frontend, backend, data, data-ingestion) has its own Git repository and is mounted locally in this orchestration setup.
