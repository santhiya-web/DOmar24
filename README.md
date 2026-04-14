# DevOps Training: Next.js + FastAPI on Google Cloud Run

This project is a training ground for DevOps consultants to practice deploying a modern full-stack application (Next.js + FastAPI) to Google Cloud Platform (GCP) using Cloud Run and GitHub Actions.

## Project Structure

- `frontend/`: Next.js application (React, Tailwind CSS, TypeScript).
- `backend/`: FastAPI application (Python, in-memory dictionary storage).
- `.github/workflows/`: CI/CD pipelines.

---

## Local Development

### 1. Prerequisites
- **Node.js 18+** & **npm**
- **Python 3.11+**
- **Docker** (optional, for local container testing)

### 2. Setup & Configuration

Copy the `.env.example` files to create your local environment configurations:

**Frontend:**
```bash
cd frontend
cp .env.example .env
```
_Edit `frontend/.env` and ensure `NEXT_PUBLIC_API_URL` is set to `http://localhost:8080`._

**Backend:**
```bash
cd backend
cp .env.example .env
```

### 3. Running the Backend (FastAPI)
```bash
cd backend
python -m venv venv
source venv/bin/activate  # On Windows use `venv\Scripts\activate`
pip install -r requirements.txt
uvicorn main:app --reload --port 8080
```
The API will be available at `http://localhost:8080`. You can view the automatic docs at `http://localhost:8080/docs`.

### 4. Running the Frontend (Next.js)
```bash
cd frontend
npm install
npm run dev
```
The app will be available at `http://localhost:3000`.

---

## Testing

### Backend Tests
```bash
cd backend
pytest
```

### Frontend Tests
```bash
cd frontend
npm test
```

---

## Deployment to GCP Cloud Run

### CI/CD with GitHub Actions
This project uses two workflows located in `.github/workflows/`:

1. **CI (`ci.yml`)**: Runs on every pull request to `main`. It installs dependencies and executes tests for both frontend and backend.
2. **CD (`cd.yml`)**: Runs on merge/push to `main`. It builds Docker images, pushes them to Google Artifact Registry, and deploys them to Cloud Run.

### Required Secrets
To enable the CD pipeline, you must set the following **Actions Secrets** in your GitHub Repository Settings:

- `GCP_PROJECT_ID`: Your Google Cloud Project ID.
- `GCP_SERVICE_ACCOUNT`: The email of the service account used for deployment.
- `GCP_WORKLOAD_IDENTITY_PROVIDER`: The full name of your Workload Identity Provider.

**Note**: You must also manually create the Artifact Registry repository named `containers` in the `us-central1` region before the first deployment.

Refer to [SECRETS.md](./SECRETS.md) for the exact `gcloud` commands and full setup details.

---

## Infrastructure Highlights

- **Cloud Run**: Both services are containerized and deployed as serverless functions.
- **Artifact Registry**: Used to store container images.
- **Workload Identity Federation**: Secure, keyless authentication between GitHub Actions and GCP.
- **Multi-stage Dockerfiles**: Optimized for speed and security.
- **CORS Configuration**: The backend is configured to safely allow requests from the frontend origin.
