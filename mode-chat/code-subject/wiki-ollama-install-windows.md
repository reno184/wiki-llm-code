# wiki-ollama-install-windows — installer Ollama sur un PC client Windows

Usage: procédure d'installation sur PC client (piste médecin/avocat — Ollama local chez le client). **Architecture de déploiement : le code de Renaud est protégé dans Docker, Ollama est externalisé sur la machine hôte (GPU)** — l'API Node tourne en conteneur, Ollama en natif. Mode chat Renaud, jamais chargé par un agent. Runtime et patterns → [wiki-ollama.md](wiki-ollama.md).

## Ce qu'il faut savoir avant de commencer

1. **Pas de WSL2 à installer soi-même** : l'installeur officiel Windows d'Ollama gère son propre environnement WSL2 intégré (mini-distro interne installée automatiquement). Rien à configurer à la main.
2. **Le vrai point d'attention = le GPU** :
   - **GPU NVIDIA** (cas le plus courant, le mieux supporté) : installer le **driver CUDA côté Windows** (pas dans WSL) — WSL2 y accède via la passerelle CUDA.
   - **GPU AMD/Intel** : support récent et partiel (Vulkan/Rocm expérimental) → à vérifier AVANT d'acheter/provisionner la machine client.
3. **Différence clé avec le Mac : pas de mémoire unifiée**. La limite n'est pas la RAM totale du PC mais la **VRAM dédiée du GPU** (ex. 8 Go) — souvent bien plus contraignant qu'un Mac M4 (11,84 Go). Un 9b quantifié peut ne pas tenir → adapter la taille du modèle à la VRAM (cf. [wiki-local-model.md](wiki-local-model.md)).
4. **`.wslconfig`** : si la RAM de WSL est limitée (fichier `.wslconfig` sur le PC), Ollama en hérite — une limite trop basse fait échouer le chargement des modèles.
5. **API inchangée** : `localhost:11434` fonctionne depuis Windows ET depuis WSL2 (réseau partagé) → les appels Node (API Express du client) ne changent pas.

## Piège Docker compose (le déploiement client utilise TOUJOURS Docker)

L'API Node (code protégé) tourne en conteneur ; Ollama en natif avec le GPU. Le piège concerne le **cas où Ollama lui-même est conteneurisé** (ex. stack entièrement dockerisée) :

- **GPU dans le compose** : sans le bloc ci-dessous, Ollama en conteneur tourne CPU = très lent.

```yaml
services:
  ollama:
    image: ollama/ollama:0.5.x        # pinner la version, pas :latest
    container_name: ollama_service
    ports:
      - "11434:11434"
    volumes:
      - ollama:/root/.ollama         # volume nommé : modèles persistants (jamais ~/.ollama_data sur Windows — le ~ se résout différemment selon le shell)
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
volumes:
  ollama:
```

- **Le prérequis qui fait échouer 90 % des cas** : le bloc compose ne suffit pas. Il faut le driver NVIDIA installé **côté Windows** (pas dans WSL) **et** le NVIDIA Container Toolkit configuré (`nvidia-ctk runtime configure --runtime=docker`), sinon erreur `could not select device driver nvidia`.
- **Test avant tout** : `docker run --rm --gpus all nvidia/cuda:12.5.0-base-ubuntu22.04 nvidia-smi`. `nvidia-smi` qui marche dans WSL ne prouve PAS que Docker a accès au GPU.
- **Piège port 11434** : install natif ET conteneur sur le même PC = conflit de port. Chez le client : un seul des deux (architecture cible = Ollama natif, donc pas de conteneur ollama).
- **GPU AMD/Intel** : le bloc `driver: nvidia` ne s'applique pas — support partiel, à valider avant provisionnement.

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
