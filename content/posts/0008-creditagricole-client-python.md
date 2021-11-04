---
title: "Client en Python pour la banque Crédit Agricole"
date: 2021-09-27T00:00:00+01:00
draft: false
tags: ['python', 'client']
---

Le [client Python](https://github.com/dmachard/creditagricole-particuliers) est à destination des particuliers souhaitant récupérer ses opérations bancaires stockées par le Crédit Agricole.

Il permet de:
- lister l'ensemble des comptes bancaires associés au compte
- récupérer le solde total
- récupérer la liste des opérations au format JSON

# Installation

Le module python `creditagricole-particuliers` disponible sur pypi


```bash
pip install creditagricole-particuliers
```

# Exemple

Code pour calculer le solde total de l'ensemble des comptes.

```python
from creditagricole_particuliers import Authenticator, Accounts

# user credentials
login = "xxxxxxx"
pwd = [x, x, x, x, x, x]
region = "normandie"

# make auth
session = Authenticator(username=login, password=pwd, region=region)

# get all accounts
accounts = Accounts(session=session)

print("")
total_solde = 0
for acc in accounts:
    total_solde += acc.get_solde()
    print(f'{acc.descr["libelleProduit"]:>25}  {acc.get_solde():>20}')

print("")
print( f"{'TOTAL':>25} {total_solde:>20}" )
```