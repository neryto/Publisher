# Publicar Librerías AAR en GitHub Packages

Este documento describe el proceso para publicar librerías AAR en GitHub Packages y cómo configurarlas para que se utilicen en proyectos Android.

## 1. Introducción.

GitHub Packages es un servicio de alojamiento de paquetes de software que permite alojar paquetes de forma privada o pública y utilizarlos como dependencias en proyectos.
GitHub Packages ofrece diferentes registros de paquetes para los gestores de paquetes más utilizados, como npm, RubyGems, Apache **Maven**, Gradle, Docker y NuGet.

## 2. Configuración del Repositorio.

- Iniciar sesión en tu cuenta de GitHub.
- Crear un nuevo repositorio para alojar las librerías AAR. Asegúrse de habilitar GitHub Packages en el repositorio.

## 3. Aunteticación.

GitHub Packages sólo admite la autenticación mediante un token de acceso personal (clásico). Este token es necesario para publicar, instalar y eliminar paquetes privados, internos y públicos.

### 3.1. Crear un token de acceso personal.

Se puede utilizar un token de acceso personal (clásico) para autenticar con los paquetes de GitHub o con la API de GitHub. Cuando se crea un token de acceso personal (clásico), se puede asignar diferentes ámbitos en función de lo que se necesite.
A continuación se describe el proceso de como crear un token de acceso personal y configurarlo para administrar Github Packages.

#### 3.1.1. Generar un Token de Acceso Personal

  1. Iniciar sesión en GitHub.
  2. En la esquina superior derecha de la página, dar clic en la foto de perfil, y luego en **Settings**.
  3. En el menú de la izquierda, seleccionar **Developer settings**.
  4. Seleccionar **Personal access tokens** y luego **Tokens (classic)**.
  5. Dar clic en el botón **Generate new token**.

#### 3.1.2. Configurar el Token de Acceso

1. Asignar un nombre descriptivo para el token en el campo **Note**.
2. Selecciona los ambitos necesarios para el token:
    - Marcar la casilla **write:packages** para permitir la publicación de paquetes.
    - Marcar la casilla **read:packages** para permitir la lectura de paquetes.
    - Se puede marcar **delete:packages** si es necesario eliminar paquetes.
3. Establecer una fecha de vencimiento para el token usando el menú desplegable **Expiration**.
4. Hacer clic en **Generate token** al final del formulario.

### Notas: 
 * Es importante almacenar el token en un lugar seguro ya que una vez se cierra esta sección ya no es posible visualizar el token nuevamente.
 * El token de acceso personal solo tiene el nivel de acceso que el usuario posee. Si el usuario no tiene acceso a ciertos repositorios o paquetes, el token no podrá realizar operaciones sobre esos recursos.

## 4. Publicar las librerías.

Por practicidad para la publicación de las librerías se optó por hacer uso de un proyecto en Android Studio implementando el plugin **maven-publish** que proporciona gradle.

### 4.1. Configuración del proyecto.

  - En el archivo **build.gradle** (Nivel proyecto) agregar el plugin **maven-publish**

   ```
   plugins {
   ...
   id 'maven-publish'
   }
```

### 4.2. Agregar un archivo gradle-mvn-publish.gradle.

Este archivo contiene la configuración para la publicación de las librerías en nuestro respositorio Github Packages. 

gradle-mvn-publish.gradle

```
apply plugin: 'maven-publish'

publishing {
    publications {
        //libraria1
        library1(MavenPublication) {
            groupId = 'com.example'
            artifactId = 'my-library1'
            version = '1.0.0'

            // ruta del aar a publicar
            artifact("$buildDir/outputs/aar/my-library-release1.aar")

            // configuración del archivo pom aquí se agregan las dependencias de la librería a publicar
            pom.withXml {
            }
        }

       //libraria2
        library2(MavenPublication) {
            groupId = 'com.example'
            artifactId = 'my-library2'
            version = '1.0.0'

            // ruta del aar a publicar
            artifact("$buildDir/outputs/aar/my-library-release2.aar")

            // Ejemplo de dependencias, libreria2 tiene dependecia de libreria1
            pom.withXml {
                def dependenciesNode = asNode().appendNode('dependencies')
                def dependency = dependenciesNode.appendNode('dependency')
                dependency.appendNode('groupId', "com.example")
                dependency.appendNode('artifactId',"my-library1")
                dependency.appendNode('version', "1.0.0")
            }
        }

        .
        .
        .

    }
    //repositorio en donde se va a publicar la librería 
    repositories {
        maven {
            url = uri("https://maven.pkg.github.com/OWNER/REPOSITORY")
            credentials {
                username = "USERNAME"
                password = "TOKEN_DE_ACCESO"
            }
        }
    }
}

```
Una vez configurado el archivo gradle-mvn-publish.gradle se debe de indicar que se construya el script, esto se indica en el archivo **build.gradle** de módulo app o en su caso el múdulo que define la librería a publicar.

```
buildscript {
    apply {
        from("$rootDir/app/gradle-mvn-push.gradle")

    }
}
```
### 4.3. Publicación de las librerías.

Para publicar la(s) librería(s) se siguen los siguentes pasos:

  - Sincronizar el proyecto con los archivos gradle
  - En la parte derecha de Android Studio seleccionar el ícono de gradle
  - En el menú de gradle seleccionar Tasks -> publising -> publish

### Nota: 
 * Si no se visualiza la sección de publishing en el menú de tareas de grale ir a Android Studio -> Settings -> Experimental y activar la casilla **Configure all Gradle task** 

<img width="527" alt="Screenshot 2024-08-09 at 9 41 47 p m" src="https://github.com/user-attachments/assets/690a551c-3be5-4a96-bfaf-9fed6881f7b6">

## 5. Consumir la Librería AAR en un Proyecto Android.

### 5.1. Configurar el Repositorio en **build.gradle**

En el archivo build.gradle del módulo del proyecto Android, agregar el repositorio de GitHub Packages:

```
repositories {
    maven {
        url = uri("https://maven.pkg.github.com/OWNER/REPOSITORY")
        credentials {
            username = "USERNAME"
            password = "TOKEN"
        }
    }
}
```
### 5.2. Agregar la Dependencia

Agrega la dependencia de la librería en tu archivo build.gradle del módulo:

```
dependencies {
    implementation 'com.example:my-library1:1.0.0'
}
```

## 6. Fuentes.

[Github Packages Maven](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry)











