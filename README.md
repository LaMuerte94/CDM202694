# 🏆 CDM 2026 — Pronostics entre amis

## 🌐 Accès à l'application

👉 **[https://darkslategrey-trout-744243.hostingersite.com/](https://darkslategrey-trout-744243.hostingersite.com/)**

---

Application de pronostics pour la Coupe du Monde 2026, avec connexion par nom + mot de passe (créé par l'admin) et sync temps réel via **Supabase**.

## ✨ Fonctionnalités

- 12 joueurs prédéfinis (modifiables côté admin)
- Chaque joueur se connecte avec **son nom + son mot de passe** (créé par l'admin)
- L'admin crée / modifie / supprime les joueurs et leurs mots de passe depuis l'onglet ⚙️ Admin
- Code admin séparé pour saisir les résultats officiels
- Synchronisation live entre tous les amis (polling toutes les 10 s)
- Fallback `localStorage` si Supabase indisponible
- 72 matches de groupes + 32 matches de phases finales (16es → finale)
- Classement automatique : 5 pts score exact / 3 pts bon écart / 1 pt bon résultat / 0 sinon

## 🚀 Mise en place Supabase (1 fois)

Ouvrir le **SQL Editor** sur https://supabase.com/dashboard et exécuter :

```sql
create table if not exists public.cdm_state (
  id int primary key default 1,
  data jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now(),
  constraint cdm_state_singleton check (id = 1)
);

insert into public.cdm_state (id, data)
values (1, '{}'::jsonb)
on conflict (id) do nothing;

-- Accès anonyme (projet entre amis, pas d'auth)
alter table public.cdm_state enable row level security;

drop policy if exists "cdm read"   on public.cdm_state;
drop policy if exists "cdm insert" on public.cdm_state;
drop policy if exists "cdm update" on public.cdm_state;

create policy "cdm read"   on public.cdm_state for select using (true);
create policy "cdm insert" on public.cdm_state for insert with check (true);
create policy "cdm update" on public.cdm_state for update using (true) with check (true);
```

> ⚠️ La clé `sb_publishable_...` est volontairement publique (clé anon). Les écritures sont autorisées car c'est un projet entre amis. Si tu veux limiter le vandalisme, ajoute des règles RLS plus strictes ou un proxy edge function.

## 🖥️ Lancement local

Ouvre simplement `index.html` dans un navigateur, ou sers le dossier :

```bash
npx serve .
```

## 🌍 Synchronisation automatique des résultats FIFA

L'application récupère automatiquement les scores officiels depuis **[TheSportsDB](https://www.thesportsdb.com/league/4429-fifa-world-cup)** — gratuit, **sans inscription ni clé API**.

### Utilisation

L'admin a un bouton **🔄 Synchroniser les résultats officiels** dans l'onglet ⚙️ Admin. Un clic et tous les matchs terminés depuis le début du tournoi sont importés automatiquement (poules + phases finales). Les paris des joueurs restent intacts, seul le score officiel est mis à jour.

### Détails techniques

- Endpoint : `https://www.thesportsdb.com/api/v1/json/3/eventsseason.php?id=4429&s=2026`
- Aucune authentification requise (clé publique `3`)
- Matching des équipes FR↔EN via une table interne (`TEAM_FR_TO_EN`)
- Si une équipe n'est pas reconnue, elle apparaît dans la console du navigateur (touche **F12**)

## 🔐 Code admin par défaut

`admin2026` — à changer dans l'onglet **⚙️ Admin** dès la première connexion.

## 📁 Structure

- `index.html` — markup + logique JS de l'application
- `style.css` — feuille de style (extraite pour faciliter la maintenance)
- `data.json` — état initial (utilisé uniquement comme template)

## 🔄 Migration depuis l'ancienne version GitHub

Tous les paris/résultats existants peuvent être réimportés via l'onglet **⚙️ Admin → 📤 Importer** en uploadant l'ancien `data.json`.
