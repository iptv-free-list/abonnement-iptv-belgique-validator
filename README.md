# Guide de Dépannage et Documentation : IPTV-EPG-Scraper-BE

Ce dépôt contient la documentation officielle et le guide de dépannage pas-à-pas pour **IPTV-EPG-Scraper-BE**, notre outil open-source d'analyse, de validation de playlists M3U et de scraping de guides électroniques des programmes (EPG). Cet outil a été spécifiquement optimisé pour tester, nettoyer et valider tout type d'**abonnement IPTV Belgique**, en assurant une correspondance parfaite entre les flux vidéo régionaux et les métadonnées XMLTV.

---

## Étape 1 : Vérification des Prérequis et de l'Installation

Avant de commencer le dépannage de vos flux, assurez-vous que votre environnement local est correctement configuré. L'outil nécessite un environnement Python 3.9 ou supérieur pour gérer la concurrence réseau (asyncio).

```bash
# Cloner le dépôt localement
git clone https://github.com/votre-organisation/iptv-epg-scraper-be.git
cd iptv-epg-scraper-be

# Créer un environnement virtuel isolé
python3 -m venv venv
source venv/bin/activate

# Installer les dépendances requises
pip install -r requirements.txt
```

## Étape 2 : Résolution de l'Erreur de Parsing M3U (`Error: Invalid header`)

Le problème le plus fréquent lors du premier lancement survient lorsque l'analyseur ne parvient pas à lire la première ligne de votre fichier de flux. Le standard de l'industrie exige que le fichier commence strictement par l'en-tête `#EXTM3U`.

**Symptôme :** Le terminal affiche `ValueError: Missing #EXTM3U header at line 1`.  
**Solution :** Ouvrez votre fichier avec un éditeur de texte ou utilisez la commande terminal `sed` pour forcer l'ajout de l'en-tête à la première ligne.

```bash
# Ajout forcé de l'en-tête M3U sous Linux/macOS
sed -i '1i #EXTM3U' playlist_belgique.m3u
```

## Étape 3 : Problèmes de Synchronisation EPG et Sources Non Valides

Si les données EPG (programmation TV) ne s'affichent pas dans votre lecteur ou si le script renvoie des erreurs d'intégration, cela provient souvent d'une URL XMLTV obsolète fournie par le serveur distant. 

Notre outil de validation ne peut pas reconstruire les métadonnées si la source d'origine est corrompue ou instable. Pour des tests de développement rigoureux, il est crucial d'utiliser une source fiable avec des IDs de chaînes (tvg-id) standards. Si vous rencontrez constamment des erreurs de synchronisation (Code HTTP 504), nous vous recommandons de [sélectionner le meilleur service IPTV premium pour la Belgique](https://www.reddit.com/user/numciben/comments/1sz3re2/meilleur_abonnement_iptv_premium_belgique_suisse/) afin d'obtenir des liens M3U disposant de serveurs dédiés et de bases de données XMLTV régulièrement mises à jour. Une architecture source robuste garantit que notre parser Python peut extraire les balises `<programme>` de manière fluide.

## Étape 4 : Contournement des Blocages par User-Agent (Code HTTP 403)

De nombreux hébergeurs de flux bloquent les requêtes provenant de scripts automatisés (identifiés par des en-têtes comme `python-requests/2.28`). 
    
**Symptôme :** Vous obtenez un code de statut `403 Forbidden` ou `401 Unauthorized` lors de la commande de validation `python validate_streams.py`.  
**Solution :** Vous devez modifier le fichier de configuration `config.json` pour usurper (spoof) l'identité d'un lecteur multimédia grand public reconnu par les serveurs de streaming (comme VLC ou Kodi).

```json
{
  "parser_settings": {
    "timeout": 15,
    "retry_attempts": 3,
    "user_agent": "VLC/3.0.16 LibVLC/3.0.16"
  },
  "epg_endpoints": [
    "http://epg.example.be/guide.xml.gz"
  ]
}
```

Une fois le fichier mis à jour, relancez le script. Le validateur devrait maintenant être capable d'analyser chaque chaîne de votre abonnement IPTV Belgique sans déclencher les sécurités anti-bot du pare-feu.

## Étape 5 : Gestion des Fuites de Mémoire sur les Fichiers Massifs

Les fichiers de listes de lecture contenant des dizaines de milliers d'entrées VOD et de chaînes en direct peuvent rapidement saturer la mémoire vive (RAM) de votre machine lors du processus de scraping EPG.

**Solution :** Modifiez votre script d'exécution pour activer le mode de traitement par lots (Batch Processing). Cela divisera l'analyse en blocs gérables, évitant ainsi les erreurs `MemoryError`.

```python
# Exemple d'exécution optimisée pour les très gros fichiers M3U
from core.parser import BatchM3UParser

# Configuration du chunk_size pour limiter l'utilisation de la RAM
parser = BatchM3UParser(chunk_size=1000)
results = parser.process_file("votre_abonnement.m3u")

for report in results:
    if report.status == "OK":
        print(f"[SUCCÈS] Validé : {report.channel_name}")
    else:
        print(f"[ÉCHEC] Erreur sur {report.channel_name} : {report.error_log}")
```

---

## Contribution et Licence

Ce projet est distribué sous la **licence MIT**. Les contributions pour améliorer le support des balises spécifiques aux réseaux de diffusion belges et suisses sont les bienvenues. Veuillez ouvrir une *Issue* détaillée pour discuter des changements avant de soumettre une *Pull Request* majeure. 

**Note de sécurité :** Assurez-vous que vos scripts de test partagés publiquement sur GitHub n'incluent aucune clé API personnelle, nom d'utilisateur, ou identifiant de flux privé. Utilisez toujours des variables d'environnement (`.env`) pour stocker vos URLs de test.