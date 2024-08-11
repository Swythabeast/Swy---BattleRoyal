# BattleArena
Un cadre complet pour les matchs et événements dans Minecraft. Il prend en charge la création de modes via des fichiers de configuration ou des modes entièrement personnalisés via des plugins.

## Modes par défaut
Les jeux actifs dans BattleArena sont appelés des Compétitions. BattleArena prend en charge nativement deux types de compétitions :

## Match : 
Un jeu qui commence lorsqu'une certaine condition est remplie (par exemple, un nombre de joueurs), ou qui est toujours actif. Ces jeux peuvent être rejoints à tout moment, tant qu'il y a des cartes disponibles.

## Événement : 
Un jeu qui commence à intervalles réguliers ou lorsqu'il est déclenché par une action du serveur. Ces jeux ne peuvent pas être rejoints normalement sauf si l'événement est actif.

### Types de Match intégrés

Arène : Mode de duels simple où vous combattez avec ce qui vous est donné dans la configuration.
Escarmouche : Vous apportez les objets avec lesquels vous souhaitez combattre. Le jeu est toujours en cours, et vous pouvez rejoindre et quitter à tout moment.
Colisée : Combat à mort par équipe en 4 contre 4. L'équipe survivante gagne.
Champs de bataille : Match d'une minute où le gagnant est le joueur avec le plus de kills.
Types d'Événements intégrés
Chacun pour soi : Combat à mort chacun pour soi qui commence toutes les 30 minutes. Le dernier joueur en vie gagne.
Match à mort : Événement de 2 minutes où, si vous mourrez, vous réapparaissez. Le joueur avec le plus grand nombre de kills gagne.
Tournoi : Tournoi à élimination directe pour un nombre quelconque d'équipes.
Pour les développeurs
BattleArena est conçu pour être facilement extensible. Vous pouvez créer vos propres modes, événements, et même compétitions. Vous pouvez également créer vos propres commandes et écouteurs pour gérer les événements à votre manière.

Créer une Arène
Dans BattleArena, la logique principale d'un jeu se trouve dans la classe Arena. Cette classe est responsable de la gestion de la logique du jeu et est la classe principale à étendre lors de la création d'un nouveau mode. La plupart des aspects de BattleArena sont basés sur des événements, ce qui signifie que plutôt que d'implémenter ou de remplacer des méthodes, vous écouterez divers événements du jeu ou ajouterez les vôtres. Voici un exemple simple d'une classe Arena :
```java
public class MyArena extends Arena {
    
    @ArenaEventHandler
    public void onArenaJoin(ArenaJoinEvent event) {
        event.getPlayer().sendMessage("Welcome to my arena!");
    }
}
```

And for registering it:
```java
BattleArena.getInstance().registerArena("MyArena", MyArena.class, MyArena::new);
```

### Arena Events
Arena events are at the core of BattleArena. They are fired when certain actions happen in the game, and can be used to listen for and handle these actions. BattleArena opts to use the `@ArenaEventHandler` annotation compared to Bukkit's `@EventHandler` annotation, as it allows for capturing events specifically in an Arena, rather than globally. 

A list of all Arena events can be found in the `org.battleplugins.arena.event` package.

Here is an example of an Arena event handler:
```java
@ArenaEventHandler
public void onInteract(PlayerInteractEvent event) {
    event.getPlayer().sendMessage("Interact while in Arena!");
}
```

Arena event listeners must implement the `ArenaListener` class rather than Bukkit's `Listener` class and rather than registered through Bukkit's `PluginManager`, they are registered through the `ArenaEventManager`. Here is an example of registering an Arena event listener:
```java
Arena arena = ...; // your Arena instance
arena.getEventManager().registerEvents(new MyArenaListener());
```

It is important to note that the `@ArenaEventHandler` annotation will not work for every event. They can only listen for events that can capture a player (i.e. PlayerInteractEvent, PlayerMoveEvent, etc.) as BattleArena needs to know which Arena to fire the event in. Any event that implements `PlayerEvent` or `EntityEvent` will automatically be captured by this. If you wish to implement your own resolver to capture an event to an Arena, you can use the `ArenaEventManager#registerArenaResolver` method.

### Creating Custom Event Triggers
BattleArena has multiple event triggers implemented by default used in the config. These include `on-join`, `on-complete`, and many others used throughout the plugin. However, you can create your own event triggers to use in the config. 

In order to add your own, ensure your `Event` class implements the `ArenaEvent` or `ArenaPlayerEvent` class. The difference between the two is that `ArenaPlayerEvent` will only capture a single player, which is the player in the event (see `on-join` as an example), while `ArenaEvent` will be fired for all players in a competition (see `on-complete` as an example).

Once you have added the `@EventTrigger` annotation, then run `ArenaEventType.create(<name>, <event class>)` which will create the event type and allow it to be used in the config. Then, in order to trigger this, call your event through the `ArenaEventManager` visible in your `Arena` class.