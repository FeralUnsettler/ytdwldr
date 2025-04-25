# Streamlit para baixar vídeos do YouTube e que use OAuth para login (permitindo acesso a vídeos privados)

- **Streamlit** para interface.
- **`pytube` ou `youtube_dl`** para download de vídeos.
- **OAuth 2.0 com Google/Youtube API** para acesso autenticado, incluindo vídeos privados.
- **`google-auth`, `google-auth-oauthlib` e `google-api-python-client`** para integração com o Google.

Abaixo, um **exemplo funcional** com Streamlit para autenticação com OAuth e download de vídeo via URL atual (com `pytube`). Note que o acesso a vídeos privados só funcionará após autenticação correta e permissões dadas via Google Cloud Console:

---

### 🧠 Pré-requisitos

1. Vá até [Google Cloud Console](https://console.cloud.google.com/)  
2. Crie um projeto.
3. Ative a **YouTube Data API v3**.
4. Configure uma tela de consentimento OAuth e crie uma credencial do tipo **OAuth 2.0 (Web Application)**.
5. Salve o `client_secret.json`.

---

### ✅ Código do Streamlit (`app.py`)

```python
import streamlit as st
from pytube import YouTube
import os
import google.auth.transport.requests
import google_auth_oauthlib.flow
import googleapiclient.discovery
import pickle

st.set_page_config(page_title="YouTube Downloader com OAuth", layout="centered")

st.title("🔐 YouTube Downloader com OAuth")
st.markdown("Faça login com sua conta Google para acessar vídeos privados e baixe a URL atual do YouTube.")

CLIENT_SECRETS_FILE = "client_secret.json"
SCOPES = ["https://www.googleapis.com/auth/youtube.readonly"]
API_SERVICE_NAME = "youtube"
API_VERSION = "v3"

if "credentials" not in st.session_state:
    st.session_state.credentials = None

def authenticate_user():
    flow = google_auth_oauthlib.flow.Flow.from_client_secrets_file(
        CLIENT_SECRETS_FILE, scopes=SCOPES)
    flow.redirect_uri = "http://localhost:8501"
    authorization_url, state = flow.authorization_url(
        access_type="offline", include_granted_scopes="true")

    st.session_state.flow = flow
    st.markdown(f"[Clique aqui para fazer login com o Google]({authorization_url})")

def complete_auth(code):
    flow = st.session_state.flow
    flow.fetch_token(code=code)
    credentials = flow.credentials
    st.session_state.credentials = credentials
    st.success("✅ Autenticado com sucesso!")

if "code" in st.experimental_get_query_params():
    code = st.experimental_get_query_params()["code"][0]
    complete_auth(code)

if not st.session_state.credentials:
    authenticate_user()
    st.stop()

youtube = googleapiclient.discovery.build(
    API_SERVICE_NAME, API_VERSION, credentials=st.session_state.credentials)

st.subheader("🎥 Baixar vídeo do YouTube")

url = st.text_input("Cole a URL do vídeo (ou vá até o vídeo e copie o link atual):")

if url:
    try:
        yt = YouTube(url)
        st.write(f"**Título:** {yt.title}")
        stream = yt.streams.get_highest_resolution()

        if st.button("📥 Baixar vídeo"):
            stream.download(output_path="downloads")
            st.success("Vídeo baixado com sucesso na pasta 'downloads'.")

    except Exception as e:
        st.error(f"Erro: {e}")
```

---

### 📁 Estrutura Esperada

```
📁 projeto
│
├── app.py
├── client_secret.json
└── downloads/
```

---

### ⚠️ Observações importantes:

- Streamlit **não é ideal** para lidar com callbacks OAuth diretos. Isso é feito aqui com workaround, redirecionando localmente.
- Para produção, considere hospedar isso com HTTPS e configurar o redirect URI corretamente.
- Vídeos **privados só podem ser acessados se o usuário autenticado for o dono** ou tiver acesso permitido ao vídeo.

Quer que eu te ajude com o `client_secret.json` ou com os passos no Google Cloud?
