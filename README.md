# Helm Workshop

## Tabla de Contenidos

- [Helm Workshop](#helm-workshop)
  - [Tabla de Contenidos](#tabla-de-contenidos)
  - [Bienvenidos al Workshop](#bienvenidos-al-workshop)
  - [Requisitos Previos](#requisitos-previos)
  - [Ejercicios del Workshop](#ejercicios-del-workshop)
    - [1. Instala una aplicación con Kubernetes Manifest](#1-instala-una-aplicación-con-kubernetes-manifest)
    - [2. Instala WordPress con Helm](#2-instala-wordpress-con-helm)
    - [3. Convierte aplicación de Kubernetes manifest a Chart](#3-convierte-aplicación-de-kubernetes-manifest-a-chart)
    - [4. Conviértelo en un Chart parametrizado](#4-conviértelo-en-un-chart-parametrizado)
    - [6. Modifica el Chart y Haz Upgrade](#6-modifica-el-chart-y-haz-upgrade)
    - [7. Haz Rollback a la versión anterior](#7-haz-rollback-a-la-versión-anterior)
    - [8. Utiliza Template Functions](#8-utiliza-template-functions)
    - [9. Utiliza Flow Controls](#9-utiliza-flow-controls)
    - [10. Aplica Configuraciones por entornos](#10-aplica-configuraciones-por-entornos)
  - [Ejercicios Bonus](#ejercicios-bonus)
    - [11. Publica el Chart en GitHub](#11-publica-el-chart-en-github)
    - [12. Permitir múltiples instalaciones en un mismo namespace](#12-permitir-múltiples-instalaciones-en-un-mismo-namespace)


## Bienvenidos al Workshop
En este workshop aprenderás los fundamentos de Helm y cómo utilizarlo para gestionar despliegues declarativos en Kubernetes. A través de ejercicios prácticos, explorarás cómo simplificar y automatizar la implementación de aplicaciones en tu cluster.

## Requisitos Previos
Antes de comenzar, asegúrate de cumplir con los siguientes requisitos:

- Instalar Helm: [Guía de instalación oficial](https://helm.sh/docs/intro/install/)
- Configurar un cluster local: Puedes usar Minikube o el cluster de Kubernetes que viene con Docker Desktop.
  - [Instalar Minikube](https://minikube.sigs.k8s.io/docs/start/)
  - [Configurar Kubernetes en Docker Desktop](https://www.docker.com/blog/how-to-set-up-a-kubernetes-cluster-on-docker-desktop/)
- Tener instalado Git: [Guía de instalación](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

**Importante:**: Ten a la mano [Helm Cheat Sheet](https://helm.sh/docs/intro/cheatsheet/) durante el workshop.

## Ejercicios del Workshop
### 1. Instala una aplicación con Kubernetes Manifest
- En la carpeta [k8s_webapp_color/](k8s_webapp_color/) están los manifiestos a instalar.
- Usa `kubectl apply` para implementarlo en tu cluster.
- Explora cómo se ve el recurso creado usando `kubectl get` y `kubectl describe`

### 2. Instala WordPress con Helm
Usaremos un chart oficial para instalar WordPress. Ejecuta los siguientes comandos:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-wordpress bitnami/wordpress
```
Prueba cambiar configuraciones como credenciales:
```bash
helm install my-wordpress2 bitnami/wordpress --set wordpressUsername=xxxx --set wordpressPassword=xxxx
```

### 3. Convierte aplicación de Kubernetes manifest a Chart
3.1 Crea un nuevo chart usando:
```bash
helm create webapp_color_chart
```
3.2 Explora la estructura del chart
 generada y sus componentes principales (`values.yaml`, `templates/*`, etc.).

3.3 Convierte los manifests a templates
Elimina los archivos en `templates/*` y copia los manifiestos desde la carpeta `k8s_webapp_color/`. La estructura resultante será:
```bash
webapp_color_chart
├── Chart.yaml
├── charts
├── templates
│   ├── configmap.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── values.yaml
```
3.4 Ahora procedamos a instalar la aplicación con el Chart
 1. Primero debes desinstalar la aplicación creada con los manifests
 2. Usa este comando para instalar el chart:
```bash
helm install my-app ./webapp_color_chart
```
 3. Verifica el estado de la aplicación:
```bash
kubectl get pod,svc
```

### 4. Conviértelo en un Chart parametrizado
Usa variables como la imagen, el número de réplicas, o etiquetas.
```yaml
- image: {{ .Values.deployment.image.repository }}:{{ .Values.deployment.image.tag }}
```

Comandos útiles:
```bash
helm template my-app ./webapp_color_chart
helm lint ./webapp_color_chart
helm show values ./webapp_color_chart
helm upgrade my-app ./webapp_color_chart
```
Revisa [Helm Cheat Sheet](https://helm.sh/docs/intro/cheatsheet/) y ejecuta más comandos y comparte con la comunidad.

### 6. Modifica el Chart y Haz Upgrade
Cambia configuraciones en `values.yaml` y observa los cambios en el cluster:
```bash
helm upgrade my-app ./webapp_color_chart
```
### 7. Haz Rollback a la versión anterior
```bash
helm history my-app
helm rollback my-app <revision>
```

### 8. Utiliza Template Functions
Aprende a usar funciones como `required`, `default` y formateo de strings.
Modifica [webapp_color_chart/templates/configmap.yaml](webapp_color_chart/templates/configmap.yaml)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
  key: {{ .Values.myKey | default "defaultValue" }}
```

### 9. Utiliza Flow Controls
Crea templates con lógica condicional:
```yaml
{{- if .Values.createConfig }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
  key: value
{{- end }}
```

### 10. Aplica Configuraciones por entornos
Vamos a instalar la aplicación simulando la instalación en dos ambientes. dev y prod.
10.1 Crea un Namespace por ambiente: dev y prod
```bash
kubectl create ns dev
kubectl create ns prod
```
10.2 Usar configuraciones específicas por entorno
Crea y edita los archivos en el directorio [config/](config/):
- config/dev.yaml
- config/prod.yaml

10.3 Instala la aplicación en distintos entornos
```bash
helm install my-app ./webapp_color_chart -n dev -f config/dev.yaml
helm install my-app ./webapp_color_chart -n prod -f config/prod.yaml
```
10.4 Utiliza los comandos `kubectl` para comprobar

## Ejercicios Bonus
### 11. Publica el Chart en GitHub
Usa GitHub Pages o genera un índice de repositorio:
```bash
helm repo index . --url https://<tu-usuario>.github.io/<tu-repositorio>
```

Añade el repo a Helm:
```bash
helm repo add my-repo https://<tu-usuario>.github.io/<tu-repositorio>
helm search repo my-repo
```

### 12. Permitir múltiples instalaciones en un mismo namespace
Haz los ajustes necesarios para habilitar instalaciones múltiples.

*¡Buena suerte con el workshop!*