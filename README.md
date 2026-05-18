# Fabric Workspace Catalog

Sentralisert GitHub Pages-katalog for alle Power BI / Fabric workspaces i organisasjonen.

## Arkitektur

```
org/
├── fabric-catalog/          ← Denne repo (katalog + GitHub Pages)
│   ├── docs/index.html      ← Nettsiden (statisk, ingen backend)
│   ├── repos.txt            ← Liste over alle workspace-repos
│   └── .github/workflows/
│       └── deploy-catalog.yml
│
├── ws-salg/                 ← Workspace-repo (ett per workspace)
│   ├── workspace.json       ← Metadata (leses av katalogen)
│   ├── salg-dashboard.pbix  ← Power BI Desktop-fil
│   ├── model/               ← TMDL (valgfritt, for kode-diff)
│   ├── dataflows/           ← JSON-eksport av dataflows
│   └── .github/workflows/
│       └── deploy-fabric.yml
│
├── ws-finans/
├── ws-hr/
└── ... (20+ workspaces)
```

## Branch-strategi

| Branch | Miljø | Fabric-workspace |
|--------|-------|-----------------|
| `main` | Produksjon | `ws-NAVN [Prod]` |
| `dev`  | Utvikling  | `ws-NAVN [Dev]`  |

Pull request fra `dev` → `main` trigger deploy til prod.

## Komme i gang — nytt workspace

```bash
# 1. Kopier template
gh repo create org/ws-NAVN --template org/workspace-template --private

# 2. Fyll ut metadata
nano workspace.json

# 3. Legg til i katalogen
echo "ws-NAVN" >> repos.txt
git commit -am "add ws-NAVN" && git push

# 4. Sett opp secrets (én gang per org med GitHub Environments)
gh secret set FABRIC_CLIENT_ID    -r org/ws-NAVN
gh secret set FABRIC_CLIENT_SECRET -r org/ws-NAVN
gh secret set FABRIC_TENANT_ID    -r org/ws-NAVN
gh secret set CATALOG_TOKEN       -r org/ws-NAVN
```

## For Power BI-utviklere

```bash
# Klon workspace-repo
git clone https://github.com/org/ws-NAVN
cd ws-NAVN
git checkout dev           # Jobb alltid i dev

# Åpne i Power BI Desktop
start salg-dashboard.pbix  # Windows
open  salg-dashboard.pbix  # Mac

# Commit og push
git add salg-dashboard.pbix
git commit -m "oppdater salgsrapport: legg til regional filter"
git push origin dev

# Lag PR til main når klar for prod
gh pr create --base main --title "Release: regional filter i salgsrapport"
```

## Secrets som trengs (per org)

| Secret | Beskrivelse |
|--------|-------------|
| `FABRIC_CLIENT_ID` | Service Principal App ID |
| `FABRIC_CLIENT_SECRET` | Service Principal Secret |
| `FABRIC_TENANT_ID` | Azure AD Tenant ID |
| `CATALOG_TOKEN` | GitHub PAT for å trigge katalog-oppdatering |

## GitHub Pages

Katalogen hostes på `https://ORG.github.io/fabric-catalog/` og oppdateres automatisk ved:
- Push til `main` i katalog-repo
- Deploy fra et workspace-repo (`repository_dispatch`)

## Tilgangsstyring

- **Workspace-repo**: `CODEOWNERS` peker på ansvarlig team
- **Branch protection**: `main` krever 1 review + status checks
- **Fabric**: Service Principal deployer — ikke personlige kontoer
- **Katalogen**: Synlig for alle med GitHub-tilgang til org
