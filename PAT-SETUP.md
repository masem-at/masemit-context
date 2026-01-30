# Personal Access Token (PAT) Setup

## 1. Alten Token löschen (optional)

GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
→ Alten Token finden und löschen

---

## 2. Neuen Token erstellen

1. **GitHub → Profilbild (rechts oben) → Settings**
2. **Ganz unten links: Developer settings**
3. **Personal access tokens → Tokens (classic)**
4. **Generate new token (classic)**

### Token-Einstellungen:

| Feld | Wert |
|------|------|
| Note | `masemit-context-sync` (oder was du willst) |
| Expiration | 90 days / 1 year / No expiration |
| Scopes | ✅ `repo` (Full control of private repositories) |

5. **Generate token** klicken
6. **TOKEN SOFORT KOPIEREN** – du siehst ihn nur einmal!

---

## 3. Token als Secret in Source-Repos hinzufügen

Für jedes Repo das synchen soll (ChainSights, TellingCube, etc.):

1. Repo → **Settings** → **Secrets and variables** → **Actions**
2. **New repository secret**
3. Name: `CONTEXT_REPO_PAT`
4. Value: Token einfügen
5. **Add secret**

---

## 4. Token sicher aufbewahren

Speicher ihn in deinem Passwort-Manager (1Password, Bitwarden, etc.) unter:
- Name: `GitHub PAT - masemit-context-sync`
- Token: `ghp_xxxxxxxxxxxx`

---

## Betroffene Repos

Diese Repos brauchen das Secret `CONTEXT_REPO_PAT`:

- [ ] `masem-at/chainsights`
- [ ] `masem-at/tellingcube` (wenn Sync aktiviert)
- [ ] weitere...
