Um eine einfache Quarkus-Applikation zu erstellen, die ein MQTT-Topic von einem Broker abonniert, befolgen Sie diese Schritte:

1. **Projekt einrichten:**
   - Erstellen Sie ein neues Quarkus-Projekt.
   - Fügen Sie die benötigten Abhängigkeiten für MQTT hinzu.

2. **MQTT-Client konfigurieren:**
   - Konfigurieren Sie die Verbindung zum MQTT-Broker.

3. **Service implementieren:**
   - Implementieren Sie einen Service, der das MQTT-Topic abonniert und die Nachrichten verarbeitet.

Hier ist der Schritt-für-Schritt-Prozess:

### 1. Neues Quarkus-Projekt erstellen

Erstellen Sie ein neues Quarkus-Projekt über das Quarkus CLI oder Maven:

```bash
mvn io.quarkus:quarkus-maven-plugin:create \
    -DprojectGroupId=com.example \
    -DprojectArtifactId=quarkus-mqtt-subscriber \
    -DclassName="com.example.mqtt.MqttSubscriber" \
    -Dpath="/mqtt"
cd quarkus-mqtt-subscriber
```

### 2. Abhängigkeiten hinzufügen

Öffnen Sie die `pom.xml` und fügen Sie die MQTT-Abhängigkeiten hinzu:

```xml
<dependencies>
    <!-- Quarkus Core -->
    <dependency>
        <groupId>io.quarkus</groupId>
        <artifactId>quarkus-resteasy</artifactId>
    </dependency>
    
    <!-- Eclipse Paho MQTT Client -->
    <dependency>
        <groupId>org.eclipse.paho</groupId>
        <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
        <version>1.2.5</version>
    </dependency>
</dependencies>
```

### 3. MQTT-Konfiguration

Erstellen Sie eine Konfigurationsdatei `application.properties` im Ordner `src/main/resources`:

```properties
mqtt.broker=tcp://broker.hivemq.com:1883
mqtt.topic=test/topic
mqtt.clientId=quarkus-client
```

### 4. MQTT-Client Service implementieren

Erstellen Sie eine neue Klasse `MqttService` im Paket `com.example.mqtt`:

```java
package com.example.mqtt;

import io.quarkus.runtime.StartupEvent;
import org.eclipse.paho.client.mqttv3.IMqttDeliveryToken;
import org.eclipse.paho.client.mqttv3.IMqttMessageListener;
import org.eclipse.paho.client.mqttv3.MqttCallback;
import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.eclipse.paho.client.mqttv3.MqttMessage;

import javax.enterprise.context.ApplicationScoped;
import javax.enterprise.event.Observes;
import javax.inject.Inject;
import org.eclipse.microprofile.config.inject.ConfigProperty;

@ApplicationScoped
public class MqttService {

    @ConfigProperty(name = "mqtt.broker")
    String broker;

    @ConfigProperty(name = "mqtt.topic")
    String topic;

    @ConfigProperty(name = "mqtt.clientId")
    String clientId;

    private MqttClient client;

    void onStart(@Observes StartupEvent ev) {
        try {
            client = new MqttClient(broker, clientId);
            MqttConnectOptions connOpts = new MqttConnectOptions();
            connOpts.setCleanSession(true);
            client.connect(connOpts);

            client.subscribe(topic, new IMqttMessageListener() {
                @Override
                public void messageArrived(String topic, MqttMessage message) throws Exception {
                    System.out.println("Received message: " + new String(message.getPayload()));
                }
            });

            System.out.println("Connected to broker: " + broker);

        } catch (MqttException me) {
            me.printStackTrace();
        }
    }
}
```

### 5. Anwendung starten

Starten Sie die Quarkus-Applikation:

```bash
./mvnw compile quarkus:dev
```

Die Applikation wird nun gestartet und abonniert das konfigurierte MQTT-Topic. Eingehende Nachrichten werden in der Konsole ausgegeben.

### Nächste Schritte

**a.** Schreiben Sie Unit-Tests für den MQTT-Service.\
**b.** Fügen Sie erweiterte Fehlerbehandlung und Reconnect-Logik für den MQTT-Client hinzu.
