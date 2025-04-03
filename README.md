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

