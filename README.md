## Exemple des codes d'état HTTP dans une application Spring Boot
Dans ce readme, nous verrons comment renvoyer différents codes d'état HTTP dans Spring Boot, tout en développant une API REST.

### Aperçu
---
Le rôle en tant que développeur d'API est de fournir une bonne expérience à l'utilisateurs et, entre autres, 
de satisfaire cette demande. Permettre aux autres développeurs de déterminer facilement si votre API renvoie 
une erreur ou non vous permet d'aller loin et, dans le premier cas, de faire savoir aux autres développeurs pourquoi 
vous permet d'aller encore plus loin.<br/>

*L'erreur est-elle causée par un service interne de l'API ? Ont-ils envoyé une valeur non analysable ? Le serveur traitant ces requêtes a-t-il carrément planté ?*<br/>

La réduction des possibilités d'échec permet aux développeurs utilisant votre service de faire leur travail plus efficacement. 
C'est là que les codes d'état HTTP entrent en jeu, avec un court message dans le corps de la réponse, décrivant ce qui se passe.

### Que sont les codes d'état HTTP ?
---
En termes simples, un code d'état HTTP fait référence à un code à 3 chiffres faisant partie de la réponse HTTP d'un serveur. 
Le premier chiffre du code décrit la catégorie dans laquelle se situe la réponse. Cela donne déjà un indice pour déterminer 
si la demande a réussi ou non.<br/>

Voici les différentes catégories : <br/>
* **Informationnel (1xx)** : Indique que la demande a été reçue et que le processus se poursuit. Il avertit l'expéditeur d'attendre une réponse finale.
* **Réussi (2xx)** : Indique que la demande a été reçue, comprise et acceptée avec succès.
* **Redirection (3xx)** : indique que d'autres mesures doivent être prises pour terminer la demande.
* **Erreurs client (4xx)** : Indique qu'une erreur s'est produite lors du traitement de la demande et que c'est le client qui a causé l'erreur.
* **Erreurs du serveur (5xx)** : Indique qu'une erreur s'est produite lors du traitement de la requête mais qu'elle a été provoquée par le serveur.

Bien que la liste ne soit pas exhaustive, voici quelques-uns des codes HTTP les plus courants que vous rencontrerez :<br/>
* 200	- d'accord	- La demande a été complétée avec succès.
* 201	- Créé	- Une nouvelle ressource a été créée avec succès.
* 400	- Mauvaise demande	- La demande n'était pas valide.
* 401	- Non autorisé	- La demande n'incluait pas de jeton d'authentification ou le jeton d'authentification avait expiré.
* 403	- Interdit	- Le client n'avait pas l'autorisation d'accéder à la ressource demandée.
* 404	- Pas trouvé	- La ressource demandée n'a pas été trouvée.
* 405	- Méthode Non Autorisée	- La méthode HTTP dans la demande n'était pas prise en charge par la ressource. Par exemple, la méthode DELETE ne peut pas être utilisée avec l'API Agent.
* 500	- Erreur Interne du Serveur	- La demande n'a pas abouti en raison d'une erreur interne côté serveur.
* 503	- service non disponible	- Le serveur n'était pas disponible.

### Renvoyer les codes d'état HTTP dans Spring Boot
---
Dans Spring Boot, il existe de nombreuses situations dans lesquelles nous aimerions décider du code d'état HTTP 
qui sera renvoyé nous-mêmes dans la réponse et Spring Boot nous offre plusieurs façons d'y parvenir.<br/>

* **Renvoi des codes d'état de réponse avec @ResponseStatus** - Cette annotation prend comme argument, le code d'état HTTP, à retourner dans la réponse.
C'est une annotation très polyvalente et peut être utilisée dans les contrôleurs au niveau d'une classe ou d'une méthode, 
sur des classes d'exception personnalisées et sur des classes annotées avec `@ControllerAdvice` (au niveau de la classe ou de la méthode).<br/>
Cela fonctionne de la même manière dans les deux classes annotées avec `@ControllerAdvice` et celles annotées avec `@Controller`.

*Exemple :*<br/>
```
@Controller
@ResponseBody
@ResponseStatus(HttpStatus.SERVICE_UNAVAILABLE)
public class TestController {
    
    @GetMapping("/classlevel")
    public String serviceUnavailable() {
        return "The HTTP Status will be SERVICE_UNAVAILABLE (CODE 503)\n";
    }

    @GetMapping("/methodlevel")
    @ResponseStatus(code = HttpStatus.OK, reason = "OK")
    public String ok() {
        return "Class Level HTTP Status Overriden. The HTTP Status will be OK (CODE 200)\n";
    }    
}
```

* **Renvoi des codes d'état de réponse avec ResponseEntity** : La classe `ResponseEntity` est utilisée lorsque nous spécifions 
par programme tous les aspects d'une réponse HTTP. Cela inclut les en-têtes, le corps et, bien sûr, le code d'état. 
Cette méthode est la manière la plus détaillée de renvoyer une réponse HTTP dans Spring Boot, mais aussi la plus personnalisable.<br/>

*Exemple : *<br/>
```
@Controller
@ResponseBody
public class TestController {
    
    @GetMapping("/response_entity")
    public ResponseEntity<String> withResponseEntity() {
        return ResponseEntity.status(HttpStatus.CREATED).body("HTTP Status will be CREATED (CODE 201)\n");
    }   
}
```

ou

```
@Controller
@ResponseBody
public class TestController {
    
    @GetMapping("/response_entity")
    public ResponseEntity<String> withResponseEntity() {
        int randomInt = new Random().ints(1, 1, 11).findFirst().getAsInt();
        if (randomInt < 9) {
            return ResponseEntity.status(HttpStatus.EXPECTATION_FAILED).body("Expectation Failed from Client (CODE 417)\n");   
        } else {
            return ResponseEntity.status(HttpStatus.I_AM_A_TEAPOT).body("April Fool's Status Code (CODE 418)\n");
        }
    }   
}
```

* **Renvoi des codes d'état de réponse avec ResponseStatusException** : Une classe utilisée pour renvoyer des codes d'état 
dans des cas exceptionnels est la classe `ResponseStatusException`. Il est utilisé pour renvoyer un message spécifique et le 
code d'état HTTP qui sera renvoyé lorsqu'une erreur se produit. C'est une alternative à l'utilisation de `@ExceptionHandler` et 
`@ControllerAdvice`. La gestion des exceptions à l'aide `ResponseStatusException` est considérée comme plus fine.

*Exemple : *<br/>
```
@Controller
@ResponseBody
public class TestController {

    @GetMapping("/rse")
    public String withResponseStatusException() {
        try {
            throw new RuntimeException("Error Occurred");
        } catch (RuntimeException e) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, "HTTP Status will be NOT FOUND (CODE 404)\n");
        }
    }   
}
```

* **Classes d'exception personnalisées et renvoi des codes d'état HTTP** : une autre façon de gérer les exceptions se fait par les annotations `@ResponseStatus` et `@ControllerAdvice` et des classes d'exception personnalisée.

```
@Controller
@ResponseBody
@ResponseStatus(HttpStatus.SERVICE_UNAVAILABLE)
public class TestController {

    @GetMapping("/caught")
    public String caughtException() {
        throw new CaughtCustomException("Caught Exception Thrown\n");
    }

    @GetMapping("/uncaught")
    public String unCaughtException() {
        throw new UnCaughtException("The HTTP Status will be BAD REQUEST (CODE 400)\n");
    }
}
```
Maintenant, définissons ces exceptions et leurs propres codes `@ResponseStatus` par défaut (qui remplacent le statut au niveau de la classe) :
```
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class CaughtCustomException extends RuntimeException{
    public CaughtCustomException(String message) {
        super(message);
    }
}

@ResponseStatus(HttpStatus.BAD_REQUEST)
public class UnCaughtException extends RuntimeException {
    public UnCaughtException(String message) {
        super(message);
    }
}
```
Enfin, nous allons créer un contrôleur `@ControllerAdvice`, qui est utilisé pour configurer la façon dont Spring Boot gère les exceptions :
```
@ControllerAdvice
@ResponseBody
public class TestControllerAdvice {

    @ExceptionHandler(CaughtCustomException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public String handleException(CaughtCustomException exception) {
        return String.format("The HTTP Status will be Internal Server Error (CODE 500)\n %s\n",exception.getMessage()) ;
    }
}
```

### Conclusion
---
Dans ce dêpot, nous avons examiné comment renvoyer les codes d'état HTTP dans Spring Boot à l'aide de `@ResponseStatus`, 
`ResponseEntity` et `ResponseStatusException`, ainsi que comment définir des exceptions personnalisées et les gérer à la fois via 
`@ControllerAdvice` et sans.