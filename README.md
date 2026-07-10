# Jenkins Pipeline - Build e Push di un'immagine Docker con Podman

Questa pipeline Jenkins automatizza il processo di:

- clonazione del repository Git;
- scelta del branch da compilare;
- generazione automatica del tag dell'immagine;
- build dell'immagine tramite **Podman**;
- pubblicazione dell'immagine su **Docker Hub**.

---

# Struttura della pipeline

## Variabile globale

```groovy
def IMAGE_TAG = ''
```

### Cosa fa

Questa variabile conterrà il **tag finale** dell'immagine Docker.

All'inizio è vuota perché il valore viene deciso successivamente in base al branch oppure ad un eventuale tag Git.

---

## Pipeline

```groovy
pipeline {
```

Questa istruzione definisce una **Declarative Pipeline** di Jenkins.

All'interno vengono specificati:

- agente di esecuzione
- parametri
- variabili d'ambiente
- stage della pipeline

---

# Agent

```groovy
agent {
    label 'docker-agent'
}
```

### Cosa fa

Indica su quale nodo Jenkins verrà eseguita la pipeline.

In questo caso Jenkins cercherà un nodo con la label:

```
docker-agent
```

che deve avere installato Podman e tutti gli strumenti necessari.

---

# Parametri

```groovy
parameters {
    choice(
        name: 'BRANCH',
        choices: ['main', 'develop']
    )
}
```

### Cosa fa

Quando viene lanciata la pipeline, l'utente può scegliere quale branch compilare.

Sono disponibili:

- `main`
- `develop`

Questo rende la pipeline riutilizzabile senza modificarne il codice.

---

# Variabili d'ambiente

```groovy
environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    IMAGE_NAME = "mariannabrnrd/flask-app-example"
}
```

## Docker Hub Credentials

```groovy
credentials('dockerhub-credentials')
```

Recupera username e password salvati nelle **Credentials di Jenkins**.

In questo modo le credenziali:

- non sono scritte nel codice;
- rimangono protette.

---

## Nome dell'immagine

```groovy
IMAGE_NAME = "mariannabrnrd/flask-app-example"
```

Specifica il repository Docker Hub dove verrà pubblicata l'immagine.

---

# Stage 1 - Checkout

```groovy
stage('Checkout')
```

## Codice

```groovy
checkout([
    $class: 'GitSCM',
    branches: [[name: "${params.BRANCH}"]],
    userRemoteConfigs: [[
        url: 'https://github.com/...'
    ]]
])
```

### Cosa fa

Clona il repository Git e scarica il branch selezionato.

Il branch viene letto dal parametro:

```groovy
params.BRANCH
```

Questo permette di compilare repository diversi senza modificare la pipeline.

---

# Stage 2 - Set Image Tag

Questo è probabilmente lo stage più importante.

```groovy
if (env.TAG_NAME) {
    IMAGE_TAG = env.TAG_NAME
}
```

## Caso 1 - Build da Tag Git

Se la pipeline è stata avviata tramite un **tag Git**, l'immagine userà direttamente quel tag.

Esempio:

```
v1.0.0
```

diventa

```
flask-app-example:v1.0.0
```

---

```groovy
else if (params.BRANCH == 'main') {
    IMAGE_TAG = 'latest'
}
```

## Caso 2 - Branch main

Se il branch è **main**, l'immagine viene pubblicata con il tag:

```
latest
```

ottenendo:

```
flask-app-example:latest
```

---

```groovy
else if (params.BRANCH == 'develop') {
    IMAGE_TAG = "develop-${env.GIT_COMMIT[0..6]}"
}
```

## Caso 3 - Branch develop

Per il branch **develop** viene aggiunto il prefisso:

```
develop-
```

seguito dai primi 7 caratteri dell'hash del commit.

Esempio:

Commit

```
1f8b76a92c...
```

Tag generato

```
develop-1f8b76a
```

In questo modo ogni build è facilmente identificabile.

---

```groovy
else {
    IMAGE_TAG = "branch-${env.GIT_COMMIT[0..6]}"
}
```

## Caso 4 - Altri branch

Per eventuali branch differenti viene utilizzato il formato:

```
branch-<hash_commit>
```

---

# Stage 3 - Build

```groovy
sh "podman build -t ${IMAGE_NAME}:${IMAGE_TAG} ./app"
```

### Cosa fa

Costruisce l'immagine usando Podman.

Il Dockerfile deve trovarsi nella cartella:

```
./app
```

L'immagine prodotta avrà un nome simile a:

```
mariannabrnrd/flask-app-example:latest
```

oppure

```
mariannabrnrd/flask-app-example:develop-1f8b76a
```

---

# Stage 4 - Push

## Login

```groovy
echo $DOCKERHUB_CREDENTIALS_PSW | podman login docker.io \
-u $DOCKERHUB_CREDENTIALS_USR \
--password-stdin
```

### Cosa fa

Effettua il login su Docker Hub utilizzando le credenziali salvate in Jenkins.

L'opzione

```
--password-stdin
```

evita di mostrare la password nei log della pipeline.

---

## Push

```groovy
podman push ${IMAGE_NAME}:${IMAGE_TAG}
```

### Cosa fa

Carica l'immagine appena costruita sul repository Docker Hub.

Ad esempio:

```
mariannabrnrd/flask-app-example:latest
```

oppure

```
mariannabrnrd/flask-app-example:develop-1f8b76a
```

---

# Flusso completo della pipeline

```
Scelta del branch
        │
        ▼
Checkout del repository
        │
        ▼
Generazione del tag
        │
        ▼
Build immagine con Podman
        │
        ▼
Login su Docker Hub
        │
        ▼
Push dell'immagine
```

---

# Vantaggi di questa pipeline

- Automazione completa del processo di build.
- Gestione sicura delle credenziali tramite Jenkins.
- Tag automatici in base al branch o al tag Git.
- Utilizzo di Podman al posto di Docker.
- Pipeline facilmente estendibile con test, analisi del codice o deployment.