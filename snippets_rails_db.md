# Manipulacion de campos en bd

## Agregar nuevo campo a modelo existente

```bash
rails g migration AddPhoneToUsers phone:string
```

## Agregar varios campos a modelo  existente

```bash
rails g migration AddFieldsToUsers phone:string website:string description:text 
```

##  Agregar campo id para realcion 1:n o n:n

```bash
rails g migration AddAuthorRefToBooks author:references
```

## Cambiar nombre de campo

```bash
rails generate migration RenameEdadToAgeInUsers
```

## Agregar campo role a devise

```bash
rails g devise User role:integer
```

## Agregar a la migración el valor por defecto de rol 0:user

```bash
t.integer :role, default:0
```

## Ejecutar la migración

```bash
rails db:migrate
```
