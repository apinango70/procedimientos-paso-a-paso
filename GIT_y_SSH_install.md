# Instrucciones para instalar Git en Ubuntu y conectar Git con Github.com

## Actualizar el núcleo de Ubuntu

```hash
sudo add-apt-repository ppa:git-core/ppa 
```

## Instala Git

```hash 
sudo apt-get update && sudo apt-get -y install git 
```
## Configura tu nombre de usuario y dirección de correo electrónico:

```hash
git config --global user.name "first_name last_name" 
```
```hash
git config --global user.email "email" 
```

## Para verificar si los datos se registraron correctamente, ejecutar: 

## en linux

```hash
cat .gitconfig 
```

## en windows
## _Conectar Git con GitHub.com_

```hash
git config --list
```
## Genera una clave SSH:

```hash
ssh-keygen -t rsa -b 4096 -C "email"
```

## Agrega la clave SSH a tu agente SSH

```hash
ssh-add ~/.ssh/id_rsa
```

## Si te da error de Agente SSH no iniciado, ejecutar:

```hash
eval "$(ssh-agent -s)"
```

## Ejecutar de nuevo:

```hash
ssh-add ~/.ssh/id_rsa
```

## Verificar la lista de claves SSH agregadas

```hash
ssh-add -l
```

## _Agrega la clave SSH a tu cuenta de GitHub.com_

## Mostrar la clave SSH generada, esta clave se debe copiar en github.com

```hash
cat ~/.ssh/id_rsa.pub
```

## Copiar bloque desde ssh-rsa ... hasta el final del email.

## Confirma que la conexión funciona

```hash
ssh -T git@github.com
```

## Si al hacer push github.com aún pide user y pass, ejecutar:

```hash
git remote set-url origin git@github.com:gihub_user_name/nombre_repositorio.git
```

## Intentar hacer de nuevo push.
