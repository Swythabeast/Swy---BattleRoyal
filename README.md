# BattleRoyal
Un cadre complet pour les matchs et événements dans Minecraft. Il prend en charge la création de modes via des fichiers de configuration ou des modes entièrement personnalisés via des plugins.

## Modes par défaut
Les jeux actifs dans BattleRoyal sont appelés des Compétitions. BattleRoyal prend en charge nativement deux types de compétitions :

## Match : 
Un jeu qui commence lorsqu'une certaine condition est remplie (par exemple, un nombre de joueurs), ou qui est toujours actif. Ces jeux peuvent être rejoints à tout moment, tant qu'il y a des cartes disponibles.

## Événement : 
Un jeu qui commence à intervalles réguliers ou lorsqu'il est déclenché par une action du serveur. Ces jeux ne peuvent pas être rejoints normalement sauf si l'événement est actif.

## Types de Match intégrés

### Arène : 
Mode de duels simple où vous combattez avec ce qui vous est donné dans la configuration.

### Escarmouche : 
Vous apportez les objets avec lesquels vous souhaitez combattre. Le jeu est toujours en cours, et vous pouvez rejoindre et quitter à tout moment.

### Colisée : 
Combat à mort par équipe en 4 contre 4. L'équipe survivante gagne.

### Champs de bataille : 
Match d'une minute où le gagnant est le joueur avec le plus de kills.

## Types d'Événements intégrés

### Chacun pour soi : 
Combat à mort chacun pour soi qui commence toutes les 30 minutes. Le dernier joueur en vie gagne.

### Match à mort : 
Événement de 2 minutes où, si vous mourrez, vous réapparaissez. Le joueur avec le plus grand nombre de kills gagne.

### Tournoi : 
Tournoi à élimination directe pour un nombre quelconque d'équipes.

## Pour les développeurs

BattleRoyal est conçu pour être facilement extensible. Vous pouvez créer vos propres modes, événements, et même compétitions. Vous pouvez également créer vos propres commandes et écouteurs pour gérer les événements à votre manière.

Créer une Arène

Dans BattleRoyal, la logique principale d'un jeu se trouve dans la classe Arena. Cette classe est responsable de la gestion de la logique du jeu et est la classe principale à étendre lors de la création d'un nouveau mode. La plupart des aspects de BattleRoyal sont basés sur des événements, ce qui signifie que plutôt que d'implémenter ou de remplacer des méthodes, vous écouterez divers événements du jeu ou ajouterez les vôtres. Voici un exemple simple d'une classe Arena :
```java
public class MyArena extends Arena {
    
    @ArenaEventHandler
    public void onArenaJoin(ArenaJoinEvent event) {
        event.getPlayer().sendMessage("Bienvenue dans mon arene!");
    }
}
```

Et pour l'enregistré:
```java
BattleArena.getInstance().registerArena("MyArena", MyArena.class, MyArena::new);
```

### Événements d'Arène
Les événements d'Arène sont au cœur de BattleArena. Ils sont déclenchés lorsque certaines actions se produisent dans le jeu et peuvent être utilisés pour écouter et gérer ces actions. BattleArena opte pour l'annotation @ArenaEventHandler comparée à l'annotation @EventHandler de Bukkit, car elle permet de capturer des événements spécifiquement dans une Arène, plutôt qu'à l'échelle globale. 

Une liste de tous les événements d'Arène se trouve dans le package org.battleplugins.arena.event.

Voici un exemple de gestionnaire d'événements d'Arène :
```java
@ArenaEventHandler
public void onInteract(PlayerInteractEvent event) {
    event.getPlayer().sendMessage("Interact while in Arena!");
}
```

Les écouteurs d'événements d'Arène doivent implémenter la classe 'ArenaListener' plutôt que la classe 'Listener' de Bukkit et, au lieu d'être enregistrés via le PluginManager de Bukkit, ils sont enregistrés via l''ArenaEventManager'. Voici un exemple d'enregistrement d'un écouteur d'événements d'Arène :
```java
Arena arena = ...; // your Arena instance
arena.getEventManager().registerEvents(new MyArenaListener());
```

Il est important de noter que l'annotation '@ArenaEventHandler' ne fonctionnera pas pour chaque événement. Elle ne peut écouter que les événements qui peuvent capturer un joueur (par exemple, 'PlayerInteractEvent', 'PlayerMoveEvent', etc.), car BattleRoyal doit savoir dans quelle Arène déclencher l'événement. Tout événement qui implémente 'PlayerEvent' ou 'EntityEvent' sera automatiquement capturé par cela. Si vous souhaitez implémenter votre propre résolveur pour capturer un événement dans une Arène, vous pouvez utiliser la méthode 'ArenaEventManager#registerArenaResolver'.

### Créer des Déclencheurs d'Événements Personnalisés
BattleRoyal possède plusieurs déclencheurs d'événements implémentés par défaut et utilisés dans la configuration. Cela inclut on-join, on-complete, et beaucoup d'autres utilisés dans tout le plugin. Cependant, vous pouvez créer vos propres déclencheurs d'événements à utiliser dans la configuration. 

Pour ajouter le vôtre, assurez-vous que votre classe 'Event' implémente la classe 'A'renaEvent' ou 'ArenaPlayerEvent'. La différence entre les deux est que 'ArenaPlayerEvent' ne capturera qu'un seul joueur, c'est-à-dire le joueur dans l'événement (voir on-join comme exemple), tandis que 'ArenaEvent' sera déclenché pour tous les joueurs dans une compétition (voir on-complete comme exemple).

Une fois que vous avez ajouté l'annotation '@EventTrigger', exécutez 'ArenaEventType.create(<nom>, <classe de l'événement>)' qui créera le type d'événement et permettra son utilisation dans la configuration. Ensuite, pour déclencher cela, appelez votre événement via l''ArenaEventManager' visible dans votre classe 'Arena'.