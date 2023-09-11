# Configuración gema active_storage

## Instalaciones previas.

### En el termina ejecutar:

```hash
sudo apt update
sudo apt-get -y install libvips-dev
sudo apt update
sudo apt install ffmpeg
sudo apt update
sudo apt-get install libpoppler-dev
sudo apt update
sudo apt-get -y install mupdf
sudo apt update
```

## Instalación gema active_storage:

```hash
rails active_storage:install
rails db:migrate
```

## Hago commit

```hash
git  commit -m "Active storage instalado."
```

## Descomentar en el Gemfile

```hash
gem "image_processing", ">= 1.2"
```

## Ejecutar bundle
























