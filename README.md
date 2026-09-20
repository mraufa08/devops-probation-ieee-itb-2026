# Indore Route Planner

A web application for finding routes in Indore using a React frontend, Express backend, and MongoDB database.

This project was prepared as part of the IEEE ITB DevOps Engineer probation task. The main focus is to make the application easy to run, test, and maintain using Docker, Docker Compose, environment configuration, health checks, logging, and GitHub Actions CI.

## Architecture

```text
                    Docker Compose
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   Frontend          Backend           MongoDB
   React/Vite       Express.js          MongoDB
    :5173             :5000             :27017
        │                │
        └───────────────►│
                         │
                         ▼
                      MongoDB
```

### Services

| Service | Technology | Port | Fuction |
|---|---|---:|---|
| Frontend | React + Vite | 5173 | User interface |
| Backend | Node.js + Express | 5000 | API  |
| Database | MongoDB | 27017 | Stores application data |

All services are managed through one Docker Compose configuration.

## Project Structure

```text
indore-route-planner/
├── frontend/
│   ├── src/
│   ├── package.json
│   ├── package-lock.json
│   └── Dockerfile
│
├── backend/
│   ├── models/
│   ├── routes/
│   ├── utils/
│   ├── server.js
│   ├── package.json
│   ├── package-lock.json
│   └── Dockerfile
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── .env
├── .env.example
├── .gitignore
├── docker-compose.yml
└── README.md
```

## Tools Used

- **Docker** : runs each service in a container.
- **Docker Compose** : starts and manages the complete application.
- **Node.js** : runs the frontend build tools and backend.
- **React and Vite** : builds the frontend.
- **MongoDB** : stores application data.
- **GitHub Actions** : runs automated CI checks.
- **Git/GitHub** : tracks and stores the project source code.

## Requirements

Install the following before running the project:

- Docker Desktop
- Git

Docker Desktop includes Docker Compose.

Verifying by run this command to see the version of them.
```bash
docker --version
```

```bash
docker compose version
```

```bash
git --version
```

## Environment Configuration

The project uses environment variables for configuration.

Create a `.env` file in the project root:

```env
MONGO_URI=mongodb://mongodb:27017/indore_metro
PORT=5000
```

A template is also provided in `.env.example`:

```env
MONGO_URI=mongodb://mongodb:27017/indore_metro
PORT=5000
```

The `.env` file is ignored by Git and should not be committed.

### Important

MongoDB hostname is:

```text
mongodb
```

This is the Docker Compose service name.

Do not change it to `localhost` when the backend is running inside Docker. Inside the backend container, `localhost` refers to the backend container itself.

## Run the Application

Open a terminal in the project root.

Build and start all services:

```bash
docker compose up -d --build
```

Check the service status:

```bash
docker compose ps
```

The expected services are:

```text
indore-mongodb
indore-backend
indore-frontend
```

### Open the Application

Frontend:

```text
http://localhost:5173
```

Backend:

```text
http://localhost:5000
```

## Health Check

The backend provides a health endpoint:

```text
http://localhost:5000/health
```

A healthy response looks like:

```json
{"status": "healthy", "database": "connected"}
```

The endpoint checks whether the backend can connect to MongoDB.

MongoDB also has a Docker health check. The backend waits for MongoDB to become healthy before starting.

## Logs

Docker Compose provides access to service logs.

View backend logs:

```bash
docker compose logs backend
```

View MongoDB logs:

```bash
docker compose logs mongodb
```

View frontend logs:

```bash
docker compose logs frontend
```

Follow new backend logs:

```bash
docker compose logs -f backend
```

Press `Ctrl + C` to stop viewing the logs.

The containers continue running after this command is stopped.

## Stop and Restart

Stop the application:

```bash
docker compose down
```

Start it again:

```bash
docker compose up -d
```

Restart a specific service:

```bash
docker compose restart backend
```

Check the service status:

```bash
docker compose ps
```

## Database Persistence

MongoDB uses a Docker volume:

```yaml
volumes:
  - mongodb_data:/data/db
```

The volume keeps MongoDB data when the containers are stopped or recreated.

To stop the application without deleting the database volume:

```bash
docker compose down
```

Do not use:

```bash
docker compose down -v
```

unless you intentionally want to remove the stored database volume.

## Docker Configuration

The frontend and backend each have their own Dockerfile.

The backend container:

1. Uses Node.js.
2. Installs the backend dependencies.
3. Copies the application files.
4. Starts the Express server.

The frontend container:

1. Uses Node.js.
2. Installs the frontend dependencies.
3. Builds the React application.
4. Starts the Vite server.

MongoDB runs using the official MongoDB Docker image.

## CI Pipeline

GitHub Actions runs automatically when code is pushed to the `main` branch or when a pull request targets `main`.

The workflow is located at:

```text
.github/workflows/ci.yml
```

The pipeline has three jobs.

### Frontend Checks

```text
npm ci
↓
npm run lint
↓
npm run build
```

This checks that the frontend dependencies install correctly, the code passes linting, and the application can be built.

### Backend Checks

```text
npm ci
↓
node --check server.js
```

This checks that the backend dependencies install and that the server file has valid JavaScript syntax.

### Docker Build

```text
docker compose build
```

This checks that the Docker images can be built successfully.

The CI pipeline was also tested with an intentional backend syntax error to confirm that the workflow detects broken code.

## Troubleshooting

### 1. Backend is not healthy

Check the backend logs:

```bash
docker compose logs backend
```

Then check MongoDB:

```bash
docker compose logs mongodb
```

Make sure MongoDB is healthy:

```bash
docker compose ps
```

### 2. Frontend does not open

Check the frontend container:

```bash
docker compose ps
```

Then view its logs:

```bash
docker compose logs frontend
```

Make sure port `5173` is available on the host machine.

### 3. Backend cannot connect to MongoDB

Check the `.env` file:

```env
MONGO_URI=mongodb://mongodb:27017/indore_metro
```

The hostname must be `mongodb` when both services are running through Docker Compose.

Then restart the application:

```bash
docker compose down
docker compose up -d --build
```

### Need to see all service logs

Run:

```bash
docker compose logs
```

## Commands

| Task | Command |
|---|---|
| Build and start | `docker compose up -d --build` |
| Stop services | `docker compose down` |
| Check status | `docker compose ps` |
| Backend logs | `docker compose logs backend` |
| Frontend logs | `docker compose logs frontend` |
| MongoDB logs | `docker compose logs mongodb` |
| Follow logs | `docker compose logs -f backend` |
| Restart backend | `docker compose restart backend` |
| Rebuild images | `docker compose build` |

## Known Issues

- The backend currently shows MongoDB driver warnings for `useNewUrlParser` and `useUnifiedTopology`. These options are deprecated in the current MongoDB driver, but they do not prevent the application from running.
- The backend does not have a full automated unit test suite. The current CI checks JavaScript syntax instead of using the placeholder `npm test` command.
- The application is configured for local Docker-based use. Production deployment is outside the scope of this setup.

## AI Involvement

AI tools were used as development assistance during the DevOps setup.

They were used to help with:

- Understanding Docker and Docker Compose configuration.
- Creating and reviewing Dockerfiles.
- Setting up GitHub Actions CI.
- Troubleshooting container and service configuration.
- Improving project documentation.

All configurations were reviewed and tested in the local environment before being used.

## Verification

The following parts of the project were tested successfully:

- Frontend runs in a Docker container.
- Backend runs in a Docker container.
- MongoDB runs as part of the Docker Compose environment.
- Backend connects successfully to MongoDB.
- `/health` reports a healthy backend and connected database.
- MongoDB has persistent Docker storage.
- Environment variables are loaded through `.env`.
- Docker Compose can build the services.
- GitHub Actions frontend checks pass.
- GitHub Actions backend checks pass.
- GitHub Actions Docker build passes.
- CI correctly detects an intentional backend syntax error.
- Services can be restarted through Docker Compose.