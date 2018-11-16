## cassandra on k8s


## Create StatefulSet and Service:
```
# kubectl apply -f cassandra-svc.yaml
# kubectl apply -f cassandra-app.yaml
```

## Listing StatefulSet and Service:
```
# kubectl get statefulset
# kubectl get pods
# kubectl get svc
```

## Listem Status Cassandra Nodes:
```
# kubectl exec cassandra-0 -- nodetool status
# kubectl get pods -l app=cassandra -o json | jq '.items[] | {"name": .metadata.name,"hostname": .spec.nodeName, "hostIP": .status.hostIP, "PodIP": .status.podIP}'
# kubectl exec -it cassandra-0 -- nodetool getendpoints classicmodels offices 6
```

## Now that we are inside the shell, we can create a keyspace and populate it:
```
# kubectl exec -it cassandra-0 -- cqlsh


CREATE KEYSPACE classicmodels WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 3 };
	
CONSISTENCY QUORUM;
Consistency level set to QUORUM.

use classicmodels;

CREATE TABLE offices (officeCode text PRIMARY KEY, city text, phone text, addressLine1 text, addressLine2 text, state text, country text, postalCode text, territory text);

INSERT into offices(officeCode, city, phone, addressLine1, addressLine2, state, country ,postalCode, territory) values 
	'1','San Francisco','+1 650 219 4782','100 Market Street','Suite 300','CA','USA','94080','NA');

INSERT into offices(officeCode, city, phone, addressLine1, addressLine2, state, country ,postalCode, territory) values 
	'2','Boston','+1 215 837 0825','1550 Court Place','Suite 102','MA','USA','02107','NA');

INSERT into offices(officeCode, city, phone, addressLine1, addressLine2, state, country ,postalCode, territory) values 	
	'3','NYC','+1 212 555 3000','523 East 53rd Street','apt. 5A','NY','USA','10022','NA');

INSERT into offices(officeCode, city, phone, addressLine1, addressLine2, state, country ,postalCode, territory) values
	'4','Paris','+33 14 723 4404','43 Rue Jouffroy abbans', NULL ,NULL,'France','75017','EMEA');

INSERT into offices(officeCode, city, phone, addressLine1, addressLine2, state, country ,postalCode, territory) values 		
	'5','Tokyo','+81 33 224 5000','4-1 Kioicho',NULL,'Chiyoda-Ku','Japan','102-8578','Japan');

INSERT into offices(officeCode, city, phone, addressLine1, addressLine2, state, country ,postalCode, territory) values 
	'6','Sydney','+61 2 9264 2451','5-11 Wentworth Avenue','Floor #2','Teste','Australia','NSW 2010','APAC');

INSERT into offices(officeCode, city, phone, addressLine1, addressLine2, state, country ,postalCode, territory) values 
	'7','London','+44 20 7877 2041','25 Old Broad Street','Level 7',NULL,'UK','EC2N 1HN','EMEA');

INSERT into offices(officeCode, city, phone, addressLine1, addressLine2, state, country ,postalCode, territory) values 
	'8','Mumbai','+91 22 8765434','BKC','Building 2',NULL,'MH','400051','APAC');


SELECT * FROM classicmodels.offices;
```

## Simulating node failure:
```
# NODE=`kubectl get pods cassandra-0 -o json | jq -r .spec.nodeName`
# kubectl cordon ${NODE}
# kubectl get nodes

# kubectl delete pod cassandra-0
# kubectl get pods -o wide

# kubectl exec cassandra-0 -- cqlsh -e 'select * from classicmodels.offices'
# kubectl exec cassandra-1 -- nodetool status
```

## Recovering node:
```
# kubectl uncordon ${NODE}
# kubectl delete pod cassandra-0
# kubectl get pods -o wide

# kubectl exec cassandra-0 -- cqlsh -e 'select * from classicmodels.offices'
# kubectl exec cassandra-1 -- nodetool status
```