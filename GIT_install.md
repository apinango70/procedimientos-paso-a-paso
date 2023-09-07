## Instrucciones para instalar Git en Ubuntu y conectar Git con Github.com

# Actualizar el núcleo de Ubuntu

´´´bash
sudo add-apt-repository ppa:git-core/ppa 
´´´
sudo apt-get update && sudo apt-get -y install git 

git config --global user.name "first_name last_name" 

git config --global user.email "email" 

cat .gitconfig 

CONECTAR GIT CON GITHUB.COM

ssh-keygen -t rsa -b 4096 -C "email"

ssh-add ~/.ssh/id_rsa

Iniciar el agente SSH

eval "$(ssh-agent -s)"

Agregar la clave SSH

ssh-add ~/.ssh/id_rsa

Verificar la lista de claves SSH agregadas

ssh-add -l

Agrega la clave SSH a tu cuenta de GitHu

cat ~/.ssh/id_rsa.pub

Confirma que la conexión funciona

ssh -T git@github.com

Si pide user y pass al hacer push ejecutar:

git remote set-url origin git@github.com:gihub_user_name/nombre_repositorio

