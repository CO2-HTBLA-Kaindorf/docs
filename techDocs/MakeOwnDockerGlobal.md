# Push Docker image to ghcr.io

Damit wir unseren Docker-Container auch von einer anderen Stelle holen können, müssen wir ihn irgendwo hochladen. Hierzu bietet sich `ghcr.io` an.
`ghcr.io` gehört zu github und benötigt somit keine weitere Anmeldung.

Was wir brauchen ist ein Access Token

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

Nun sollte auf eurem github Account unter packages das neue Package erscheinen.

![image](https://github.com/CO2-HTBLA-Kaindorf/docs/assets/16894982/37985b16-5e41-4a4c-91d2-878b7730fe77)
