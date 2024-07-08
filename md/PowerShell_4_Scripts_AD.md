---
title: Scripts PowerShell
description: Scripts típicos de PowerShell para la gestión de usuarios y grupos en dominios Windows
permalink: /PowerShell_4_Scripts_AD/
---

<h1>PowerShell. Ejemplos Scripts típicos</h1>

<h3>Tabla de contenidos</h3>

- [1. Scripts de creación de recursos de empresas](#1-scripts-de-creación-de-recursos-de-empresas)
  - [1.1. Ejemplo 1: Importar un fichero CSV](#11-ejemplo-1-importar-un-fichero-csv)
  - [1.2. Ejemplo 2: Creación de unidades organizativas de un fichero](#12-ejemplo-2-creación-de-unidades-organizativas-de-un-fichero)
  - [1.3. Ejemplo 3: Creación de grupos de un fichero](#13-ejemplo-3-creación-de-grupos-de-un-fichero)
  - [1.4. Ejemplo 4: Creación de usuarios](#14-ejemplo-4-creación-de-usuarios)



# 1. Scripts de creación de recursos de empresas
En los siguiente ejemplos se ve el típico ejercicio de creación de unidades, grupo y usuarios con PowerShell

## 1.1. Ejemplo 1: Importar un fichero CSV
Para la importación de un fichero CSV, es necesaria la función **import-csv** y el fichero tiene que tener un formato especificado con los títulos de los campos en la primera línea (y puntos en la segunda ????)
```powershell
$nombres = import-csv .\archivosCsv\nombres.csv
clear
forEach ( $usuario in $nombres){
    echo $usuario
    echo $usuario.nombre = "adfa "
    echo $usuario.streataddress = "Calle Merluza"
    echo $usuario.city = "Ciudad"
}
```

## 1.2. Ejemplo 2: Creación de unidades organizativas de un fichero 

Ojo con los archivos csv, que la primera líneas no vale, contiene los encabezados que indican el nombre de los campos.

Ejemplo : leerCsv.ps1

```powershell
$nombres = import-csv .\archivoscsv\unidades_org.csv
# el formato es:
# nombre, direccion, descripcion
forEach ( $unidad in $nombres){
    Write-Host "Creando unidad : " + $unidad.nombre + " con descripcion " + $unidad.descripcion
    new-ADOrganizationalUnit -name $unidad.nombre -Path "dc=dominio,dc=curso" -description $unidad.descripcion
    echo Creado
}
```

## 1.3. Ejemplo 3: Creación de grupos de un fichero 
```powershell
$nombres = import-csv .\archivoscsv\departamentos.csv
forEach ( $grupo in $nombres){
    Write-Host ( "Creando grupo : " + $unidad.nombre + " en " + $unidad.ubicacion)
    $ruta = "ou=" + $grupo.ubicacion + ",dc=dominio,dc=lan"
    new-ADGroup -name $unidad.nombre -GroupCategory security -GroupScope Global  -Path "dc=dominio,dc=curso" -description $unidad.descripcion $ruta
    echo Creado
}
```

## 1.4. Ejemplo 4: Creación de usuarios 
no se puede crear un usuario añadiéndolo a un grupo, por eso se hace después en otra línea
```powershell
$nombres = import-csv .\archivoscsv\departamentos.csv
forEach ( $usuario in $nombres){
    echo ("Creando usuario: " + $usuario)

    $nomlogin = $usuario.nombre + $usuario.apellido1.substring(0,1) + $usuario.apellido2.substring(0,1)
    $apellidos = $usuario.apellido1 + " " + $usuario.apellido2
    $nombrever = $usuario.nombre + " " + $usuario.apellido1
    $cargoou = "ou=" + $usuario.cargo + ",dc=dominio,dc=lan"
    $passwd = $(ConvertTo-SecureString "Abc12345" -AsPlainText -Force)
    $correo = ($nomlogin + "@miempresa.com")

    try{
        echo ("Creando usuario : " + $nomlogin)
        New-ADUser -Name $nomlogin -UserPrincipalName $nomlogin -surname $apellidos -GivenName $usuario.nombre -DisplayName $nombrever -AccountPassword $passwd -path $cargoou -Enabled $true -ChangePasswordAtLogon:$true -Department -city "Alcoi" -eMailAddress $correo
        Add-ADGroupMember -Identity $usuario.Departamento -Members $nomlogin

        catch {
            echo ("Error al crear usuario: " + $nomlogin + $_.exception.message ) + " : " >> ficErrores.log

            #$_.exception.itemName # por si queremos ver el mensaje del error
            #break  #por si quisieramos para el proceso
        }
    }
}
```

```
