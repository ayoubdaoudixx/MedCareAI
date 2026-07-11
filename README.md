# MedCareAI — Medical Chatbot with LLMs, LangChain, Pinecone & Flask

MedCareAI is a Retrieval-Augmented Generation (RAG) chatbot that answers medical
questions grounded in a medical knowledge base. It ingests a medical reference
(the *Gale Encyclopedia of Medicine*), embeds the content into a Pinecone vector
database, and uses a Groq-hosted **Llama 3.3 70B** model through LangChain to
generate concise, context-aware answers served via a Flask web app.

> ⚕️ **Disclaimer:** MedCareAI is a demo/educational project. It is **not** a
> substitute for professional medical advice, diagnosis, or treatment.

---

## 🧠 How it works

1. **Ingestion** — Medical PDFs in `data/` are loaded and split into ~500-token
   chunks (`RecursiveCharacterTextSplitter`).
2. **Embeddings** — Each chunk is embedded with the HuggingFace
   `sentence-transformers/all-MiniLM-L6-v2` model (384-dimensional vectors).
3. **Vector store** — Embeddings are upserted into a Pinecone serverless index
   (`medical-chatbot`, cosine similarity).
4. **Retrieval + Generation** — At query time the most relevant chunks are
   retrieved and passed as context to the Groq **Llama 3.3 70B** model via a
   LangChain retrieval chain, which returns a grounded answer.
5. **Serving** — A Flask app exposes a simple chat UI.

---

## 🧰 Tech stack

- **Python 3.10**
- **LangChain** — orchestration & retrieval chain
- **HuggingFace** `all-MiniLM-L6-v2` — sentence embeddings
- **Pinecone** — serverless vector database
- **Groq** — LLM inference (`llama-3.3-70b-versatile`)
- **Flask** — web server / chat interface
- **PyPDF** — PDF ingestion

---

## 🚀 How to run

### STEP 01 — Clone the repository

```bash
git clone https://github.com/ayoubdaoudixx/MedCareAI.git
cd MedCareAI
```

### STEP 02 — Create a conda environment

```bash
conda create -n medcareai python=3.10 -y
conda activate medcareai
```

### STEP 03 — Install the requirements

```bash
pip install -r requirements.txt
```

### STEP 04 — Set up environment variables

Create a `.env` file in the project root and add your Pinecone & Groq
credentials:

```ini
PINECONE_API_KEY="xxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
GROQ_API_KEY="xxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

> Get a free Pinecone key at [pinecone.io](https://www.pinecone.io/) and a Groq
> key at [console.groq.com](https://console.groq.com/keys).
> `.env` is gitignored — never commit your keys.

### STEP 05 — Build the vector index

Embed the documents in `data/` and store them in Pinecone:

```bash
python store_index.py
```

### STEP 06 — Run the app

```bash
python app.py
```

Then open your browser at:

```
http://localhost:8080
```

---

## 📁 Project structure

```
MedCareAI/
├── data/                 # Source medical PDFs (Gale Encyclopedia of Medicine)
├── research/
│   └── trials.ipynb      # End-to-end RAG experimentation notebook
├── src/
│   ├── helper.py         # PDF loading, chunking & embedding helpers
│   └── prompt.py         # System / RAG prompt templates
├── store_index.py        # One-off script to build the Pinecone index
├── app.py                # Flask application
├── requirements.txt
├── setup.py
└── README.md
```

---

# ☁️ AWS CI/CD Deployment with GitHub Actions

## 1. Log in to the AWS console

## 2. Create an IAM user for deployment

**With specific access:**

1. **EC2 access** — virtual machine to host the app
2. **ECR** — Elastic Container Registry to store the Docker image

**Deployment steps this pipeline performs:**

1. Build a Docker image of the source code
2. Push the Docker image to ECR
3. Launch an EC2 instance
4. Pull the image from ECR into EC2
5. Run the Docker container on EC2

**Policies to attach:**

1. `AmazonEC2ContainerRegistryFullAccess`
2. `AmazonEC2FullAccess`

## 3. Create an ECR repository to store the Docker image

Save the repository URI, e.g.:

```
<your-account-id>.dkr.ecr.us-east-1.amazonaws.com/medcareai
```

## 4. Create an EC2 machine (Ubuntu)

## 5. Install Docker on the EC2 machine

```bash
# optional
sudo apt-get update -y
sudo apt-get upgrade -y

# required
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu
newgrp docker
```

## 6. Configure EC2 as a self-hosted runner

`Settings > Actions > Runners > New self-hosted runner > choose OS >` then run
the provided commands one by one.

## 7. Set up GitHub secrets

| Secret | Description |
| --- | --- |
| `AWS_ACCESS_KEY_ID` | IAM access key |
| `AWS_SECRET_ACCESS_KEY` | IAM secret key |
| `AWS_DEFAULT_REGION` | e.g. `us-east-1` |
| `ECR_REPO` | ECR repository URI |
| `PINECONE_API_KEY` | Pinecone API key |
| `GROQ_API_KEY` | Groq API key |

---

## 👤 Author

**Ayoub Daoudi** — [ayoubdaoudi2001@gmail.com](mailto:ayoubdaoudi2001@gmail.com)

## 📄 License

This project is licensed under the terms of the [LICENSE](LICENSE) file.
