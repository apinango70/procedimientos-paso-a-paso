# INSTALAR PSQL

```bash
sudo apt install -y postgresql postgresql-contrib
```

## Instalar cabecera

```bash
sudo apt install -y libpq-dev
```

## INICIO AUTOMATICO DE POSTGRESQL

```bash
sudo service postgresql enable
```

## for checking the status of your database.

```bash
sudo service postgresql status
```

## to start running your database.

```bash
sudo service postgresql start
```

## to stop running your database.

```bash
sudo service postgresql stop
```

## Ejecutar servicio con user postgres

```bash
sudo -u postgres psql
```

```bash
CREATE ROLE apinango WITH LOGIN SUPERUSER CREATEDB CREATEROLE PASSWORD 'apinango' ;
```

