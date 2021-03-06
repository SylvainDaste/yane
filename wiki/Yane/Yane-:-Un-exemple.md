Dans ce tutoriel nous allons émuler le réseau suivant :

host-A	=============	host-B


Celui-ci est composé de 2 hôtes (A et B) reliés par un simple lien.

Vous pouvez récupérez le fichier yane.yml dans examples/tuto-1/yane.yml


## Créer le réseau

Pour spécifier notre réseau à yane, nous utilisons YAML. Créons le fichier yane.yml dans lequel nous saisissons les lignes suivantes :

```yaml
network:
  name: basic
  version: 1.0
  hosts:
    -
      name: host-a
      mode: netns
    -  
      name: host-b
      mode: netns
```

Expliquons un peu ! Tout d'abord le réseau doit avoir un nom et une version. Ensuite nous définissons nos hôtes par un `name` et un `mode` grâce à la balise `hosts`. Vous pouvez spécifier autant d'hôtes que vous le voulez. Le `mode` fait référence à la technique de virtualisation utilisée pour cet hôte. Ici nous avons pris `netns`, mais nous aurions très bien pu prendre `docker`.

**Attention**: le YAML n'utilise aucunes tabulations pour l'indentation, veillez à utiliser des espaces.

## Créer des liens

Un lien est spécifié par de la façon suivante :`if1!if2`. Cela signifie qu'un lien relie deux interfaces (`ifx`). Les interfaces sont de la forme : `h:i[:a]`. *h* correspond au nom du premier hôte, *i* au nom de l'interface de cet hôte. Enfin on peut ajouter à chaque interface une adresse IPv4 `a`.
On peut alors ajouter à notre fichier les deux lignes suivantes :

```yaml
links:
  - host-a:v0:192.168.1.1/24!host-b:v0:192.168.1.2/24
```

## Lancer la simulation

Afin d'interagir avec notre réseau. On peut afficher les consoles de chaque hôte en ajoutant ces lignes à yane.yml :

```yaml
consoles:
  - all
```
Si l'on souhaite seulement certaines consoles il suffit de les spécifier de la façon suivante :
```yaml
consoles:
  - host-A host-B
```
Enfin on peut lancer la simulation :

`# yane`

Si on préfère on peut utiliser le mode _verbose_ qui affiche plus de description (yane affiche les messages LOG):

`# yane -v`

## Tester notre simulation

Basculez sur la console de `host-a`. Nous allons essayer d'écouter notre interface :
```
# tcpdump
```
Sur la console de `host-b`. Essayez d'envoyer des données à `host-a`, on peut, par exemple, faire un ping :
```
	# ping -c 1 192.168.1.1
```
Si tout s'est déroulé comme prévu vous devriez obtenir sur `host-a` :

	17:19:23.300866 IP 192.168.1.1 > r2d2: ICMP echo request, id 8751, seq 1, length 64
	17:19:23.300887 IP r2d2 > 192.168.1.1: ICMP echo reply, id 8751, seq 1, length 64


Maintenant nous aimerions pouvoir garder traces de tous les échanges protocolaires qui se sont déroulés entre nos hôtes.
Pour cela _yane_ peut "sniffer" chaque interfaces.
Pour "sniffer" toutes les interfaces de notre réseau :
```yaml
dumpif:
  - all
```
Si vous relancez la simulation vous allez voir cette ligne apparaître :

```
wireshark -i /tmp/nssi/16977/host-a/v0 -i /tmp/nssi/16977/host-b/v0 `
```
Ce qui signifie que vous pouvez lancer wireshark pour observer le trafic de vos interfaces.

## Stopper la simulation

On peut arrêter la simulation en tapant (uniquement s'il n'y qu'une seule session) : `# yane -k`

Sinon il faut récupérer l'id de la session que l'on désire stopper : `# yane -l` puis `# yane -s <ID> -k`


# Changer d'outil de simulation

   Imaginons que vous soyez un peu exigeant·e et que vous souhaitiez
utiliser un autre outil de virtualisation que les netns. Comment
pouvez-vous faire  ? C'est très simple, il vous suffit de changer le
mode de l'hôte concerné.

   À titre d'exemple, modifions le fichier yane.yml de la façon
suivante

```yaml
network:
  name: basic
  version: 1.0
  hosts:
    -
      name: host-a
      mode: docker
    -  
      name: host-b
      mode: docker
```

   Si nous lançons à nouveau une simulation, nous constatons que c'est
bien docker qui est utilisé et non plus les netns.

# Ajouter des fichiers sur un hôte

Si l'on souhaite démarrer un réseau avec plusieurs hôtes, il est
légitime de vouloir que chacun dispose d'une configuration
différente. Le nom fournit à la descritpion de l'hôte et les adresses
IP attribuées à la création des liens sont un premier outil.

   S'il n'est pas suffisant, des fichiers de configuration peuvent
être spécifiés qui seront copiés au sein d'un hôte avant son
démarrage.
