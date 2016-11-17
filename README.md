Uber-Like
==============================

Juste un projet perso' pour se familiariser avec d'autres technos en créant un petit prototype Uber, serveur et client.

Ce projet fera l'objet d'une série longue sur [ma chaîne YouTube](https://www.youtube.com/c/louistitifr) j'ai nommé "Let's Play".

Ci-dessous mes notes / idées permettant de structurer cette série de vidéos.

## 1- Introduction
Courant juillet 2016, Uber a communiqué sur les technologies utilisées au seins de son service :
https://eng.uber.com/tech-stack-part-one/
https://eng.uber.com/tech-stack-part-two/

- Code de prototypage, ne pas utiliser en production
- Juste un "Let's Play" donc pour le fun, c'est souvent fait dans les jeux vidéos,
pourquoi pas le faire avec le dev Web
- Se lancer dans l'aventure Node.js, alors lancez-vous avec moi (je débute, donc il y aura sûrement de meilleurs pratique, n'hésitez pas d'ailleurs à les poster en commentaire tout au long du let's play)
- Explication du projet fini
(faire un Uber-Like et avec l'angouement de Pokémon GO l'été dernier, développement d'un système de géocalisation in real time, ici ce sera pour connaître la position de nos entités, chauffeurs et passagers (driver & rider)) :
    - Inscription / Connexion des clients (riders)
    - Déclenchement d'une course par le client
    - Estimated Time of Arrival (ETA) ?
    - Acceptation de la course par un chauffeur (driver)
    - Récupération du rider
    - Dépôt du rider
    - Finalisation de la course (Calcule du coût en fonction du nombre de km)
    - **peut-être faire schéma ?**
- Les technologies qui seront utilisées :
    - Ionic Framework (app hybrid), donc derrière ce sera du JavaScript avec AngularJS côté front,
    - Node.js côté back avec le micro-framework Express
    - ~~MongoDB pour des data nécessitant du temps réel (temporaire)~~
    - Redis pour des data nécessitant du temps réel (temporaire)
    - Socket.io pour la MàJ de la position du rider par exemple et la boucle globale d'une commande en cours (le tout saved dans MongoDB le temps de la course)
    - MySQL pour la persistence à la fin de la commande pour préserver les données
    - J'ai hésité à partir sur Ionic 2 / Angular 2, mais il y a déjà Node.js à prendre en compte, donc c'est pas pour tout de suite. Ici c'est l'apprentissage de Node.js qui sera mis en avant pour ce "let's play".

## 2- Installation & Pré-requis

### Backend

#### Installation
1. Installer Node.js en allant sur : https://nodejs.org/

2. Initialisation (nom du projet : u-like)
    ```sh
    $ npm init
    ```
    
3. Installation de nodemon (globalement car pas besoin de préciser un chemin dans package.json pour le start)
    ```sh
    $ npm install -g nodemon
    ```
    
4. Installation de babel-cli (--save-dev transpiler du code c'est purement développement)
On utilise Babel ici, car actuellement (08 novembre 2016) le moteur V8 de Google (utilisé par Node.js) ne comprend pas les import de modules ES6.
    ```sh
    $ npm install --save-dev babel-cli
    ```
    
5. Installation du preset ES6
    ```sh
    $ npm install --save-dev babel-preset-es2015
    ```
    
6. ESLint pour suivre des normes de développement JavaScript (ici ce sera le style guide d'Airbnb)
   <!-- ```sh
    $ npm install eslint --save-dev
    $ ./node_modules/.bin/eslint --init (dans répertoire /api)
    ```
    
- "Use a popular style guide"
- "Airbnb"
- "JSON"
Nous n'avons pas besoin des dépendances "eslint-plugin-react" et "eslint-plugin-jsx-a11y" car nous n'utilisons pas
React ni jsx.
    ```sh
    $ npm uninstall eslint-plugin-react --save-dev
    $ npm uninstall eslint-plugin-jsx-a11y --save-dev
    ```
    
Retirer les plugins "jsx-a11y" et "react" dans /api/.eslintrc.json. -->

Aujourd'hui il y a un conflict entre les différentes dépendances : https://github.com/eslint/eslint/issues/7338
Solution
```sh
$ npm install eslint-config-airbnb --save-dev
$ npm info eslint-config-airbnb peerDependencies --json
$ npm install --save-dev eslint@^3.9.1 eslint-plugin-jsx-a11y@^2.2.3 eslint-plugin-import@^2.1.0 eslint-plugin-react@^6.6.0
$ ./node_modules/.bin/eslint --init
```
    
- "Use a popular style guide"
- "Airbnb"
- "JSON"

Désactiver certaines règles par défaut d'ESLint via .eslintrc.json.

Ajouter "check": "node_modules/.bin/eslint src/**/*.js" à package.json pour check toutes les sources et modifier "build"

On aura besoin d'autres dépendances, mais pour le moment ça ira, on installera les autres au fur et à mesure que le projet avance.

7. Ajouter start dans scripts pour le développement ; Ajouter serve pour la production ; Le package.json devrait être similaire à :
    ```json
    {
      "name": "u-like",
      "version": "1.0.0",
      "description": "Just a let's play!",
      "main": "src/index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "start": "nodemon --use_strict src/index.js --exec babel-node --presets es2015",
        "check": "node_modules/.bin/eslint src/**/*.js",
        "build": "npm run check && babel src -d dist --presets es2015",
        "serve": "node dist/index.js"
      },
      "author": "Louistiti",
      "license": "MIT",
      "dependencies": {
        "express": "^4.14.0"
      },
      "devDependencies": {
        "babel-cli": "^6.18.0",
        "babel-preset-es2015": "^6.18.0",
        "eslint": "^3.9.1",
        "eslint-config-airbnb": "^13.0.0",
        "eslint-plugin-import": "^2.1.0",
        "eslint-plugin-jsx-a11y": "^2.2.3",
        "eslint-plugin-react": "^6.6.0"
      }
    }
    ```

Car pas de nodemon, ni de Babel en production, donc il suffira de faire
```sh
$ node run build
$ node run serve
```

De cette façon on transpile notre code ES6 en ES5, et on lance le serveur avec le code transpilé de la même manière que sur le serveur de production.

#### Configuration de l'IDE (PhpStorm)
1. IDE Settings > languages & framework > node.js & npm > enable core module
2. IDE Settings > languages & framework > JavaScript > ECMAScript 6
3. mark :
    - api/node_modules
    - api/dist
    - app/node_modules
    - app/platforms
    - app/plugins
    - as exclude directory et changer en "Project Files" dans l'arbre
    (de cette façon lorsque l'on effectue une recherche, ce sera plus simple de retrouver les fichiers que l'on veut)

#### Structure
Ci-dessous la structure des dossiers / fichiers constituant le serveur.
Le code transpilé (à passer en production) est dans le répertoire dist/.

- api/
    - node_modules/
    - dist/
        - ...
    - src/
        - config/
            - config.js
            - database.js
            - server.js
            - ...
        - feature-name/
            - index.js ??? Load models, etc.
            - ...
        - rides/
            - index.js ??? Load models, etc.
            - ride.controller.js
            - ride.model.js
            - ride.routes.js
        - users/
            - index.js ??? Load models, etc.
            - rider.spec.js
            - rider.controller.js
            - rider.model.js
            - driver.controller.js
            - driver.model.js
            - user.routes.js
        - index.js
        - ...
    - package.json

##### A savoir
- index.js point d'entrée (chargement des confs, appel du serveur)
- server.js (initialisation et conf du serveur)

### Frontend

#### Installation
http://ionicframework.com/docs/guide/installation.html (réindenter code)

#### Structure
- app/
    - www/
        - components/
            - component-name.directive.js
            - component-name.directive.html
            - ...
        - layout/
            - header.html
            - footer.html
            - header.controller.js
            - footer.controller.js
        - services/
            - localstorage.service.js
            - loader.service.js
        - users/
            - rider.controller.js
            - driver.controller.js
            - rider.html
            - rider-detail.html
            - driver.html
            - driver-detail.html
            - user.routes.js
            - feature.model.js ???
        - feature-name/
            - ...
        - app.config.js
        - ...
    - index.html

**angular-ui-router est déjà inclu dans Ionic, pas besoin d'injecter le module.**

## 3- Création du serveur
1. Installer Express (--save car c'est une dépendance pour faire tourner notre application)
    ```sh
    $ npm install express --save
    ```

2. Setup configs + middlewares (server class, ...)
Utilisation de import ES6 au lieu des requires, pour sélectionner la partie des modules qui nous intéresse.
Plus performant, on a une mémoire plus libre.

3. First middleware
    ```js
    // Disable from the header, else it makes hacker's life easier to know more about our system
    res.removeHeader('X-Powered-By');
    console.log('request', `${req.method} ${req.url}`);
    ```

## 4- Création et connexion à la base de données
### Création de la base de données
1. Vérifier que le démon (serveur) MySQL est lancé (Windows : services ; mysqld). Sinon le lancer (possible répertoire Wamp, etc.)
2. Se connecter au serveur MySQL (vos identifiants, ici pas de password) :
    ```sh
    $ mysql -h localhost -u root
    ```
    
3. Créer la BDD (UTF-8 Unicode)
    ```sql
    > CREATE DATABASE uberlike COLLATE utf8_unicode_ci;
    > exit
    ```

### Structure
1. Connexion via PhpStorm (ou autre database manager)
2. Création de la table "rider" (passagers)
    ![alt text](https://i.gyazo.com/c4f6f2de6431b9387ea53946c7c64e4d.png "Table Rider")

### Code
1. Installer MySQL dans le projet
    ```sh
    $ npm install mysql --save
    ```

2. Configurer la connexion à MySQL
A savoir que nous nous connectons qu'une fois à la base de données, au lancement du serveur.
Ensuite le serveur attend de nouvelle requêtes (http://i.imgur.com/Hqv5LlG.gifv :D )

Créer config/database.js

## 5- POST /riders
1. Ajouter middleware dans bootstrap() de config/server.js
2. Création de l'entité "rider"
    - users/
        - rider.controller.js
        - rider.model.js
        - rider.routes.js
3. Travailler les paramètres sur des requêtes ayant un verb autre que GET
    ```sh
    $ npm install body-parser --save
    ```

    ```js
    // Parse input values in JSON format
    app.use(bodyParser.json());
    // Parse from x-www-form-urlencoded, which is the universal content type
    app.use(bodyParser.urlencoded({
        extended: true
    }));
    ```
    
4. On installe un package pour la validation de nos données
    ```sh
    $ npm install validator --save
    ```

5. On installe un package pour générer des uuids (pour identifier nos entités)
    ```sh
    $ npm install uuid --save
    ```

6. On va chiffrer le mot de passe
    ```sh
    $ npm install bcrypt --save
    ```
    
Ajouter dossier helpers avec premier helper pour les problématiques de temps (ici datetime()).
Faire logique d'ajout en base de données + errors handling (avec EventEmitter) + tester requête avec Postman.

## 6- Uniformiser nos retours JSON
Créer structure des retours endpoints (succès et erreur) via helper "response.js"

### Erreurs
![alt text](https://i.gyazo.com/2a4ab0fe6a766c7b6b29aed628aac77c.png "Retour erreurs")

### Succès
![alt text](https://i.gyazo.com/2ad6741ae8309a1d36fca9670879f997.png "Retour succès")

## 7- Tester nos endpoints
On vient de créer notre premier endpoint, maintenant automatisons son test.

1. Installer Jasmine (--save-dev car dépendance qu'on a besoin seulement en phase de dev)
    ```sh
    $ npm install jasmine-node --save-dev 
    ```
    
2. Installer Request (exécuter des requêtes pour tester notre API avec Jasmine)
    ```sh
    $ npm install request --save-dev
    ```

SEULEMENT FAIRE TESTS POUR L'API, CAR TROP LONG EN VIDEO DE FAIRE FRONTEND EN PLUS ?
OU FAIRE LES TESTS FRONT EN OFF (faire tests après premier endpoint)

https://semaphoreci.com/community/tutorials/getting-started-with-node-js-and-mocha

[*En cours*]

# Notes
VOIR POUR UTILISER WEBPACK POUR ANGULARJS POUR NE PAS CHARGER PLEINS DE SCRIPT VIA L'INDEX.HTML

FIXER PROBLEMATIQUE : "La boucle qui controle le temps d’annulation tourne bien sur le serveur et n’attend pas la mise à jour de la webapp ? parceque la il ne s’est rien passé pendant 20 min, jusqu’a ce que le Majordome relance son navigateur"
Rendre le serveur autonomme. Node.js corrige déjà ça ? Passer par les websockets (socket.io) ?

UTILISER LES WEBSOCKETS AVEC SOCKET.IO POUR ACTUALISER LA POSITION DU DRIVER
cf http://stackoverflow.com/questions/31715179/differences-between-websockets-and-long-polling-for-turn-based-game-server

FAIRE BARRE DE PROGRESSION ANNIME (PLUS LE DRIVER APPROCHE, PLUS LA COULEUR DEVIENT FONCE)

VOIR POUR FAIRE LES TESTS AVEC JASMINE, ET UTILISER LE RUNNER KARMA
http://jasmine.github.io/
https://karma-runner.github.io
https://www.distelli.com/docs/tutorials/test-your-nodejs-with-jasmine

## Dev tips
- Utiliser "export default" lorsqu'il n'y a seulement qu'un export dans le fichier
- Ne pas exporter des entités mutables (var, let)

# Liens utiles
## Arborescence
- https://code.tutsplus.com/tutorials/build-a-complete-mvc-website-with-expressjs--net-34168
- https://blog.risingstack.com/node-hero-node-js-project-structure-tutorial/
- https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md

## Bases de données
- https://www.linkedin.com/pulse/brief-introduction-mongodb-mysql-mohammadreza-faramarzi
- http://www.theserverside.com/feature/How-NoSQL-MySQL-and-MogoDB-worked-together-to-solve-a-big-data-problem
- https://www.quora.com/Why-does-Quora-use-MySQL-as-the-data-store-instead-of-NoSQLs-such-as-Cassandra-MongoDB-or-CouchDB
- http://gotocon.com/dl/goto-aar-2014/slides/MartyWeiner_ScalingPinterest.pdf
- http://stackoverflow.com/questions/7888880/what-is-redis-and-what-do-i-use-it-for

## Map
- https://www.mapbox.com/
- http://leafletjs.com/

## A savoir
- http://node.green/
- http://stackoverflow.com/questions/22891211/what-is-difference-between-save-and-save-dev
- bcrypt https://codahale.com/how-to-safely-store-a-password/

# Liens plugins / packages
- ngCordova : module Cordova pour Angular pour profiter des composants natifs
http://ngcordova.com/docs/install/ (bower install ngCordova)
- http://ngcordova.com/docs/plugins/geolocation/ (cordova plugin add cordova-plugin-geolocation)
- Utiliser Sass avec Ionic : http://ionicframework.com/docs/cli/sass.html (ionic setup sass)
- Package plumber (npm install --save-dev gulp-plumber) de cette façon ça stoppera pas la tâche, mais affichera les erreurs liées à notre style
- Express : http://expressjs.com/fr/ (micro-framework)
- Nodemon : https://github.com/remy/nodemon (recharge automatiquement application node lorsqu'un fichier est modifié)
- Babel : https://babeljs.io/ transformer ES6 (ECMAScript 2015) en ES5. Implémentation Node.js : https://github.com/babel/example-node-server
- ESLint : https://github.com/eslint/eslint
- Package eslint-config-airbnb : https://www.npmjs.com/package/eslint-config-airbnb
- Package eslint-plugin-import : https://www.npmjs.com/package/eslint-plugin-import
- eslint-plugin-jsx-a11y : https://www.npmjs.com/package/eslint-plugin-jsx-a11y
- eslint-plugin-react : https://www.npmjs.com/package/eslint-plugin-react
- Package mysql : https://www.npmjs.com/package/mysql
- Package body-parser : https://www.npmjs.com/package/body-parser
- Package validator : https://www.npmjs.com/package/validator
- Package uuid : https://www.npmjs.com/package/uuid
- Package bcrypt : https://www.npmjs.com/package/bcrypt
- Package jasmine-node : https://www.npmjs.com/package/jasmine-node
- Package request : https://www.npmjs.com/package/request

# Auteur
**Louis Grenard** : https://www.louistiti.fr

# Licence
MIT License

Copyright (c) 2016 Louistiti <louis.grenard@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.