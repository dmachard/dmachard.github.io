---
title: "Dashboard Kibana pour le Credit Agricole"
date: 2020-06-21T00:00:00+01:00
draft: false
tags: ['dashbaord', 'banque', 'kibana']
---

Procédure de mise en oeuvre d'un dashboard Kibana permettant de suivre les opérations bancaires du crédit agricole.

# Sommaire

* [ConfGuideiguration](#guide)

## Guide

Déployer le fichier de configuration `00-creditagricole.conf` logstash dans /etc/logstash/conf.d/

- La section `input` utilise le plugin `Exec` pour exécuter les 2 scripts pythons une fois par jour.
- La section `filter` catégorise les différentes opérations avec le plugin `Translate`
- Enfin la section `output` injecte les opérations dans ElasticSearch

**logstash-creditagricole.conf**

```bash

input {
  exec {
    type => "pull_operations"
    command => "python3 /home/pull_compte.py"
    #interval => 3000
    schedule => "50 23 * * * Europe/Paris"
  }

   exec {
    type => "pull_accounts"
    command => "python3 /home/pull_accounts.py"
    #interval => 3000
    schedule => "50 23 * * * Europe/Paris"
  }

}

filter {
  if [type] == "pull_accounts" {
	json {
            source => "message"
            target => "account"
        }
  }

  if [type] == "pull_operations" {
	  json {
	    source => "message"
	    target => "operation"
	  }
  
	  mutate { remove_field => [ "message"] }
	  mutate { remove_field => [ "command"] }

	  split { field => "[operation]" }
  
	  date {
	     match => [ "[operation][dateOperation]" , "MMM dd, yyyy hh:mm:ss a" ]
	     target => "@timestamp"
	  }

	  if [operation][montant] < 0 {
	    translate {
	      field       => "[operation][libelleOperation]"
	      destination => "[categorie]"
	      dictionary  => {
                            "ECHEANCE" => "PRETS"		

                            "PHARMACIE" => "SANTE"
                            
                            "BOULANGERIE" => "COURSES"
                            
                            "SNCF" => "TRANSPORT"

                            "ELECTRICITE" => "ENERGIE/EAU/TELECOM"
                            
                            "SARENZA" => "SHOPPING"

                            "LEROY MERLIN" => "LOGEMENT"

                            "BURGER KING" => "RESTAURATION"

                            "MAIF" => "ASSURANCE/BANQUE"

                            "UGC" => "LOISIRS"
		     }
	      exact       => true
	      regex       => true
	      fallback => "NON CLASSE"
	   }
	  } else {
	
	   translate {
	      field       => "[operation][libelleOperation]"
	      destination => "[categorie]"
	      dictionary  => {
                            "PAYE" => "SALAIRE"
		    }
      		exact       => true
      		regex       => true
      		fallback => "NON CLASSE"
   	}
     }
   }


}

output {
  if [type] == "pull_operations" {
 	 elasticsearch {
 	   hosts => ["http://localhost:9200"]
 	   index => "compte_operations"
  	}
  }
  if [type] == "pull_accounts" {
         elasticsearch {
           hosts => ["http://localhost:9200"]
           index => "compte_accounts"
         }
  }
}
```
.
**pull_accounts.py**

```python
from creditagricole_particuliers import Authenticator, Accounts

import json
import time

def get_accounts():
	try:
		session = Authenticator(username="xxxxxxxxxxxxxxxx",
		                        password=[1, 2, 3, 4, 5, 6])
                        
		accounts = Accounts(session=session)

		
		ret = {"epargne": 0}
		for acc in accounts:
			if acc["accountNatureShortLabel"] == "HABITATION":
				ret["solde"] = int(acc["balanceValue"])
			
			ret["epargne"] += int(acc["balanceValue"])

		print( json.dumps(ret) )
	except Exception:
		return False
	return True


while True:
	success = get_accounts()
	if success:
		break
	else:
		time.sleep(60)


```

**pull_compte.py**


```python
from creditagricole_particuliers import Authenticator, Operations

from datetime import date
import json
import time
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--begin", help="date", type=str)
parser.add_argument("--end", help="date", type=str)
args = parser.parse_args()

def get_operations():
	try:
        # init authenticator with account number and password
		session = Authenticator(username="xxxxxxxxxxxxxxxx",
		                        password=[1, 2, 3, 4, 5, 6])
        
        # get the date of the date
		today = date.today()
		date_start = today.strftime("%Y-%m-%d")
		date_fin = today.strftime("%Y-%m-%d")

        # otherwise check if the date is 
        # provided from the command line
        if args.begin is not None:
                date_start = args.begin
        if args.end is not None:
                date_fin = args.end
        
        # get operations according to the time interval
		operations = Operations(session=session,
		                        date_start=date_start,
		                        date_stop=date_fin)

        # print results
		print( json.dumps(operations.list) )
	except Exception:
		return False
	return True


# get operations 
# repeat every 60 seconds until success
while True:
	success = get_operations()
	if success:
		break
	else:
		time.sleep(60)

```