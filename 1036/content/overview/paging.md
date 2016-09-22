
# Pagination de données Microsoft Graph dans votre application 
 
Lorsque les demandes Microsoft Graph renvoient trop d’informations à afficher sur une page, vous pouvez utiliser la pagination pour répartir les informations en parties gérables. 

Vous pouvez consulter les pages vers l’avant et l’arrière dans les réponses Microsoft Graph. Une réponse qui contient des résultats paginés inclut un jeton de saut (**odata.nextLink**) qui vous permet d’accéder à la page de résultats suivante. Ce jeton de saut peut être combiné avec un argument de requête **previous-page=true** pour consulter les pages vers l’arrière.

L’exemple de demande suivant indique la pagination vers l’avant :

```
https://graph.microsoft.com/v1.0/users?$top=5$skiptoken=X'4453707402.....0000'
```
Le paramètre **$skiptoken** de la réponse précédente est inclus et vous permet d’obtenir la page de résultats suivante.

L’exemple de demande suivant indique la pagination vers l’arrière :

```
https://graph.microsoft.com/v1.0/users?$top=5$skiptoken=X'4453707.....00000'&previous-page=true
```
Le paramètre **$skiptoken** de la réponse précédente est inclus. Lorsqu’il est combiné avec le paramètre **&previous-page=true**, la page de résultats précédente est récupérée.

Voici les étapes de demande/réponse pour la pagination vers l’avant et vers l’arrière :

1. Une demande est effectuée pour obtenir une liste des 10 premiers utilisateurs sur un total de 15. La réponse contient un jeton de saut pour indiquer la dernière page de 10 utilisateurs.
2. Pour obtenir les 5 utilisateurs finaux, une autre requête est effectuée qui contient le jeton de saut renvoyé de la réponse précédente.
3. Pour consulter les pages vers l’arrière, une demande est effectuée à l’aide du jeton de saut renvoyé à l’étape 1 et le paramètre **&previous-page=true** est ajouté à la demande.
4. La réponse contient la (première) page précédente de 10 utilisateurs. Dans un autre scénario où d’autres pages sont laissées, un nouveau jeton de saut est renvoyé. Ce nouveau jeton de saut peut être ajouté à la demande avec le paramètre **&previous-page=true** pour consulter à nouveau les pages vers l’arrière.

Les restrictions suivantes s’appliquent aux demandes de pagination :

- La taille de page par défaut est 100. La taille de page maximale est 999.
- Les requêtes sur les rôles ne prennent pas en charge la pagination. Cela inclut la lecture d’objets de rôle eux-mêmes ainsi que de membres de rôle.
- La pagination n’est pas prise en charge pour les recherches de lien, comme l’interrogation de membres de groupe.
