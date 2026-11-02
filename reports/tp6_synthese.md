# Orchestrateur ou integration continue ? (TP06)

| Critere | Prefect (orchestrateur) | GitHub Actions (CI) |
|---|---|---|
| Ce qui declenche l'execution | Une planification cron (chaque lundi a 6 h), un Quick run dans l'interface, ou un lancement manuel | Chaque `git push` sur le depot (et le bouton Run workflow) |
| Ou s'execute le traitement | Sur notre poste, dans le processus `serve` ou le worker du work pool `pool-tp` | Sur une machine Linux ephemere fournie par GitHub (`ubuntu-latest`) |
| Qui surveille, et ou | Le serveur Prefect, interface sur http://127.0.0.1:4200 : etat, duree et logs de chaque tache | GitHub, onglet Actions : statut et logs de chaque etape, commit marque vert ou rouge |
| Ce qui se passe en cas d'echec | La tache `rejouer_pipeline` est relancee 2 fois ; si `valider` echoue, le flow passe en Failed, la tache fautive est designee et `notifier` n'est pas executee | L'etape renvoie le code 1, le job et le commit passent au rouge ; les rapports sont conserves grace a `if: always()` |
| Type de tache adapte | Traitements recurrents sur les donnees : reentrainement hebdomadaire, pipelines longs, executions sans intervention | Verification de chaque modification du code avant integration : tests, regle de qualite |
| Ce qui est versionne dans Git | `flow_mlops.py`, `planifier.py` (et `prefect.yaml` s'il est cree) ; l'historique des runs reste dans la base du serveur Prefect | `.github/workflows/ci.yml`, `verifier_qualite.py`, `requirements-ci.txt` ; l'historique des runs est conserve par GitHub |

## Conclusion (5 lignes)

GitHub Actions valide le code a chaque modification : un push qui degrade le modele (F1 = 0.25) est immediatement marque en rouge.
Prefect execute et surveille un traitement recurrent : le reentrainement hebdomadaire tourne sans intervention et s'arrete seul sous le seuil.
La meme regle de qualite (seuil_f1 = 0.82) est appliquee des deux cotes, a deux moments differents.
Le test avec max_depth: 6 (F1 = 0.8227, accepte) montre que la protection depend entierement du choix du seuil.
Les deux outils sont complementaires : la CI protege le code, l'orchestrateur protege les executions en production.