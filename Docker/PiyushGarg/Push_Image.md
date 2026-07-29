# Docker Hub Push Cheat Notes

* Create a Docker Hub account.
* Create a new repository (e.g., **dockerex1**).
* Login to Docker from the terminal:

```bash
docker login
```

* Enter your Docker Hub **username** and **password/access token**.

---

### Build the image with your Docker Hub repository name

```bash
docker build -t <username>/<repository-name>:latest .
```

Example:

```bash
docker build -t sujitajitumbalkar/dockerex1:latest .
```

---

### Verify the image

```bash
docker images
```

Example output:

```text
REPOSITORY                     TAG      IMAGE ID
sujitajitumbalkar/dockerex1    latest   abc123456789
```

---

### Push the image

```bash
docker push <username>/<repository-name>:latest
```

Example:

```bash
docker push sujitajitumbalkar/dockerex1:latest
```

(`:latest` is optional because it is the default tag.)

---

### Verify on Docker Hub

* Open your Docker Hub repository.
* Refresh the page.
* The uploaded image should appear.

---

## Optional: Push a Versioned Image

Tag the image:

```bash
docker tag sujitajitumbalkar/dockerex1:latest sujitajitumbalkar/dockerex1:v1
```

Push it:

```bash
docker push sujitajitumbalkar/dockerex1:v1
```

---

## Pull the Image on Another Machine

```bash
docker pull sujitajitumbalkar/dockerex1:latest
```

---

## Run the Pulled Image

```bash
docker run -p 8080:8080 sujitajitumbalkar/dockerex1:latest
```

---

## Complete Flow

```bash
docker login

docker build -t sujitajitumbalkar/dockerex1:latest .

docker images

docker push sujitajitumbalkar/dockerex1:latest

docker pull sujitajitumbalkar/dockerex1:latest

docker run -p 8080:8080 sujitajitumbalkar/dockerex1:latest
```

### Memory Flow

```text
Create Repository
        │
        ▼
docker login
        │
        ▼
docker build -t username/repository:tag .
        │
        ▼
docker images
        │
        ▼
docker push username/repository:tag
        │
        ▼
Docker Hub
        │
        ▼
docker pull username/repository:tag
        │
        ▼
docker run -p Host:Container username/repository:tag
```
