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

Les statistiques que me donne sonarqube, sur la duplication de code sont:

- Densité : **3.3%**
- Nombres de blocs de code identiques : **142**
- Nombres de lignes de code identiques : **3443**
- Nombres de classes identiques : **68**

Je m'interesse a la classe **Aggregate.java (package:common.src.main.java.org.red5.server.net.rtmp.event)**, qui dans la methode **releaseInternal()** utilise la condition :

```java
if(data!=null){
    
}
```
ainsi que dans la methode **writeExternal(ObjectOutput out)**. 

je factorise cette condition dans un methode **isDataNotNull()**.

```java
private static boolean isDataNotNull(Object data) {
    return data != null;
}
```

>5. ajouter un test pertinent

Pour ajouter un test pertinent, concentrons-nous sur la méthode refactorisée `isDataNotNull` dans la classe **Aggregate.java**. Voici un exemple de test unitaire pour cette méthode :

### Exemple de test unitaire

```java
package org.red5.server.net.rtmp.event;

import static org.junit.Assert.assertFalse;
import static org.junit.Assert.assertTrue;

import org.junit.Test;

public class AggregateTest {

    @Test
    public void testIsDataNotNull() {
        // Cas où l'objet est non null
        Object nonNullData = new Object();
        assertTrue("L'objet non null devrait retourner true", Aggregate.isDataNotNull(nonNullData));

        // Cas où l'objet est null
        Object nullData = null;
        assertFalse("L'objet null devrait retourner false", Aggregate.isDataNotNull(nullData));
    }
}
```

## 8 - GRANDES MODIFICATIONS

>1. décomposer une god classe

La classe **PlayEngine.java (package:common.src.main.java.org.red5.server.stream)** est un **god class**. Je m'interresse donc a la decomposition de cette classe.

---

### Proposition de décomposition pour la classe `PlayEngine`

#### Problèmes identifiés :
- La méthode `playVOD` gère plusieurs responsabilités :
  - Modifier l'état du flux (`subscriberStream.setState`).
  - Libérer des messages en attente (`releasePendingMessage`).
  - Initialiser la lecture VOD (`sendVODInitCM`).
  - Gérer les sauts dans le flux vidéo (`sendVODSeekCM`).
  - Envoyer des messages (`sendIMessage`).

#### Décomposition proposée :
Nous pouvons extraire ces responsabilités dans des classes ou méthodes spécialisées.

---

#### 1. **Classe `StreamStateManager`** :
Responsable de la gestion des états du flux.

```java
public class StreamStateManager {
    public void setStreamState(SubscriberStream stream, StreamState state) {
        stream.setState(state);
    }
}
```

---

#### 2. **Classe `MessageManager`** :
Responsable de la gestion des messages en attente et de l'envoi des messages.

```java
public class MessageManager {
    public void releasePendingMessage() {
        // Implémentation pour libérer les messages en attente
    }

    public IMessage sendIMessage(IMessageInput input, long itemLength, long timestampOffset) throws IOException {
        IMessage msg = input.pullMessage();
        if (msg instanceof RTMPMessage) {
            IRTMPEvent body = ((RTMPMessage) msg).getBody();
            if (itemLength == 0) {
                while (body != null && !(body instanceof VideoData)) {
                    msg = input.pullMessage();
                    if (msg != null && msg instanceof RTMPMessage) {
                        body = ((RTMPMessage) msg).getBody();
                    } else {
                        break;
                    }
                }
            }
            if (body != null) {
                body.setTimestamp(body.getTimestamp() + timestampOffset);
            }
        }
        return msg;
    }
}
```

---

#### 3. **Classe `VODInitializer`** :
Responsable de l'initialisation et de la gestion des sauts dans le flux VOD.

```java
public class VODInitializer {
    public void sendVODInitCM(CurrentItem currentItem) {
        // Implémentation pour initialiser la lecture VOD
    }

    public long sendVODSeekCM(CurrentItem currentItem, int start) {
        // Implémentation pour gérer les sauts dans le flux
        return start; // Exemple de retour
    }
}
```

---

#### 4. **Classe refactorisée `PlayEngine`** :
La classe `PlayEngine` délègue désormais les responsabilités aux classes spécialisées.

```java
public class PlayEngine {
    private StreamStateManager stateManager = new StreamStateManager();
    private MessageManager messageManager = new MessageManager();
    private VODInitializer vodInitializer = new VODInitializer();

    public IMessage playVOD(boolean withReset, long itemLength, SubscriberStream subscriberStream, CurrentItem currentItem, IMessageInput input, long timestampOffset) throws IOException {
        // Modifier l'état du flux
        stateManager.setStreamState(subscriberStream, StreamState.PLAYING);

        // Libérer les messages en attente si nécessaire
        if (withReset) {
            messageManager.releasePendingMessage();
        }

        // Initialiser la lecture VOD
        vodInitializer.sendVODInitCM(currentItem);

        // Gérer les sauts dans le flux
        int start = (int) currentItem.getStart();
        long streamOffset = 0;
        if (start > 0) {
            streamOffset = vodInitializer.sendVODSeekCM(currentItem, start);
            if (streamOffset == -1) {
                streamOffset = start;
            }
        }

        // Envoyer les messages
        return messageManager.sendIMessage(input, itemLength, timestampOffset);
    }
}
```

---

### Avantages de la décomposition :
1. **Responsabilités séparées** : Chaque classe a une responsabilité unique, ce qui améliore la lisibilité et la maintenabilité.
2. **Réutilisabilité** : Les classes comme `MessageManager` ou `StreamStateManager` peuvent être réutilisées ailleurs dans le projet.
3. **Testabilité** : Les classes plus petites sont plus faciles à tester unitairement.

>2. ajouter une super classe pour supprimer des méthodes dupliquées

Pour supprimer des méthodes dupliquées, vous pouvez introduire une **super classe** qui regroupe les fonctionnalités communes. Voici comment procéder :



>3. supprimer des classes static

Grâce a la commande shell :

```bash
grep -rnw . -e "static class" --include="*.java"
```

j'identifie les classes statics, et je les supprime.

- classe **/extras/audio/mp3/src/main/java/org/red5/io/mp3/impl/MP3Stream.java**, ligne 351 : je supprime la classe :

```java
/**
     * A class representing the bit field of an MPEG header. It allows convenient access to specific bit groups.
     */
    private static class HeaderBitField {
        /** The internal value. */
        private int value;

        /**
         * Adds a byte to this field.
         *
         * @param b
         *            the byte to be added
         */
        public void add(int b) {
            value <<= 8;
            value |= b;
        }

        /**
         * Returns the value of the bit group from the given start and end index. E.g. ''from'' = 0, ''to'' = 3 will return the value of the first 4 bits.
         *
         * @param the
         *            from index
         * @param to
         *            the to index
         * @return the value of this group of bits
         */
        public int get(int from, int to) {
            int shiftVal = value >> from;
            int mask = (1 << (to - from + 1)) - 1;
            return shiftVal & mask;
        }

        /**
         * Returns the value of the bit with the given index. The bit index is 0-based. Result is either 0 or 1, depending on the value of this bit.
         *
         * @param bit
         *            the bit index
         * @return the value of this bit
         */
        public int get(int bit) {
            return get(bit, bit);
        }

        /**
         * Returns the internal value of this field as an array. The array contains 3 bytes.
         *
         * @return the internal value of this field as int array
         */
        public byte[] toArray() {
            byte[] result = new byte[3];
            result[0] = (byte) get(16, 23);
            result[1] = (byte) get(8, 15);
            result[2] = (byte) get(0, 7);
            return result;
        }
    }
```

ainsi de suite pour les autres classes statics.


>4. fusionner des classes

