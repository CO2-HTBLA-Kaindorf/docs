# Push Docker image to ghcr.io

## Erzeugen eines Access Tokens

Unter `github - profile picture - Settings - Developer Settings - Personal access tokens - Tokens (classic)` einen `private access token`
erzeugen

Login mit den github Zugangsdaten bei ghcr.io

```bash
docker login --username <gh-username> --password <privat-access-token> ghcr.io
```

und push das oben erstellte image nach ghcr.io

```bash
docker push ghcr.io/<gh-username>/co2backend:latest
```
