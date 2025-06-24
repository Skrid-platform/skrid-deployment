# SKRID Deployment

This repository provides Docker-based orchestration for the SKRID platform.

> **Note**: This repository only handles setup and deployment. For details on functionality and usage, refer to the individual repositories:
>
> * [Frontend README](https://gitlab.inria.fr/skrid/frontend/-/blob/main/README.md)
> * [Backend README](https://gitlab.inria.fr/skrid/backend/-/blob/main/README.md)

---

## Requirements

* [Docker](https://docs.docker.com/get-docker/)
* [Docker Compose](https://docs.docker.com/compose/)

---

## Quick Setup

1. **Clone this repository and the frontend/backend repositories** side by side:

   ```bash
   git clone git@gitlab.inria.fr:skrid/skrid-deployment.git
   cd skrid-deployment
   git clone git@gitlab.inria.fr:skrid/frontend.git
   git clone git@gitlab.inria.fr:skrid/backend.git
   ```

2. **Create the `.env` file** by copying the example and adjusting values if needed:

   ```bash
   cp .env.example .env
   ```

3. **Initialize the Neo4j volume** (this will create `neo4j/neo4j_data/`):

   ```bash
   docker compose up neo4j
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

5. **Asset generation** (PDF, SVG, MID, MSCZ):

Assets need to be generated for front-end display. Go to [Data ingestion](https://gitlab.inria.fr/skrid/data-ingestion) for a CLI tool which transforms MEI files into cypher, pdf, midi and SVG.

6. **Launch the full platform** (Neo4j, backend, frontend):

   ```bash
   docker compose up --build
   ```

   Access the application at [http://localhost:3000](http://localhost:3000)

---

## Alternative: Run a single container

You can also run only the database, for development or debugging:

```bash
docker compose up neo4j
```

And run the frontend or backend locally from their respective folders:

* **Frontend** (`./frontend`):

  ```bash
  npm install
  node index.js
  ```

* **Backend** (`./backend`):

  ```bash
  pip install -r requirements.txt
  python3 api.py
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
* Each project (frontend/backend) has its own Git repository and is mounted locally in this orchestration setup.
