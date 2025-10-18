# Instalacion WSL en PowerShell de windows

## Listar distribuciones instaladas

```bash
wsl -l
```

## borrar distribucion

```bash
wsl --unregister <distroName>
```

## Listar distribuciones disponibles en linea para instalar

```bash
wsl -l -o
```

## Instalar distribucion

```bash
wsl --install -d <Distribution Name>
```

## Agregar permisos de escritura sin ser root

```bash
sudo chown -R apinango:apinango /home/apinango/rails
```



# INSTALAR RUBY CON RBENV

## Paso 1: Instalar dependencias

```bash
sudo apt install -y git-core build-essential curl wget openssl libssl-dev libreadline-dev dirmngr zlib1g-dev libmagickwand-dev imagemagick-6.q16 libffi-dev libpq-dev cmake libwebp-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev software-properties-common libcurl4-openssl-dev
```

## Paso 2: Instalar rbenv

```bash
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
```

## Paso 3: Configurar rbenv

```bash
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc
```

## Paso 4: Instalar ruby-build

```bash
git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
```

## Paso 5: Configurar ruby-build

```bash
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## Paso 6: Verificar versiones disponibles de Ruby

```bash
rbenv install --list-all
```

### Puedes verificar las versiones estables de Ruby ejecutando el siguiente comando:

```bash
rbenv install --list
```

## Paso 7: Instalar una versión de Ruby

```bash
rbenv install 3.2.2
```

## Paso 8: Establecer la versión global de Ruby

```bash
rbenv global 3.2.2
```

## Paso 9: Verificar la versión de Ruby

```bash
ruby -v
```

## Paso 10: Instalar gema Bundler

```bash
gem install bundler
```

## Paso 11: Configurar Bundler para GitHub

```bash
bundle config github.https true
```

## Paso 12: Instalar Rails

### Instalar una versión específica

```bash
gem install rails -v 7.0.8
```

### Instalar la versión más reciente de rails

```bash
gem install rails
```

## DESACTIVAR DOCUMENTACION AL INSTALAR GEMAS

```bash
echo "gem: --no-document" > ~/.gemrc
```


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



