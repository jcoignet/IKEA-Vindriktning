# IKEA Vindriktning to Home Assistant
## Introduction
Le but de ce tuto est de connecter le capteur IKEA Vindriktning sur Home Assistant en utilisant un ESP.  
Pour ce tuto le WEMOS D1 mini lite a été utilisé.  
Il est possible de passer le ventilo sur du 3.3V pour le rendre plus silencieux, cependant la manip semble affecter les mesures https://github.com/Hypfer/esp8266-vindriktning-particle-sensor/issues/47 elle n'est donc pas appliquéé dans ce tuto.  

## Hardware
- [IKEA Vindriktning](https://www.ikea.com/fr/fr/p/vindriktning-capteur-qualite-de-lair-70498242/)
- [WEMOS D1 mini lite](https://www.amazon.fr/gp/product/B0754N794H/ref=ppx_yo_dt_b_asin_title_o01_s01?ie=UTF8&psc=1) ou autre board ESP8266/ESP32 avec wifi
- Cables (environ 3x5cm)
- Fer à souder
- Cable usb-C (non inclu avec le Vindriktning)
- Cable micro usb (pour flasher l'ESP)

## Installation
### Etape 1 : Install & flash
- /!\ WEMOS D1 MINI /!\ : Sur un PC windows, télécharger et installer le [driver CH230 de wemos](https://www.wemos.cc/en/latest/ch340_driver.html). Pour des boards différents, le driver ne doit pas être le même.
- Dans HA, installer l'addon [ESPHome](https://my.home-assistant.io/redirect/supervisor_addon/?addon=5c53de3b_esphome&repository_url=https%3A%2F%2Fgithub.com%2Fesphome%2Fhome-assistant-addon)
- Dans ESPHome sur HA, créer la device. Remplir name (ex: ikea-vindriktning-01) et les creds du WIFI.
- Suivre le procédé d'installation sur l'interface HA. La board doit être connectée au PC via le port micro usb (evidement avec un cable data).
- Vérifier que la connection wifi fonctionne. Débrancher du PC et brancher sur secteur. Même si la device apparaît offline dans l'HA, vérifier si l'on recoit bien les logs dans `Logs > Wirelessly`.
- Dans ESPHome, modifier la configuration de l'appareil. 

Il faut ajouter
```
uart:
  rx_pin: D2 # Le bon port en fonction du board
  baud_rate: 9600

sensor:
  - platform: pm1006
    pm_2_5:
      name: "Particulate Matter 2.5µm Concentration"
```
Et modifier
```
esp8266:
  board: d1_mini_lite # Le bon board cf https://registry.platformio.org/platforms/platformio/espressif8266/boards

# Désactiver le logger
logger:
  baud_rate: 0
```

Optionnel:
En cas de problème de résolution DNS, mettre une IP statique ajouter (avec la bonne IP):
```
wifi:
  manual_ip:  # Custom
      # Set this to the IP of the ESP
    static_ip: 192.168.0.25
    # Set this to the IP address of the router. Often ends with .1
    gateway: 192.168.0.1
    # The subnet of the network. 255.255.255.0 works for most home networks.
    subnet: 255.255.255.0
```

Voir le fichier ikea-vindriktning-01.yaml pour un exemple complet de configuration
- Faire SAVE et INSTALL

### Etape 2 : Soudure
- Préparer 3 cables d'environ 5cm : dénuder, tordre et présouder le fil
- Démonter entièrement l'IKEA Vindriktning : retirer les 4 vis du boitier, débrancher le capteur + ventilo du board, retirer les 3 vis du board
- Souder :
    - Cable rouge : +5V vers 5V
    - Cable noir : GND vers G
    - Cable marron : RST vers D2

![V1nofan](https://user-images.githubusercontent.com/6169199/198835796-f17a8df0-d156-4547-a4cf-8bd2dca67c6a.jpg)

### Etape 3 : Vérification
Alimenter via le port USB-C. La device devrait être détectée dans l'HA et dans les logs sur ESPHome vous devriez voir apparaître:
```
[15:04:43][D][pm1006:091]: Got PM2.5 Concentration: 183 µg/m³
[15:04:43][D][sensor:126]: 'Particulate Matter 2.5µm Concentration': Sending state 183.00000 µg/m³ with 0 decimals of accuracy
```

Dans le firmware IKEA, le palier de la LED jaune est 30de 30μg/m³ et rouge à partir de 100μg/m³.
## Liens
https://next.esphome.io/components/sensor/pm1006.html  
https://www.instructables.com/Connecting-a-IKEA-Vindriktning-to-Home-Assistant-U/
