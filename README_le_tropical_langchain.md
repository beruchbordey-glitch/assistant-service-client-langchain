# Assistant IA - Le Tropical (LangChain)

Étude de cas : construire, couche par couche, un assistant de service client pour une marque de cosmétiques, d'un simple chatbot jusqu'à un agent autonome capable de choisir ses actions.

## Contexte

Le Tropical, marque de cosmétiques congolaise en ligne, veut un assistant qui réponde aux clients 24/7 sur les produits, les commandes et les livraisons, qui sache passer la main à un humain sur les sujets sensibles, et qui ne raconte jamais n'importe quoi sur ses produits.

## Ce que fait l'assistant

- Répond aux clients en gardant le fil de la conversation (mémoire).
- Conseille sur les vrais produits du catalogue, avec noms, prix et usages exacts (il ne les invente pas).
- Crée des tickets de réclamation, vérifie le statut d'une commande et la disponibilité d'un produit.
- Choisit lui-même l'action adaptée à chaque demande : conseiller, chercher, créer un ticket, ou simplement répondre.
- Passe la main à un conseiller humain sur les cas sensibles (effet indésirable, litige, produit incertain), avec un délai et un canal de contact.

## Construit en trois couches

1- Chatbot (mémoire). Un modèle de langage est sans état : il oublie tout d'un appel à l'autre. La mémoire consiste simplement à lui renvoyer l'historique complet de la conversation (une liste de messages qui grandit à chaque tour), plus un message système qui fixe son rôle, son ton et ses règles d'escalade.

2- RAG (connaissance des produits). Pour éviter que l'assistant invente, on lui donne accès au catalogue. Chaque produit est transformé en vecteur (embedding) qui capture son sens, puis rangé dans un index. À chaque question, on récupère les produits les plus proches par le sens et on les injecte dans le prompt : le modèle répond en citant des fiches réelles, pas de mémoire. Modèle d'embeddings multilingue (catalogue en français), exécuté en local et gratuit.

3- Agent (autonomie). La recherche produit et les autres actions sont transformées en outils. Le modèle reçoit plusieurs outils et décide seul lequel appeler selon la demande  c'est la différence entre un agent (qui raisonne sur la marche à suivre) et une chaîne figée (qui exécute toujours la même séquence). Le modèle ne fait jamais l'action lui-même : il la demande, et le code l'exécute.

## Compétences mises en œuvre

Modèles de chat (langchain-anthropic), gestion de l'historique de conversation, messages système / rôle, embeddings et recherche sémantique (langchain-huggingface, InMemoryVectorStore), RAG, définition d'outils (@tool), tool calling, routage multi-outils, boucle d'agent multi-étapes, conception d'escalade vers un humain.

## Résultat observé

- Recherche sémantique : la requête << cheveux très secs et abîmés >> (sans aucun nom de produit) remonte bien les soins capillaires nourrissants, et écarte le reste du catalogue.
- Autonomie : sur trois messages différents, l'agent appelle la recherche produit pour un conseil, la création de ticket pour une réclamation complète, et n'appelle aucun outil pour un simple bonjour.
- Fin des hallucinations : après ajout du RAG, l'assistant cite les vrais produits (noms, prix, ingrédients) au lieu d'en inventer.

## Limites et suites

- Le catalogue doit être tenu à jour ; en production, il viendrait de vrais fichiers (PDF, Excel) via un document loader.
- Les actions sont simulées (le ticket et les vérifications renvoient une confirmation) : le mécanisme est prêt, il reste à brancher les vrais systèmes (base de données, API d'envoi de mail).
- Pour toute action irréversible (mail réel, remboursement), prévoir une validation humaine.
- Agent unique, suffisant pour ce périmètre ; une organisation multi-agents ne se justifierait que pour des tâches nettement plus complexes.

