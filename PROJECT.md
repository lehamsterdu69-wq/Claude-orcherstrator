# Claude Orchestrator

> Un système multi-agents local-first, modulaire, capable de se spécialiser dynamiquement pour presque n'importe quelle tâche réalisable avec les ressources d'un PC.

---

## 1. Vision

Orchestrator est un système multi-agents généraliste permettant à plusieurs IA de collaborer pour réaliser des tâches complexes.

Le système n'est **pas** spécialisé dans le développement logiciel. Le développement logiciel est une spécialisation possible parmi d'autres.

Selon la mission donnée par l'utilisateur, Orchestrator construit dynamiquement une équipe adaptée :

```
"Construis une application web"  → Architect, Backend Dev, Frontend Dev, Tester, Doc
"Analyse ces données"            → Data Analyst, Statistician, Python Analyst, Researcher
"Fais une recherche approfondie" → Researcher, Fact Checker, Analyst, Synthesizer, Writer
"Crée un jeu vidéo"              → Game Designer, Programmer, Tech Artist, Tester
```

La seule vraie limite : ce que la machine peut faire avec les modèles disponibles, les CLIs installées, Python, les fichiers accessibles, les APIs, Internet, les skills disponibles.

---

## 2. Philosophie

```
USER → UNDERSTAND → CLARIFY → ANALYZE → SPECIALIZE → DISCOVER SKILLS → PLAN
     → DISTRIBUTE → COLLABORATE → EXECUTE → TEST/VERIFY → AUDIT
     → PATCH/IMPROVE → DOCUMENT → UPDATE MEMORY → FINAL REVIEW → DONE
```

Ce n'est pas une collection de chatbots qui s'envoient des prompts. Chaque agent a : un rôle, des capacités, un contexte, des permissions, des tâches, des responsabilités, des canaux de communication, un accès contrôlé aux ressources.

---

## 3. Architecture générale

```
USER → WEB DASHBOARD (:6767) → ORCHESTRATOR (planning, supervision, spécialisation)
         │
         ├── TASK ENGINE
         ├── SKILL SYSTEM (find-a-skill)
         └── MEMORY (Obsidian)
         │
         ▼
   GEMINI API (2.5 Flash) → distribution des tâches/skills, assistance
         │
         ▼
   AGENTS PARALLÈLES (Kimi, GLM, Qwen...) → INTER-AGENT BUS
         │
         ▼
   TESTING → AUDIT → PATCH → MEMORY UPDATE → OBSIDIAN → FINAL REVIEW
```

---

## 4. Agents : architecture modulaire

Un rôle n'est **jamais** lié définitivement à un modèle.

```yaml
roles:
  orchestrator:
    agent: sonnet   # peut devenir agent: codex sans toucher au moteur
```

Même principe pour : coder, researcher, analyst, architect, reviewer, tester, helper, documentation, security, data analyst, etc. Les rôles peuvent aussi être créés dynamiquement par l'orchestrateur selon la mission.

---

## 5. Agent Registry

```yaml
agents:
  sonnet:
    adapter: claude_code
    model: sonnet-5
    enabled: true
    capabilities: [reasoning, planning, orchestration, coding]

  kimi:
    adapter: freebuff
    model: kimi-2.7-code
    enabled: true
    capabilities: [complex_coding, architecture, reasoning]

  glm:
    adapter: codex
    model: glm-5.2
    enabled: true
    capabilities: [coding, auditing, performance]

  qwen_1:
    adapter: qwen_cli
    model: qwen-27b
    enabled: true
    capabilities: [coding]
```

Le registre doit rester extensible.

---

## 6. Ajouter / supprimer un agent

Ajouter :
```
1. créer ou sélectionner un Adapter
2. déclarer le modèle
3. déclarer ses capacités
4. activer l'agent
```
Le moteur central n'est jamais modifié.

Supprimer :
```yaml
agents:
  qwen_2:
    enabled: false
```
Orchestrator recalcule automatiquement la répartition des tâches. Un agent indisponible ne doit jamais bloquer le système.

---

## 7. Adapter System

```
AgentManager
├── ClaudeCodeAdapter
├── FreebuffAdapter
├── QwenCLIAdapter
├── CodexAdapter
├── AiderAdapter
└── FutureAdapter
```

Le reste de l'application ne parle qu'à l'interface abstraite de l'Adapter.

---

## 8. Configuration initiale

| Fonction | Modèle | Interface |
|---|---|---|
| Orchestrator | Sonnet 5 | Claude Code CLI |
| Heavy Coding | Kimi 2.7 Code | Freebuff CLI |
| Coding / Audit | GLM-5.2 | Codex CLI |
| Coding | Qwen #1 | Qwen CLI |
| Coding | Qwen #2 | Qwen CLI |
| Helper ×3 | Llama | Aider |
| Documentation | Llama #4 | Aider |
| Dispatcher | Gemini 2.5 Flash | API (gratuite) |

### Contraintes matérielles

Ressources disponibles : **8 Go VRAM + 30 Go RAM**, via Ollama (superposition VRAM/RAM).

Cette configuration à 10 agents est un **plafond théorique**, pas un point de départ réaliste : plusieurs modèles locaux lourds (27B, etc.) tournant en simultané sur 8 Go de VRAM vont largement déborder sur la RAM/CPU, avec un impact direct sur la latence. Le nombre d'agents locaux actifs en simultané doit être dimensionné en fonction de la VRAM réellement libre au runtime, pas fixé statiquement dans la config. L'orchestrateur doit pouvoir réduire dynamiquement le nombre d'agents actifs si la mémoire devient critique.

L'API Gemini gratuite reste soumise à des quotas (requêtes/minute et par jour) côté Google — l'orchestrateur doit prévoir un fallback ou une file d'attente si le quota est atteint, plutôt que de considérer Gemini comme une ressource illimitée.

---

## 9. Orchestrator

Par défaut : Sonnet 5 via Claude Code CLI. Il ne distribue pas une simple checklist : il comprend le problème global.

---

## 10. Clarification

```
USER → WEB UI → ORCHESTRATOR
```
Si une info importante manque : ORCHESTRATOR → QUESTION → WEB UI → USER → ANSWER → ORCHESTRATOR. Les questions apparaissent directement dans l'interface ; l'utilisateur répond sans interrompre le système.

---

## 11. Project / Task Analysis

Orchestrator détermine : objectif, contraintes, ressources, environnement, complexité, dépendances, risques, résultats attendus, outils/agents disponibles, skills nécessaires. Pour une tâche logicielle : repository, architecture, dépendances, tests, documentation, Git — uniquement si pertinent.

---

## 12. Dynamic Specialization

```
Mission simple        → 1 agent
Mission moyenne        → 2-3 agents
Mission complexe       → 4-6 agents
Mission très complexe  → équipe complète
```
Les rôles sont déterminés dynamiquement, jamais systématiquement au complet.

---

## 13. Master Context & Master Checklist

Le Master Context contient : objectif, état actuel, requirements, contraintes, architecture, ressources, agents disponibles, capacités, skills pertinents, problèmes connus, dépendances, résultats attendus, exigences qualité.

Chaque tâche de la Master Checklist a au minimum : ID, titre, description, contexte global/local, rôle, agent, skills, dépendances, chemins autorisés/interdits, priorité, requirements, contraintes, critères d'acceptation, exigences de test, output attendu, statut.

---

## 14. Gemini 2.5 Flash

Gemini n'est **pas un agent** — uniquement un service API, sans terminal, sans session persistante, sans accès direct au workspace ou aux agents.

Orchestrator lui envoie : contexte projet + état actuel + architecture + tâches + capacités des agents + skills disponibles + dépendances + contraintes + résultats attendus. Gemini peut alors distribuer les tâches, déterminer les rôles précis, distribuer les skills, identifier les dépendances, aider un agent, analyser un problème, faire une petite recherche, produire des données structurées. Réponses en JSON strict quand elles sont consommées automatiquement.

---

## 15. Skills

Un skill = une capacité spécialisée (python, react, docker, database, security, statistics, research, mathematics, 3d, video, audio, image-processing, git, performance-analysis, documentation...). Pas limité à la programmation.

**Discovery** : le skill `find-a-skill` est utilisé quand l'orchestrateur identifie une capacité manquante.

**Distribution** : les skills découverts sont envoyés à Gemini avec le contexte projet, qui décide de l'assignation (ou décide qu'un skill n'est pas nécessaire). Jamais imposés artificiellement.

---

## 16. Mémoire partagée — Obsidian

```
OBSIDIAN ← Sonnet / Kimi / Qwen → SHARED MEMORY
```

Structure du vault :
```
ORCHESTRATOR_MEMORY/
├── Projects/  ├── Tasks/    ├── Agents/   ├── Skills/
├── Knowledge/ ├── Decisions/├── Research/ ├── Solutions/
├── Errors/    └── Context/
```
Configurable.

Avant une tâche importante : chercher dans la mémoire → lire → comprendre → travailler. Après une tâche significative : identifier le savoir à retenir → mettre à jour Obsidian (découvertes, solutions, décisions, erreurs importantes, connaissances réutilisables, contexte durable). Pas de spam de notes inutiles.

Chaque entrée doit être traçable : auteur, agent, timestamp, projet, tâche, source, confiance.

---

## 17. Communication inter-agents

Deux canaux : chat privé (Agent A ↔ Agent B) et chat de groupe. L'utilisateur et l'orchestrateur peuvent lire, écrire, intervenir, poser des questions dans toutes les conversations.

Style de communication des agents : amical, naturel, collaboratif, professionnel, utile — pas de bavardage artificiel.

---

## 18. Workspace & isolation

L'utilisateur définit le workspace depuis le Dashboard (ex. `/projects/mon-projet`), qui devient `WORKSPACE_ROOT`.

**Isolation : conteneur, pas de blocage applicatif.** L'ensemble du système (orchestrateur + agents + CLIs) tourne dans un conteneur qui n'a accès qu'à un disque dédié monté en volume, contenant uniquement les projets. Il n'y a pas de path validator ni de command interceptor applicatif à maintenir en Python : l'isolation est garantie au niveau du conteneur/volume, pas par une couche de sécurité logicielle en plus à l'intérieur du système.

Points à garder en tête pour que cette isolation soit réelle et pas juste sur le papier :
- le conteneur ne doit monter **que** le disque projets, en lecture/écriture limitée à ce volume — pas le reste du filesystem hôte ;
- les accès réseau du conteneur doivent être limités aux domaines/API strictement nécessaires (Gemini API, registries de paquets si besoin) ;
- les agents CLI qui font du `subprocess`/réseau doivent rester **dans** le conteneur — aucun adapter ne doit avoir de chemin de sortie vers l'hôte ;
- les secrets/API keys sont injectés au conteneur (env vars / secrets manager), jamais commités dans le disque projets.

---

## 19. Terminal Management

Chaque agent CLI réel a son propre terminal/processus :
```
Terminal #1 → Claude Code   Terminal #2 → Freebuff   Terminal #3/4 → Qwen CLI
Terminal #5 → Codex CLI     Terminal #6-9 → Aider
```
Association claire maintenue : Agent ↔ Process ↔ Terminal ↔ Task.

**Live Terminal View** (Dashboard) : affichage temps réel du terminal de chaque agent (scroll, recherche, copie, historique, erreurs, commandes, état du process). Quand les permissions le permettent, l'utilisateur et l'orchestrateur peuvent envoyer des commandes à un agent — toute commande reste contenue par l'isolation du conteneur (section 18).

---

## 20. Dashboard (localhost:6767)

Vues principales : Progress global, Agents (statut/tâche/%), Conversations (par agent + groupes), Tasks (ID, titre, agent, rôle, statut, progression, priorité, dépendances, dates, fichiers, tests, résultat), Task History (persistant), Logs (événements, erreurs, commandes, appels Gemini, modifs mémoire, audits), Workspace (sélection/état), Agent Configuration (activation/désactivation).

---

## 21. Progress, dépendances, ownership

```
PENDING → ASSIGNED → ANALYZING → RUNNING → TESTING → REVIEW → DONE
États additionnels : BLOCKED, FAILED, PAUSED, CANCELLED
```

Dépendances : une tâche ne démarre pas avant que ses dépendances soient satisfaites ; si bloqué → communication → résolution → resume.

File Ownership : une tâche déclare owner + chemins autorisés/interdits, pour éviter que deux agents écrivent au même endroit sans coordination.

---

## 22. Testing & Audit

Chaque agent valide son travail selon une méthode adaptée à la mission (pas uniquement pensée pour le code) :
- Software : format → lint → unit test → integration test → build
- Recherche : sources, cohérence, contradictions, confiance
- Données : statistiques, outliers, validation, cohérence

Audit adapté à la mission (perf/sécu/archi pour du code, fact-checking pour de la recherche, etc.).

---

## 23. Final Review

L'orchestrateur vérifie exigences, résultats, qualité, cohérence, erreurs, documentation, mémoire, état des tâches. FAIL → nouvelle tâche → agent. PASS → complete.

---

## 24. Python Core

Tout le projet est en Python, **sauf le frontend du dashboard**. Ça inclut : Orchestrator, Agent Manager, Adapters, Task Engine, Scheduler, Communication Bus, Memory System, Obsidian Integration, Skill System, Gemini API, Workspace/Volume config, Terminal Manager, Monitoring, Logging, Persistence, Configuration, Backend API.

Backend : HTTP API, WebSocket/realtime, agent management, terminal streaming, task management, messaging, memory, skills, persistence, monitoring.

**Persistence** : projects, agents, roles, tasks, task history, messages, conversations, skills, memory references, logs, audits, résultats. Un redémarrage du serveur ne détruit pas l'état.

---

## 25. Design Rules

1. Général — pas limité au software engineering
2. Spécialisation dynamique — l'équipe est construite selon la mission
3. Model agnostic — un rôle ne dépend jamais d'un modèle précis
4. Agents modulaires — ajouter/supprimer un agent ne touche pas au moteur
5. CLI ownership — chaque agent réel est piloté depuis le terminal de son outil
6. Gemini = API uniquement
7. Contexte complet fourni avant distribution
8. Skills trouvés dynamiquement, distribués ou ignorés
9. Mémoire partagée lue/mise à jour par les agents
10. Communication privée + groupe entre agents
11. Accès humain complet à toutes les conversations
12. **Isolation par conteneur** — aucune opération ne peut sortir du volume projets monté ; pas de couche de blocage applicative à maintenir en plus
13. Persistance des tâches, conversations, événements, résultats
14. Observabilité — terminaux et communications visibles par l'utilisateur
15. Python core, sauf frontend dashboard
16. Vérification — les résultats sont vérifiés avant d'être considérés terminés

---

## 26. Vision finale

```
Understanding → Planning → Specialization → Skill discovery → Agent selection
→ Parallel execution → Communication → Verification → Memory → Final review
```

Orchestrator n'est pas un coding swarm. C'est une plateforme locale permettant à une intelligence orchestratrice de construire et superviser dynamiquement une équipe d'agents spécialisés, isolée dans un conteneur avec accès limité au disque projets, avec une mémoire collective et une observabilité complète pour l'utilisateur.
