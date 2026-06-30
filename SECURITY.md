# 🔒 Security Scanning Guide

Scan de sécurité automatisé avec Trivy pour détecter les vulnérabilités et générer les SBOM.

## Installation de Trivy

### macOS

```bash
brew install aquasecurity/trivy/trivy
```

### Linux

```bash
# Debian/Ubuntu
apt-get install trivy

# RedHat/CentOS
yum install trivy
```

### Docker

```bash
docker run aquasec/trivy image --help
```

## Utilisation Locale

### 1. Scanner rapide (ligne de commande)

```bash
# Vulnérabilités CRITICAL et HIGH uniquement
trivy image tasklist-frontend:latest --severity CRITICAL,HIGH

# Rapport formaté en table
trivy image tasklist-frontend:latest --severity CRITICAL,HIGH --format table
```

### 2. Script automatisé

```bash
# Lancer le scan complet
./scripts/security-scan.sh

# Génère les fichiers dans security-reports/
```

**Fichiers générés:**

- `vulns-TIMESTAMP.txt` - Rapport texte des vulnérabilités
- `trivy-report-TIMESTAMP.json` - Rapport JSON détaillé
- `sbom-spdx-TIMESTAMP.json` - Bill of Materials (SPDX)
- `sbom-cyclonedx-TIMESTAMP.json` - Bill of Materials (CycloneDX)

### 3. Commandes individuelles

```bash
# Scan vulnérabilités (table)
trivy image tasklist-frontend:latest \
  --severity CRITICAL,HIGH \
  --format table \
  --output vulns.txt

# Scan complet (JSON)
trivy image tasklist-frontend:latest \
  --format json \
  --output trivy-report.json

# SBOM SPDX
trivy image tasklist-frontend:latest \
  --format spdx-json \
  --output sbom-spdx.json

# SBOM CycloneDX
trivy image tasklist-frontend:latest \
  --format cyclonedx \
  --output sbom-cyclonedx.json
```

## Formats de rapport

### Trivy Report (JSON)

```json
{
  "SchemaVersion": 2,
  "ArtifactName": "tasklist-frontend:latest",
  "Results": [
    {
      "Target": "node:20-alpine",
      "Class": "os-pkgs",
      "Type": "alpine",
      "Vulnerabilities": [...]
    }
  ]
}
```

### SBOM SPDX

Format standardisé SPDX pour la facturation logicielle. Utilisable avec:

- outils de conformité
- audits de dépendances
- rapports réglementaires

### SBOM CycloneDX

Format OWASP CycloneDX pour:

- Dépôts de composants
- Analyse de risques
- Intégration avec outils de sécurité

## CI/CD Integration

### GitHub Actions

Le workflow `.github/workflows/security-scan.yml` :

- Scan automatique à chaque push
- Upload résultats en artefacts
- Commente les PR avec résumé
- Crée des alertes de sécurité

Visible dans l'onglet "Security" du repo.

## Interprétation des résultats

### Sévérités Trivy

- 🔴 **CRITICAL** - Exploit connu, corriger immédiatement
- 🟠 **HIGH** - Faille grave, corriger rapidement
- 🟡 **MEDIUM** - Corriger dans la prochaine version
- 🔵 **LOW** - Faible impact
- ⚪ **UNKNOWN** - Sévérité inconnue

### Actions recommandées

1. **Vulnérabilité CRITICAL trouvée**
   - Notifier immédiatement
   - Évaluer l'impact
   - Déployer un patch

2. **Vulnérabilité HIGH trouvée**
   - Planifier une correction
   - Vérifier les alternatives
   - Mettre à jour les dépendances

3. **Pas de vulnérabilités critiques**
   - Continuer le déploiement
   - Archiver les SBOM pour audit

## Gestion des faux positifs

Si une vulnérabilité est un faux positif ou non applicable :

```bash
# Créer un .trivyignore
cat > .trivyignore << EOF
# Format: CVE-ID
CVE-2021-12345
CVE-2021-54321
EOF
```

Puis utiliser:

```bash
trivy image --ignorefile .trivyignore tasklist-frontend:latest
```

## Intégration SBOM

### Upload vers dépôt CycloneDX

```bash
# Exemple: envoyer vers un dépôt de composants
curl -X POST https://sbom-repo.local/api/upload \
  -F "file=@sbom-cyclonedx.json"
```

### Validation SBOM

```bash
# Vérifier le format
trivy sbom sbom-spdx.json --format json --output validation.json
```

## Métriques et Monitoring

### Tendances de vulnérabilités

- Archiver les rapports JSON hebdomadaires
- Créer un dashboard avec l'historique
- Alerter si augmentation de vulnérabilités

### Compliance

- SPDX/CycloneDX pour audits
- SonarQube pour codebase
- Trivy pour dépendances runtime

## Ressources

- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [SBOM SPDX Spec](https://spdx.github.io/)
- [CycloneDX Spec](https://cyclonedx.org/)
- [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/)
