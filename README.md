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

Pour pouvoir se connecter facilement aux différentes VMs du groupe

## Création d'un pool d'adresses de backend

Un pool d’adresses de back-ends contient les adresses IP des cartes d’interface réseau virtuelles connectées à l’équilibreur de charge.

## Création d'une sonde d’intégrité

## Création d'une règle d'équilibrage de charge

- Adresse IP frontend : celle du loadbalancer
- Port frontend : 80
- Port backend : 80
- *sans* persistance de session : chaque requête sera load-balancée en fonction des VMS disponibles. NB: ce réglage n'est pas adapté à des sites utilisant des sessions de connexion.

## Création de 3 VMS identiques

- Insertion des 3 VMS dans un "Groupe à haute disponibilité" afin d'assurer la redondance
- Ubuntu Server 18.04
- Authentification par Mdp, pas par clé publique ssh (plus simple pour un projet étudiant, déconseillé en prod)
- Ouverture port 22
- Aucune adresse IP publique, car la VM n'est accessible qui depuis le load balancer, pas depuis le monde
- Insertion de la VM dans le sous-réseau déjà créé, afin de l'isoler du reste du réseau virtuel
- Création d'un unique Groupe de sécurité réseau, appliqué aux 3 VMs. Afin d'avoir les mêmes règles de pare-feu pour toutes les VMs.
    - Autorisation du trafic entrant sur le port 80 des VMs du groupe
- Récupération d'un fichier de "code déclaratif" (format JSON) pour créer les autres VMs depuis la console (Azure Resource Manager)


