# Documentación de instalación manual en Linux

Esta guía explica paso a paso cómo preparar tu entorno en Linux para el proyecto **Analista de Sentencias Ciudadanas**: instalar Python 3.12.7 con pyenv, Docker, Ollama, Tesseract OCR, clonar el repositorio y configurar `.env`.

---

## 1. Prerrequisitos comunes

Antes de nada, asegúrate de tener:

- Una cuenta con permisos de sudo.
- Conexión a Internet.
- Herramientas básicas: `curl`, `git`, `build-essential` (o su equivalente).

En Debian/Ubuntu:

```bash
sudo apt-get update
sudo apt-get install -y git curl build-essential
```

En CentOS/RHEL 7:

```bash
sudo yum install -y git curl gcc gcc-c++ make
```

En Fedora / RHEL 8+:

```bash
sudo dnf install -y git curl @development-tools
```

En Amazon Linux 2/2023:

```bash
sudo yum update -y
sudo yum install -y git curl gcc gcc-c++ make
```

---

## 2. Instalar pyenv y Python 3.12.7

1. Instala dependencias de compilación:

   Debian/Ubuntu:

   ```bash
   sudo apt-get install -y libssl-dev zlib1g-dev libbz2-dev \
     libreadline-dev libsqlite3-dev wget llvm libncurses5-dev \
     libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev
   ```

   CentOS/RHEL/Fedora:

   ```bash
   sudo yum/dnf install -y zlib-devel bzip2 bzip2-devel \
     readline-devel sqlite sqlite-devel openssl-devel tk-devel \
     libffi-devel xz-devel
   ```

2. Clona pyenv en tu home y configura la shell:

   ```bash
   git clone https://github.com/pyenv/pyenv.git ~/.pyenv
   echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
   echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
   echo 'eval "$(pyenv init --path)"'   >> ~/.bashrc
   source ~/.bashrc
   ```

3. Instala y activa Python 3.12.7:

   ```bash
   pyenv install -s 3.12.7
   pyenv global  3.12.7
   python3 -V   # Debe mostrar Python 3.12.7
   ```

---

## 3. Instalar Docker

### Amazon Linux 2/2023

```bash
# Con amazon-linux-extras (si está disponible)
sudo amazon-linux-extras install -y docker

# O con dnf
sudo dnf install -y docker
```

### Debian / Ubuntu

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo $ID)/gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
   https://download.docker.com/linux/$(. /etc/os-release; echo $ID) \
   $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

### CentOS 7 / RHEL 7

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io
```

### Fedora / RHEL 8+

```bash
sudo dnf install -y docker
```

**Finalmente**, habilita y arranca Docker en cualquiera de los casos:

```bash
sudo systemctl enable --now docker
docker --version
```

---

## 4. Instalar Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama ps   # Debe listar contenedores (vacío al inicio)
```

> ⚠️ Si indica “CPU-only mode” no te preocupes: funcionará sin GPU.

---

## 5. Instalar Tesseract OCR

### Amazon Linux 2/2023

```bash
# Habilitar EPEL
sudo amazon-linux-extras install -y epel

# Instalar Tesseract y datos en español
sudo yum install -y tesseract tesseract-langpack-spa
```

### Debian / Ubuntu

```bash
sudo apt-get update
sudo apt-get install -y tesseract-ocr tesseract-ocr-spa
```

### Fedora / RHEL 8+ / CentOS 8+

```bash
sudo dnf install -y epel-release
sudo dnf install -y tesseract tesseract-langpack-spa
```

Verifica:

```bash
tesseract --version
```

---

## 6. Clonar el repositorio y configurar variables

1. Clona tu proyecto:

   ```bash
   git clone https://github.com/Charlytoc/documents-ai.git .
   ```

2. Copia y edita el archivo de entorno:

   ```bash
   cp .env.example .env
   ```

3. En `.env` define al menos:

   ```
   PROVIDER=ollama
   MODEL=llama3.1:8b
   TESSERACT_CMD=$(which tesseract)
   ```

   Guarda los cambios.

---

## 7. Crear y activar el entorno virtual

```bash
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

---

## 8. (Opcional) Cliente React en modo desarrollo

```bash
cd client
npm install
npm run dev
cd ..
```

---

## 9. Iniciar servicios y la aplicación

1. Asegúrate de tener Redis en Docker:

   ```bash
   docker run -d --name document_redis -p 6380:6379 redis
   ```

2. Arranca Chroma:

   ```bash
   chroma run --path media/vector_storage/ --port 8004 &
   ```

3. Inicia FastAPI:
   ```bash
   python main.py
   ```

¡Listo! Tu entorno en Linux debería estar operativo.
