# WebAssembly dans Docker : vers une cohabitation pacifique ?WebAssembly dans Docker : vers une cohabitation pacifique ?


## WebAssembly

WebAssembly (abrégé en wasm) est un langage binaire portable et performant conçu pour être exécuté dans les navigateurs web. Il permet d'exécuter du code à haute performance à travers différentes plates-formes et architectures, en fournissant une alternative aux langages de programmation traditionnels tels que JavaScript pour le développement web.

Les caractéristiques clés de WebAssembly sont les suivantes :

**Performances élevées** : WebAssembly est conçu pour être exécuté plus rapidement que le code JavaScript équivalent, ce qui permet d'exécuter des applications plus complexes et gourmandes en ressources de manière plus fluide dans un navigateur.

**Portabilité** : Les programmes WebAssembly peuvent être exécutés sur différentes plates-formes, y compris les navigateurs web, les environnements d'exécution côté serveur et même hors ligne.

**Sécurité** : WebAssembly est conçu avec la sécurité à l'esprit. Il fonctionne dans un environnement sandboxed (bac à sable) qui isole le code WebAssembly du reste du système, réduisant ainsi les risques de failles de sécurité.

**Interopérabilité** : WebAssembly peut être utilisé aux côtés de JavaScript et d'autres langages de programmation, permettant aux développeurs d'intégrer des composants écrits dans différents langages au sein de la même application.

**Taille compacte**: Les fichiers WebAssembly sont généralement plus petits que leurs équivalents JavaScript, ce qui permet un chargement plus rapide des applications web.

WebAssembly est souvent utilisé pour exécuter des applications ou des bibliothèques qui nécessitent des performances élevées, comme les jeux en ligne, les simulations, les applications de conception assistée par ordinateur (CAO) et d'autres applications complexes directement dans le navigateur, sans avoir à dépendre uniquement de JavaScript pour cela.
Docker livre un premier aperçu de l’intégration de WebAssembly. Comment aborde-t-il la cohabitation avec les conteneurs ?

_WebAssembly (Wasm) signera-t-il la fin des conteneurs ? Pas plus que ces derniers n’ont signé la fin des VM, veut-on croire chez Docker._

![Github](https://i0.wp.com/nigelpoulton.com/wp-content/uploads/2022/11/solomon-1.png?resize=1024%2C359&ssl=1)

L’entreprise exprime ce point de vue de longue date. Elle a toutefois accentué, ces derniers mois, sa communication à ce sujet. En toile de fond, l’intégration de WebAssembly dans sa boîte à outils.
Le chantier vient de se concrétiser. On peut en avoir un premier aperçu – instable – dans la preview de Docker Desktop.

L’intégration de Wasm repose sur un shim pour containerd. Et sur une fonctionnalité actuellement expérimentale dans Docker : la gestion des images par containerd. Une fois l’image récupérée, le runtime extrait et charge le module Wasm, puis le réseau est configuré.

![Contaierd](https://www.silicon.fr/wp-content/uploads/2022/10/Docker-Wasm-sch%C3%A9ma.jpg)

# Jouer avec WebAssembly et Docker

Créer target wasi:

`cargo build --target wasm32-wasi --release`

Créer target native

`cargo build --release`

## Creation Dockerfiles

_Dockerfile-wasm_

```
1. FROM rust:1.70-slim-bullseye as build                                    

COPY Cargo.toml .
COPY Cargo.lock .
COPY src src

2. RUN rustup target add wasm32-wasi                                        

3. RUN cargo build --target wasm32-wasi --release                           

4. FROM scratch                                                             

5. COPY --from=build /target/wasm32-wasi/release/hello-docker.wasm wasm.wasm 

ENTRYPOINT [ "/wasm.wasm" ]

```

_Dockerfile-native_

```
FROM rust:1.70-slim-bullseye as build

COPY Cargo.toml .
COPY Cargo.lock .
COPY src src

RUN RUSTFLAGS='-C target-feature=+crt-static' cargo build --release 

FROM scratch                                                        

COPY --from=build /target/release/hello-docker native

```