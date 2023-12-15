# Crear el MVC de devise para que el role admin pueda hacer CRUD con los users

> [!IMPORTANT]
> Para realizar este procedimiento, se debe tener instaldo y configurado:

>Devise gem https://github.com/apinango70/procedimientos-paso-a-paso/blob/main/devise_1_install_gem.md

>Active storage gem https://github.com/apinango70/procedimientos-paso-a-paso/blob/main/active_storage_gem_Install.md

## Generar el controlador para administrar a los usuarios

Se debe generar un controlador para administrar a los usuarios

```ruby
rails generate controller Admin::Users
```

## Configuración de rutas

En el archivo routes.rb se deben definir las rutas para la administración de los usuarios bajo un namespace admin, así podemos diferenciarlas de las rutas que usa devise por defecto. Para ello se debe agregar: 

```ruby
devise_for :users
namespace :admin do
  resources :users
end
```

## Definir las acciones del controlador

En el _admin::users_controller_ se deben implementar las acciones CRUD y definir los strong parameters.

```ruby
   class Admin::UsersController < ApplicationController
     before_action :authenticate_admin! # Asegura que solo los user con rol de admin puedan acceder al CRUD de los usuarios
     before_action :set_user, only: [:show, :edit, :update, :destroy] # Permite tener el usuario seleccionado disponible en @user
   
  def index
    @users = User.all
  end

  def show
    @user = User.find(params[:id])
  end

  def edit
    @user = User.find(params[:id])
  end

  def update
    @user = User.find(params[:id])
    if @user.update(user_params)
      redirect_to admin_users_path(@user), notice: 'User was successfully updated.'
    else
      redirect_to admin_users_path, alert: 'User was not updated.'
    end
  end

  def destroy
    @user = User.find(params[:id])
    @user.destroy
    redirect_to admin_users_path, notice: 'User was successfully deleted.'
  end

   def new
    @user = User.new
  end

  def create
    @user = User.new(user_params)
    if @user.save
      redirect_to admin_users_path, notice: 'User was successfully created..'
    else
      render :new
    end
  end
     
  private
 
  def authenticate_admin! # Método que verifica si el usuario que ingresó a admin/users/ tiene el rol de admin, de lo contrario muestra el alert y lo devuleve al root_path
    unless current_user && current_user.admin?
      redirect_to root_path, alert: 'Access Denied.!'
    end
  end

  def set_user
    @user = User.find(params[:id]) # Método que permite encontrar y asignar el usuario correcto según el ID proporcionado
  end

  def user_params # Strong Parameters permitidos
    params.require(:user).permit(
                                  :first_name,
                                  :last_name,
                                  :email,
                                  :role,
                                  :photo,
                                  :password,
                                  :password_confirmation
                                  )
  end

end
```

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
