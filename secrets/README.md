# Secrets Management

Dieses Verzeichnis enthält sensitive Daten für die Helm-Releases. Die Secrets werden mit [SOPS](https://github.com/mozilla/sops) und [Age](https://github.com/FiloSottile/age) verschlüsselt.

## Struktur

- `*-secrets.yaml` - Unverschlüsselte Secret-Dateien (nicht ins Repository committen)
- `*-secrets.sops.yaml` - Verschlüsselte Secret-Dateien (mit SOPS werden diese ins Repository committed)

## Git Policy

Dieses Verzeichnis hat ein `.gitignore` Pattern:
```
secrets/*.yaml
!secrets/*.sops.yaml
```

Das bedeutet:
- ✅ Verschlüsselte SOPS-Dateien (`*.sops.yaml`) werden committed
- ❌ Unverschlüsselte Dateien (`*.yaml`) werden ignoriert

## Secrets verschlüsseln

Nachdem du die Secrets in `*-secrets.yaml` Dateien erstellt hast, verschlüssele sie mit SOPS:

```bash
# Einzelne Datei verschlüsseln
sops -i secrets/authentik-secrets.yaml
# Ergebnis: secrets/authentik-secrets.sops.yaml

# Oder mit helmfile
helm secrets enc secrets/authentik-secrets.yaml
```

## Secrets View

Um verschlüsselte Secrets anzusehen:

```bash
sops secrets/authentik-secrets.sops.yaml
```

## Helmfile mit Secrets

Das `helmfile.yaml` lädt sowohl `values/` als auch `secrets/` Verzeichnisse:

```yaml
values:
  - values/authentik.yaml
  - secrets/authentik-secrets.yaml  # oder authentik-secrets.sops.yaml
```

Helmfile wird automatisch entschlüsselt, wenn mit `helm-secrets` Plugin verwendet.
