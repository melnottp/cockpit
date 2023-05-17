# Connection à la base RDS postgreSQL
psql -h postgre.tooling.services -p 8635 -U root -d postgres

# Creation de la base et user superset
create database superset;
create user superset;
alter role superset with password 'Super$et22!';
grant all privileges on database superset to superset;
alter database superset owner to superset;

# Backup de la base postgre de prod :
pg_dump -U superset -h postgre.tooling.services -p 8635 > superset-310323.sql

# Restore du backup de la base superset sur la nouvelle base RDS
psql -h postgre.tooling.services -p 8635-d superset -U superset -W -f superset.sql

# upgrade helm chart 
helm upgrade --install --values my-values-rds.yaml superset superset/superset

# Clean le namespace des pod evicted :
kubectl get pod | grep Evicted | awk '{print $1}' | xargs kubectl delete pod 
kubectl -n dev get pod | grep Evicted | awk '{print $1}' | xargs kubectl -n dev delete pod 

