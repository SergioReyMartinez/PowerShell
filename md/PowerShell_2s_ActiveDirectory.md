---
title: PowerShell AD Solución ejercicios
permalink: /PowerShell_2_ActiveDirectory_Solucion/
---




# 1. Active Directory - Permisos

## 1.1. Creación de carpetas y permisos SMB

Creación de una carpeta y asignar permisos SMB
```powershell
Set-Location e:\                                                                                 
Get-ChildItem                                                                                    
New-Item Desarrollo -ItemType Directory                                                          
New-Item Desarrollo\Web -ItemType Directory                                                      
New-Item Desarrollo\Multiplataforma -ItemType Directory                                          
New-SmbShare -Name Desarrollo -Path E:\Desarrollo\ -FullAccess Administradores -ChangeAccess gDesarrollo00,gDesarrolloMulti00 -ReadAccess gSocios00
```
## 1.2. Gestión de Permisos NTFS

Los siguientes comando describen cómo manipular los permisos NTFS de una carpeta

### 1.2.1. Ver los permisos de una carpeta
En primer lugar es esencial tener una forma de ver todos los permisos de una carpeta. 
A continuación vemos la mejor forma. 
Por una parte la función ```Get-ACL``` obtiene el listado de permiso, y mediante el método ```Access``` los muestra. Utilizamos la función ```Format-Table``` para visualizar en forma de tabla.
Por último, creamos una variable donde vamos a almacenar la dirección de la carpeta o fichero con el que vamos a trabajar. Esto no es necesario, pero puede servir para reutilizar código de forma sencilla si cambiamos de carpeta o fichero.

```powershell
$Carpeta="c:\CarpetaCompartida"
(Get-ACL -Path $Carpeta).Access | Format-Table IdentityReference, FileSystemRights, AccessControlType, IsInherited, InheritanceFlags -AutoSize
```

### 1.2.2. Deshabilitar la herencia

Para poder eliminar ciertos permisos debemos deshacer la herencia de los mismos.
Lo realizamos mediante la funcion ```SetAccessRuleProtection``` tal y como vemos a continuación.
Como podemos ver, en primer lugar metemos en una variable el listado de permisos que obtenemos mediante la funcion ```Get-ACL``` y después una vez eliminados los permisos en la variable, se vuelven a volcar o aplicar al recurso inicial mediante la función ```Set-ACL```

```powershell
$ACL = Get-Acl -Path $Carpeta                                                             
$ACL.SetAccessRuleProtection( $true, $true)                                                      
$ACL | Set-Acl -Path $Carpeta                                                             
```

### 1.2.3. Eliminar todos los permisos de un usuario

Para eliminar los permisos de un recurso, comenzamos como siempre obteniendo en una variable todos los permisos, y mediante el filtro ```Where-Object``` filtramos y obtenemos los permisos concretos que queremos eliminar.
La eliminación se realiza mediante la función ```RemoveAccessRule```, y posteriormente, como en el caso anteior, volvemos a asignar los nuevos permisos al recurso inicial

```powershell
$Acl = Get-Acl -Path $Carpeta
$AccessRule = $Ace.Access | Where-Object {($_.IdentityReference -eq 'BUILTIN\Usuarios') -and -not ($_.IsInherited)}
$Acl.RemoveAccessRule( $AccessRule)
$Acl | Set-Acl -Path $Carpeta
$Acl.Access | Format-Table IdentityReference, FileSystemRights, AccessControlType, IsInherited, InheritanceFlags -AutoSize
```

### 1.2.4. Añadir permisos a un usuario

Para añadir un nuevo permiso a una carpeta, creamos el permiso mediante la función ```New-Object System.Security.AccessControl.FileSystemAccessRule()``` y asígnamos en orden los valores para el usuario o grupo de seguridad, los permisos que tiene, opciones de herencia y de propagación, y el tipo de permiso que se esta aplicando, que puede ser de permiso o de denegación del mismo.
Una vez creado el nuevo permiso, se añade al listado ACL de la carpeta y por último, como siempre, se reescriben todos los permisos de la carpeta

```powershell
$Acl = Get-Acl -Path "c:\CarpetaCompartida"
$AccessRule = New-Object System.Security.AccessControl.FileSystemAccessRule( "gDesarrollo00", "ReadAndExecute", "ContainerInherit, ObjectInherit", "None", "Allow")
$Acl.AddAccessRule( $AccessRule)
$ACL | Set-Acl -Path "c:\CarpetaCompartida"
```

### 1.2.5. Funcion **```New-Object System.Security.AccessControl.FileSystemAccessRule()```**

Función que crea un permiso específico NTFS.

A continuación se detallan los posibles valores que pueden tomar cada uno de los parámetros de la funcioń

| Parámetro | Valores |
| --- | --- |
| ***name*** | Usuario o grupo de seguridad |
| ***FileSystemRights*** | ListDirectory, ReadData, WriteData, CreateFiles, CreateDirectories, AppendData, ReadExtendedAttributes, WriteExtendedAttributes, Traverse, ExecuteFile, DeleteSubdirectoriesAndFiles, ReadAttributes, WriteAttributes, Write, Delete, ReadPermissions, Read, ReadAndExecute, Modify, ChangePermissions, TakeOwnership, Synchronize, FullControl | 
| ***InheritanceFlags*** | None, ContainerInherit, ObjectInherit | 
| ***PropagationFlags*** | None, NoPropagateInherit, InheritOnly | 
| ***AccessControlType*** | Allow, Deny | 




# 2. Scripts de creación de recursos de empresas
En los siguiente ejemplos se ve el tipico ejercicio de creación de unidades, grupo y usuarios con PowerShell

## 2.1. Ejemplo 1: Importar un fichero CSV
Para la importación de un fichero CSV, es necesaria la funcion **import-csv** y el fichero tiene que tener un formato especificado con los titulos de los campos en la primera línea (y puntos en la segúnda ????)
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

## 2.2. Ejemplo 2: Creación de unidades organizativas de un fichero 

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

## 2.3. Ejemplo 3: Creación de grupos de un fichero 
```powershell
$nombres = import-csv .\archivoscsv\departamentos.csv
forEach ( $grupo in $nombres){
    Write-Host ( "Creando grupo : " + $unidad.nombre + " en " + $unidad.ubicacion)
    $ruta = "ou=" + $grupo.ubicacion + ",dc=dominio,dc=lan"
    new-ADGroup -name $unidad.nombre -GroupCategory security -GroupScope Global  -Path "dc=dominio,dc=curso" -description $unidad.descripcion $ruta
    echo Creado
}
```

## 2.4. Ejemplo 4: Creación de usuarios 
no se puede crear un usuario añadiendolo a un grupo, por eso se hace despues en otra línea
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

# 3. PSDrive
Se trata de una unidad donde se puede almacenar datos

En MS-DOS teniamos el **net use**
```powershell
net use j: "\\servidor\compartido"
```

Lo mismo con PSDrive
Listado de todos los PS o los de ficheros
```powershell
get-psdrive
get-psdrive -psprovider filesystem
new-psdrive -name p -psprovider filesystem -root c:\temp
```
