# Exercice 4 — Containeriser votre agent avec Docker

## Objectif

Containeriser l'agent fil-rouge dans une image Docker fonctionnelle, exposant une API REST sur le port 8000 avec un healthcheck integre.

---

## Etape 1 — Preparer les fichiers

> Verifiez que requirements.txt est a jour. Creez un .dockerignore (exclure \_\_pycache\_\_, .env, .git, chroma_db/).

### Ce qu'on a fait

**`requirements.txt`** — ajout de `fastapi` et `uvicorn` pour exposer l'agent en API :

```
openai>=1.30.0
feedparser>=6.0.11
requests>=2.31.0
python-dotenv>=1.0.0
numpy>=1.26.0
fastapi>=0.111.0
uvicorn>=0.30.0
```

**`.dockerignore`** — exclut tout ce qui n'a rien a faire dans l'image :

```
__pycache__
.env
.git
.gitignore
chroma_db/
chromadb_data/
data/
*.pyc
.venv/
tests/
docs/
README.md
```

**`api.py`** — wrapper FastAPI autour de `agent_react` (le fil-rouge etait en mode CLI, pas en API) :

```python
from fastapi import FastAPI
from pydantic import BaseModel
from main import agent_react

app = FastAPI(title="Agent Veille Technologique", version="1.0.0")

class AskRequest(BaseModel):
    question: str

class AskResponse(BaseModel):
    reponse: str

@app.get("/health")
def health():
    return {"status": "ok"}

@app.post("/ask", response_model=AskResponse)
def ask(req: AskRequest):
    reponse = agent_react(req.question)
    return AskResponse(reponse=reponse)
```

---

## Etape 2 — Ecrire le Dockerfile

> Base : python:3.11-slim. Copiez requirements.txt d'abord, installez, puis copiez le code. Exposez le port 8000. Ajoutez un HEALTHCHECK.

### Ce qu'on a fait

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Installer les dependances d'abord (cache Docker)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copier le code source
COPY . .

# Creer le dossier data (utilise au runtime)
RUN mkdir -p /app/data

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')" || exit 1

CMD ["uvicorn", "api:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Points cles :**
- `COPY requirements.txt` avant `COPY .` — exploite le cache Docker (les dependances ne sont reinstallees que si requirements.txt change)
- `--no-cache-dir` — reduit la taille de l'image (~100 Mo economises)
- `HEALTHCHECK` — utilise `urllib` (stdlib Python) pour ne pas ajouter `curl` dans l'image slim
- `mkdir -p /app/data` — cree le dossier data attendu par `config.py` au runtime

---

## Etape 3 — Construire et tester

> docker build, docker run, curl /health et /ask

### Commandes executees

```bash
# Build (33 secondes)
docker build -t agent-fil-rouge:v1 .

# Run — utiliser --env-file pour passer le .env directement
# (la variable $OPENAI_API_KEY n'est pas exportee dans le shell par defaut)
docker run -d -p 8000:8000 \
  --env-file .env \
  --name mon-agent \
  agent-fil-rouge:v1

# Test health
curl http://localhost:8000/health

# Test ask
curl -X POST http://localhost:8000/ask \
  -H "Content-Type: application/json" \
  -d '{"question": "Bonjour"}'
```

> **Note** : la commande de l'enonce utilise `-e OPENAI_API_KEY=$OPENAI_API_KEY`, mais cela suppose que la variable est exportee dans le shell. Avec `--env-file .env`, le fichier `.env` est lu directement par Docker — plus fiable.

### Resultats

| Etape | Resultat | OK ? |
|---|---|---|
| docker build sans erreur | Build 33s, image `agent-fil-rouge:v1` creee | oui |
| docker run demarre | Container demarre (port 8000, `--env-file .env`) | oui |
| /health repond 200 | `{"status":"ok"}` | oui |
| /ask retourne une reponse | `{"reponse":"Bonjour ! Comment puis-je vous aider aujourd'hui ?"}` | oui |

---

## Fichiers crees/modifies

| Fichier | Role |
|---|---|
| `fil-rouge/api.py` | Wrapper FastAPI (`GET /health`, `POST /ask`) |
| `fil-rouge/.dockerignore` | Exclusions pour le build Docker |
| `fil-rouge/Dockerfile` | Image Docker (python:3.11-slim + healthcheck) |
| `fil-rouge/requirements.txt` | Ajout fastapi + uvicorn |

## Livrable

Dockerfile fonctionnel, image construite et testee.
