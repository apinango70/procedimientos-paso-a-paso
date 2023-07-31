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
```hash
wsl -l
```
borrar distribucion

wsl --unregister <distroName>


Listar distribuciones disponibles en linea para instalar
```hash
wsl -l -o
```

Instalar distribucion
```hash
wsl --install -d <Distribution Name>
```


Instalar ruby con versiones (rbenv)


# ACTUALZIA EL LISTADO DE PAQUETES
```hash
sudo apt update
```

# INSTALAR PAQUETES UTILIZADOS POR RUBY
```hash
sudo apt install git curl libssl-dev libreadline-dev zlib1g-dev autoconf bison build-essential libyaml-dev libreadline-dev libncurses5-dev libffi-dev libgdbm-dev
```

```hash
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash
```

```hash
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
```

```hash
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
```

```hash
source ~/.bashrc
```

```hash
type rbenv
```

# LISTAR VERSIONES DISPONIBLES PARA INSTALAR

```hash
rbenv install -l
```

rbenv install 3.1.2

# PARA IR VIENDO LOS PASOS DE LA INSTALACION EJECUTAR

tail -f /tmp/ruby-build.20230731124156.8124.log

# HABILITAR LA VERSION DE RUBY INSTALADA
```hash
rbenv global 3.1.2
```

```hash
ruby -v
```

# DESACTIVAR DOCUMENTACION AL INSTALAR GEMAS

```hash
echo "gem: --no-document" > ~/.gemrc
```

# INSTALAR PSQL
```hash
sudo apt install postgresql postgresql-contrib
```

#Instalar cabecera

```hash
sudo apt install libpq-dev
```


sudo service postgresql status # for checking the status of your database.
sudo service postgresql start # to start running your database.
sudo service postgresql stop # to stop running your database.

CREATE ROLE apinango WITH LOGIN SUPERUSER CREATEDB CREATEROLE PASSWORD ‘apinango’ ;



DESCONECTAR TODOS LOS USUARIOS DE LA DB

```hash
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE usename = 'apinango';
```
