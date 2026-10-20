# wiki-ollama-install-windows — installer Ollama sur un PC client Windows

Usage: procédure d'installation sur PC client (piste médecin/avocat — Ollama local chez le client). On ne sait jamais : à relire avant tout déploiement Windows. Mode chat Renaud, jamais chargé par un agent. Runtime et patterns → [wiki-ollama.md](wiki-ollama.md).

## Ce qu'il faut savoir avant de commencer

1. **Pas de WSL2 à installer soi-même** : l'installeur officiel Windows d'Ollama gère son propre environnement WSL2 intégré (mini-distro interne installée automatiquement). Rien à configurer à la main.
2. **Le vrai point d'attention = le GPU** :
   - **GPU NVIDIA** (cas le plus courant, le mieux supporté) : installer le **driver CUDA côté Windows** (pas dans WSL) — WSL2 y accède via la passerelle CUDA.
   - **GPU AMD/Intel** : support récent et partiel (Vulkan/Rocm expérimental) → à vérifier AVANT d'acheter/provisionner la machine client.
3. **Différence clé avec le Mac : pas de mémoire unifiée**. La limite n'est pas la RAM totale du PC mais la **VRAM dédiée du GPU** (ex. 8 Go) — souvent bien plus contraignant qu'un Mac M4 (11,84 Go). Un 9b quantifié peut ne pas tenir → adapter la taille du modèle à la VRAM (cf. [wiki-local-model.md](wiki-local-model.md)).
4. **`.wslconfig`** : si la RAM de WSL est limitée (fichier `.wslconfig` sur le PC), Ollama en hérite — une limite trop basse fait échouer le chargement des modèles.
5. **API inchangée** : `localhost:11434` fonctionne depuis Windows ET depuis WSL2 (réseau partagé) → les appels Node (API Express du client) ne changent pas.

## Checklist installation PC client

- [ ] Vérifier GPU client (NVIDIA ✅ / AMD-Intel : support partiel, à valider)
- [ ] Driver NVIDIA/CUDA à jour **côté Windows**
- [ ] Télécharger l'installeur officiel (ollama.com) et lancer — WSL2 intégré géré automatiquement
- [ ] Vérifier VRAM GPU → choisir la taille de modèle qui tient (référence : [wiki-local-model.md](wiki-local-model.md))
- [ ] `ollama --version` OK, `ollama pull <modele>` OK
- [ ] Test API : `curl http://localhost:11434/api/tags` répond
- [ ] Prévoir le lancement automatique au démarrage (Ollama démarre avec la session par défaut)

## État

**vivant** — procédure de déploiement client (non encore exécutée, à valider au premier déploiement réel).
