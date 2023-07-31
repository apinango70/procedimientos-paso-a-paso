colocar un recuadro al rededor del div al colocar cursor encima
```hash
div:hover{
    outline: 2px  solid red; 
}

centra div en pantalla

display:grid;
place-items:center;
```






Listar distribuciones

wsl -l

borrar distribucion

wsl --unregister <distroName>


Listar distribuciones disponibles en linea para instalar

wsl -l -o


Instalar distribucion

wsl --install -d <Distribution Name>


Instalar ruby con versiones (rbenv)


# ACTUALZIA EL LISTADO DE PAQUETES
sudo apt update

# INSTALAR PAQUETES UTILIZADOS POR RUBY
sudo apt install git curl libssl-dev libreadline-dev zlib1g-dev autoconf bison build-essential libyaml-dev libreadline-dev libncurses5-dev libffi-dev libgdbm-dev

curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash

echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc

echo 'eval "$(rbenv init -)"' >> ~/.bashrc

source ~/.bashrc

type rbenv

# LISTAR VERSIONES DISPONIBLES PARA INSTALAR

rbenv install -l

rbenv install 3.1.2

# PARA IR VIENDO LOS PASOS DE LA INSTALACION EJECUTAR

tail -f /tmp/ruby-build.20230731124156.8124.log

# HABILITAR LA VERSION DE RUBY INSTALADA

rbenv global 3.1.2

ruby -v

# DESACTIVAR DOCUMENTACION AL INSTALAR GEMAS

echo "gem: --no-document" > ~/.gemrc

# INSTALAR PSQL

sudo apt install postgresql postgresql-contrib

#Instalar cabecera

sudo apt install libpq-dev


sudo service postgresql status # for checking the status of your database.
sudo service postgresql start # to start running your database.
sudo service postgresql stop # to stop running your database.

CREATE ROLE apinango WITH LOGIN SUPERUSER CREATEDB CREATEROLE PASSWORD ‘apinango’ ;



DESCONECTAR TODOS LOS USUARIOS DE LA DB

SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE usename = 'apinango';
