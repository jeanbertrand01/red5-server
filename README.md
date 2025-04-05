# RENDU DU PROJET : AMELIORATION DE RED5-SERVER


## Contributeur :

- Jean Bertrand KAMTCHOUM YONGA


## 6 - PETITES MODIFICATIONS 

>1. Renommage d'une classe, d'une methode et d'une variable

- **Classe : common.src.main.java.org.red5.server.Client => common.src.main.java.org.red5.server.Red5Client**

Raison : Le nom Client est générique. En le renommant en Red5Client, on indique clairement que cette classe est spécifique au projet Red5, évitant ainsi toute ambiguïté avec d'autres classes nommées Client.

- **Méthode stop() de la classe:  server.src.main.java.org.red5.server.net.rtmp.RTMPMinaTransport** renommée en **stopTransport()**

Raison : Le nom stop() est générique. En le renommant en stopTransport(), on précise que cette méthode est destinée à arrêter le transport RTMP, améliorant ainsi la clarté et la lisibilité du code.


- **Attribut log de la classe: server.src.main.java.org.red5..spring.InetAddressEditor** renommée en **logger**

```bash
private static Logger logger = LoggerFactory.getLogger(InetAddressEditor.class);
```

En le renommant en **logger**, on indique clairement que c'est un attribut de la classe InetAddressEditor et non de toute une autre classe.

>2. changer le type ou le nombre de paramètres d’une méthode

Dans la classe **BaseConnection.java (package:server.src.main.java.org.red5.server.net)** : 

Je change le type de retour de **void addListener(IConnectionListener listener)**, en **boolean addListener(IConnectionListener listener)** pour retourner true lorqu'elle a correctement ajouter un listener, et false sinon.

on modifie aussi dans l'interface **IConnection.java** la methode **addListener(IConnectionListener listener)** en **boolean addListener(IConnectionListener listener)** .


>3. créer des variables pour supprimer des nombres magiques.

Je lance le script **detect_magic_number.sh**, pour analyser le projet.

Resultat : je ne trouve aucun nombres magiques dans aucune classe.


>4. supprimer du code mort

On vas supprimer du code avec la mention **@Deprecated**.

- Je supprime dans **BaseConnection.java (package:server.src.main.java.org.red5.server.net)**, la methode **setId(int clientId)** 
- Je supprime les methodes **getDeadlockGuardScheduler()**, **setDeadlockGuardScheduler(ThreadPoolTaskScheduler deadlockGuardScheduler)** de **RTMPConnection.java (package:common.src.main.java.org.red5.server.net.rtmp)**.

>5. Réorganiser une classe pour le code soit bien structuré, les variables d’instance en début de classe, puis méthodes publiques et enfin méthodes privées

Je réorganise la classe **RTMPMinaTransport.java,(package : server.src.main.java.org.red5.server.net.rtmp)** les variables d'instances sont deja correctement mises au debut de la classe, les methodes **setters** et **getter** viennent ensuite, et les methodes privées suivent et enfin les methodes publiques a la fin.

## 7 - MOYENNES MODIFICATIONS

>1. réduire la complexité cyclomatique ou le nombre de lignes d’une méthode

Je m'interrese a la methode **playVOD(boolean withReset, long itemLength)** de la classe **PlayEngine.java, (package: common.src.main.java.org.red5.server.stream)**, en reduisant le nombre de lignes de la methode.

les blocs

```java
if(){
    // 1 ligne de code
}
```

deviennent

```java
if() // 1 ligne de code
```

>2. décomposer une méthode qui à la fois retourne des informations et modifie l’état d’un objet

Je decompose la methode **playVOD(boolean withReset, long itemLength)** de la classe **PlayEngine.java, (package: common.src.main.java.org.red5.server.stream)** en 2 methodes.

```java
private final IMessage sendIMessage(long itemLength) throws IOException {
    IMessage msg = null;
    IMessageInput in = msgInReference.get();
    msg = in.pullMessage();
    if (msg instanceof RTMPMessage) {
        // Only send first video frame
        IRTMPEvent body = ((RTMPMessage) msg).getBody();
        if (itemLength == 0) {
            while (body != null && !(body instanceof VideoData)) {
                msg = in.pullMessage();
                if (msg != null && msg instanceof RTMPMessage) body = ((RTMPMessage) msg).getBody();
                else break;
            }
        }
        if (body != null) 
            // Adjust timestamp when playing lists 
            body.setTimestamp(body.getTimestamp() + timestampOffset); 
    }
    return msg;
}
```
et 

```java
private final IMessage playVOD(boolean withReset, long itemLength) throws IOException {
    // change state
    subscriberStream.setState(StreamState.PLAYING);
    if (withReset) releasePendingMessage();
        
    sendVODInitCM(currentItem.get());
    // Don't use pullAndPush to detect IOExceptions prior to sending NetStream.Play.Start
    int start = (int) currentItem.get().getStart();
    if (start > 0) {
        streamOffset = sendVODSeekCM(start);
        // We seeked to the nearest keyframe so use real timestamp now
        if (streamOffset == -1) streamOffset = start;
    }
        
    return sendIMessage(itemLength);
}
```

>3. remplacer le fait qu’une méthode retourne un code d’erreur par le fait qu’elle lève une exception

Dans la classe **BaseConnection.java (package:server.src.main.java.org.red5.server.net)** :

La methode :

```java
public boolean addListener(IConnectionListener listener) {
    if (connectionListeners != null) {
        connectionListeners.add(listener);
        return true;
    }
    return false;
}
```

retourne un boolean indiquant si la methode a correctement ajouter un listener ou non. je modifie la methode pour qu'elle lance une exception si elle ne peut pas ajouter le listener.

```java
public void addListener(IConnectionListener listener) throws ListenerAddException {

    if (connectionListeners != null) connectionListeners.add(listener);

    else throw new ListenerAddException("Could not add listener");
}
```

>4. supprimer de la duplication de codes entre méthodes




