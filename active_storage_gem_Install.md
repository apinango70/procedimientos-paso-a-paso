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

## Proyecto de prueba:
```hash
rails new adjuntos -d postgresql
```

```hash
cd adjuntos
rails db:create
```

```hash
git add .
git commit -m "creacion inicial del proyecto"
```

```hash
rails g scaffold article title body:text
rails db:migrate
```

```hash
git add .
git commit -m "Scaffold creado"
```
## Habilitar la ruta principal en config>routes>routes.rb

	_root "articles#index"_

## Instalación gema active_storage:

```hash
rails active_storage:install
rails db:migrate
```

## Hago commit

```hash
git add .
git  commit -m "Active storage instalado."
```

## Descomentar en el Gemfile en la línea 49

```hash
gem "image_processing", ">= 1.2"
```

## Ejecutar bundle

```hash
bundle install
```

## Copiar en el modelo que estemos usando app>model>modelo.rb, modelo del ejemplo "article" app>model>article.rb y para integrarlo al modelo user app>model>user.rb.


class Article < ApplicationRecord

```hash
  has_one_attached :photo
```

end

## para agregarlo a un formulario, en este caso app>views>articles>_form.html.erb:

```hash
<div>
  <%= form.label :photo, style: "display: block" %>
  <%= form.file_field :photo %>
</div>
```

Para agregarlo al modelo user:

```hash
<div>
  <%= f.label :photo, style: "display: block" %>
  <%= f.file_field :photo %>
</div>
```

## Agregar al controller el strong parameter 

### Modelo User app>users>registration_controller.rb
```hash
   protected

    #If you have extra params to permit, append them to the sanitizer.
    def configure_sign_up_params
      devise_parameter_sanitizer.permit(:sign_up, keys: [:username, :role, :photo])
    end
  
    #If you have extra params to permit, append them to the sanitizer.
    def configure_account_update_params
      devise_parameter_sanitizer.permit(:account_update, keys: [:username, :role, :photo])
    end
```

### Modelo Article app>controller>articles_controller.rb

```hash
  private
    def article_params
      params.require(:article).permit(:title, :body, :photo)
    end
```

### Mostrar la imagen en el formulario show al crear el registro

### Modelo article app>views>articles>_article.html.erb

```hash
  <p>
    <strong>Foto:</strong>
    <%= image_tag article.photo, style: "width: 50px" if article.photo.attached? %>
  </p>
```
