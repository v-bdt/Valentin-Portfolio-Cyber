
> [!TIP]
> Permet d'identifier les chemins d'attaque.


> [!attention]
> Une fois l'accès obtenu à un hôte faisant parti du domaine, lancer sharphound depuis celui ci permet d'obtenir bien plus de données qu'avec la collecte en remote avec netexec...

# 1. COLLECTE DE DONNEES (Sharphound)

Méthodes de collecte possibles:

https://github.com/dirkjanm/BloodHound.py/blob/master/README.md#usage

# LINUX
## *Netexec*

Collecter toutes les données

```bash
nxc ldap <ip> -u user -p pass -d <domain> --kdcHost DANTE-DC02 --dns-server 172.16.2.5 --bloodhound --collection All
```

Alternative avec bloodhoud.py

```bash
bloodhound-python -c All -d '<domain>' -u 'john.doe' -p 'P@$$word123!' -ns 10.80.80.2
```

Une fois les données collectées, les importer dans bloodhound GUI

```bash
sudo neo4j start
```

```bash
bloodhound
```

# WINDOWS

https://github.com/SpecterOps/SharpHound/releases/tag/v2.6.3


```powershell
.\SharpHound.exe --help
```

Collecter toutes les données

```powershell
.\SharpHound.exe -c All --zipfilename ILFREIGHT
```

Collecter toutes les données en précisant des credentials

```
.\SharpHound.exe -c All --zipfilename zsm.local --ldapusername jamie --ldappassword P@ssw0rd
```



---

# 2. ANALYSE DE DONNEES (Bloodhound GUI)


> [!ATTENTION]
> Penser a ajouter tous les objets owned (Users/Computers/Groups...)
> Relancer la query "Shortest Path from Owned objects" à chaque nouvel objet owned ajouté

## 2.3. ACLs

Sélectionner un User > Node Info > Outbound Control Rights > Transitive Object Control

## 2.4. Trust Relationships

Analysis > Pre-Built Analytics Queries > Domain Information > Map Domain Trusts


## 2.5 CYPHER QUERIES

https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/


### Trouver les users qui peuvent PSRemote / RDP

```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```

```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanRDP*1..]->(c:Computer) RETURN p2
```

### Trouver SQLAdmin

```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```


### Find all the Edges that any UNPRIVILEGED user (based on the admincount:False) has against all the nodes:    

```Cypher
MATCH (n:User {admincount:False}) MATCH (m) WHERE NOT m.name = n.name MATCH p=allShortestPaths((n)-[r:MemberOf|HasSession|AdminTo|AllExtendedRights|AddMember|ForceChangePassword|GenericAll|GenericWrite|Owns|WriteDacl|WriteOwner|CanRDP|ExecuteDCOM|AllowedToDelegate|ReadLAPSPassword|Contains|GpLink|AddAllowedToAct|AllowedToAct|SQLAdmin*1..]->(m)) RETURN p
```