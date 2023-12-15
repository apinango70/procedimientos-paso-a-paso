# Crear el MVC de devise para que el role admin pueda hacer CRUD con los users

> [!IMPORTANT]
> Para realizar este procedimiento, se debe tener instaldo y configurado:

>Devise gem https://github.com/apinango70/procedimientos-paso-a-paso/blob/main/devise_1_install_gem.md

>Active storage gem https://github.com/apinango70/procedimientos-paso-a-paso/blob/main/active_storage_gem_Install.md

## Generar el controlador para administrar a los usuarios

> [!IMPORTANT]
> Se debe generar un controlador para administrar a los usuarios

´´´bash
rails generate controller Admin::Users
´´´

## Configuración de rutas

> [!IMPORTANT]
>En el archivo routes.rb se deben definir las rutas para la administración de los usuarios bajo un namespace admin, así podemos diferenciarlas de las rutas que usa devise por defecto. Para ello se debe agregar: 


´´´ruby
devise_for :users
namespace :admin do
  resources :users
end
´´´

## Crear 10 users de prueba con faker app/db/seed.rb

```ruby
# Crea 10 usuarios con datos ficticios
10.times do
    User.create(
      email: Faker::Internet.email,
      password: Faker::Internet.password,
      first_name: Faker::Name.first_name,
      last_name: Faker::Name.last_name,
      role: "user"
    )
  end

  puts "Seed data generated successfully!"
```

## Hago commit

```ruby
git add .
git commit -m "style: agregar bootstrap a las vistas y form del admin CRUD"
```
