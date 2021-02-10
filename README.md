# Projet

Mise en place un load balancer dans Azur, avec un simple site sous python/flask tournant sur plusieurs VM

# Etapes

Tuto suivi ici : <https://docs.microsoft.com/fr-fr/azure/load-balancer/quickstart-load-balancer-standard-public-portal?tabs=option-1-create-load-balancer-basic>

## Création d'un groupe de ressource dédié

## Création du load-balancer

Affectation dynamique de l'IP du load-balancer, c'est à dire que l'IP publique est créée et affectée au load balancer uniquement lorsqu'on démarre la VM. Elle peut changer.

## Création d'un réseau virtuel

Plage d'adresses IP privées choisies :

- `10.1.0.0/16` = de `10.1.0.0` à `10.1.255.255` = soit 65536 adresses IP disponibles, soit potentiellement 65536 VM connectées au load balancer

Sous-réseau créé également, avec la plage `10.1.0.0/24`, soit 255 adresses disponibles.

## Création d'un Bastion

Pour pouvoir se connecter facilement aux différentes VMs du groupe depuis le board d'Azur.

## Création d'un pool d'adresses de backend

Un pool d’adresses de back-ends contient les adresses IP des cartes d’interface réseau virtuelles connectées à l’équilibreur de charge.

## Création d'une sonde d’intégrité

La sonde va aller vérifier toutes les 15 minutes que les VMs du backend sont fonctionnelles. Si ce n'est pas le cas, le load balancer exclut la VM défaillante du pool backend.

## Création d'une règle d'équilibrage de charge

C'est cete règle qui fait le load balancing en tant que tel.

- Adresse IP frontend : celle du loadbalancer
- Port frontend : 80
- Port backend : 80
- *sans* persistance de session : chaque requête sera load-balancée en fonction des VMS disponibles. NB: ce réglage n'est pas adapté à des sites utilisant des sessions de connexion.

## Création de 3 VMS identiques

Avec les parametres suivants :

- Insertion des 3 VMS dans un "Groupe à haute disponibilité" afin d'assurer la redondance
- OS : Ubuntu Server 18.04
- Authentification par Mdp (déconseillé en prod)
- Ouverture port 22
- Aucune adresse IP publique, car la VM n'est accessible qui depuis le load balancer, pas depuis le monde. Elle sera identifiée par son adresse privée, parmis les adresses du sous-réseau déjà créé (`10.1.0.0/24`).
- Insertion de la VM dans le sous-réseau déjà créé (`10.1.0.0/24`), afin de l'isoler du reste du réseau virtuel
- Création d'un unique Groupe de sécurité réseau, appliqué aux 3 VMs. Afin d'avoir les mêmes règles de pare-feu pour toutes les VMs.
    - Autorisation du trafic entrant sur le port 80 des VMs du groupe
- Récupération d'un fichier de "code déclaratif" (format JSON) pour créer les autres VMs depuis la console (Azure Resource Manager) : `./deploy/template.json`

### Déploiement avec script des autres VMs

Déploiement de la VM2 avec le fichier `./deploy/template.json` :

```
az deployment group create \
    --resource-group CreatePubLBQS-rg \
    --template-uri "https://raw.githubusercontent.com/vittorio-c/azur-load-balancer-ipssi/master/deploy/template.json" \
    --name DeployVM2 \
    --parameters @.parameters.vm2.json
```

Pour examiner les changements avant de les appliquer :

```
az deployment group what-if \
    --resource-group CreatePubLBQS-rg \
    --template-uri "https://raw.githubusercontent.com/vittorio-c/azur-load-balancer-ipssi/master/deploy/template.json" \
    --name DeployVM2 \
    --parameters @.parameters.vm2.json
```

Déploiement de la VM3 :

```
az deployment group create \
    --resource-group CreatePubLBQS-rg \
    --template-uri "https://raw.githubusercontent.com/vittorio-c/azur-load-balancer-ipssi/master/deploy/template.json" \
    --name DeployVM3 \
    --parameters @.parameters.vm3.json
```

(les fichiers de parametres sont exclus de ce repo, car contenant des mots de passe de connexion ssh aux VMs)

Pour vérifier que les VMs sont bien up, se connecter avec Bastion depuis le portal d'Azur, avec mdp et username définis, et faire un petit `htop`.

## Rattachement des VMs au pool backend

Afin de rendre accessible les 3 VMs au load balancer, il faut les rattacher au pool de backend défini.

## Installation de Nginx sur les différentes VMs

Commande à lancer 3 fois, en changeant le `vm-name` :

```
az vm extension set \
  --resource-group CreatePubLBQS-rg \
  --vm-name myVM1 \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --version 2.1 \
  --settings '{"fileUris":["https://raw.githubusercontent.com/MicrosoftDocs/mslearn-welcome-to-azure/master/configure-nginx.sh"]}' \
  --protected-settings '{"commandToExecute": "./configure-nginx.sh"}'
```

Le script situé à l'adresse "https://raw.githubusercontent.com/MicrosoftDocs/mslearn-welcome-to-azure/master/configure-nginx.sh" définit la page d’accueil, `/var/www/html/index.html`, pour afficher un message d’accueil comprenant le nom d’hôte de la VM.

Ainsi, on pourra vérifier, dans un premier temps, que le load balancer fonctionne bien.

## Vérification du bon fonctionnement du loadbalancer

Afin de vérifier que le loadbalancer fonctionne, aller depuis un navigateur sur l'IP publique du load-balancer (`http://20.188.60.215/`) et observer les messages :

- "Welcome to Azure! My name is myVM1."
- "Welcome to Azure! My name is myVM2."
- "Welcome to Azure! My name is myVM3."

Le message doit changer en fonction des requêtes. Il faut désactiver le cache du navigateur et/ou appuyer sur Ctr+F5 pour forcer l'actualisation, et donc simuler la répartition des requêtes entre les différentes VMs par le loadbalancer.

## Déploiement d'une application Python Flask

TODO
