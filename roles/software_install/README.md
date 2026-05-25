# software_install

Role Ansible para instalación y gestión declarativa de software en entornos educativos mediante perfiles reutilizables.

Este role permite instalar software desde múltiples fuentes:

- APT (repositorios oficiales)
- Snap
- Flatpak
- paquetes .deb locales
- paquetes .deb remotos
- roles adicionales (instalación avanzada)

---

## Objetivo

Este role simplifica la gestión del software en aulas permitiendo:

- definir “perfiles de software”
- combinar perfiles de forma modular
- automatizar instalaciones complejas
- mantener coherencia entre equipos clonados
- evitar configuraciones manuales repetitivas

---

## Concepto clave: perfiles de software

En lugar de instalar software manualmente en cada equipo:

```yaml
apt:
  - firefox
  - libreoffice
  - gimp
```

se utilizan perfiles:

```yaml
software_profiles:
  - software_base
  - software_modulo_X
  - software_profesor_X
  - software_evento_X
```

Cada perfil contiene su propio software.

---

## Estructura de perfiles

Los perfiles se almacenan en:

```text
software/
```

Ejemplo:

```text
software/
├── software_base.yml
├── software_modulo_X.yml
├── software_profesor_Y.yml
└── software_evento_Z.yml
```

---

## Formato de un perfil

Un perfil NO tiene estructura obligatoria completa.

Cada sección es opcional:

- apt → paquetes APT
- snap → paquetes Snap
- flatpak → aplicaciones Flatpak
- deb → paquetes .deb
- roles → roles adicionales

Solo debes definir lo que necesite el perfil.

Además, el orden de los campos dentro de un perfil NO es relevante.

Ejemplo:

```yaml
apt:
  - firefox

snap:
  - code
```

es equivalente a

```yaml
snap:
  - code

apt:
  - firefox
```

### Ejemplo de perfil con todos los campos

```yaml
apt:
  - firefox
  - vlc

snap:
  - name: code
    classic: true

flatpak:
  - org.gimp.GIMP

deb:
  - chrome.deb
  - https://ejemplo.com/app.deb

roles:
  - virtualbox
  - docker
```

---

## Funcionamiento general

El role ejecuta automáticamente:

1. Validación de perfiles
2. Carga de perfiles
3. Unión de software de todos los perfiles
4. Instalación APT
5. Instalación Snap
6. Instalación Flatpak
7. Instalación .deb
8. Ejecución de roles adicionales
9. Limpieza del sistema

El orden de aparición de los tipos de aplicaciones en un perfil 
no importa; sin embargo, el orden de instalación de las
aplicaciones por tipo sí es fijo y es éste:

- APT
- Snap
- Flatpak
- DEB
- Roles adicionales
- Limpieza

---

# Roles adicionales (software_roles)

Este es uno de los elementos más potentes del sistema.

Generalmente, cada uno de estos roles adicionales
Permite ejecutar roles adicionales tras la instalación del softwa.

Generalmente, estos roles se encargarán de gestionar la instalación
de
---

## ¿Qué hace?

Ejecuta roles Ansible independientes relacionados con software complejo.

---

## Ejemplo básico

```yaml
software_roles:
  - virtualbox
  - docker
  - gns3
```

---

## 🖥️ Ejemplo en aula de informática avanzada

```yaml
software_profiles:
  - base
  - programacion

software_roles:
  - virtualbox
  - docker
  - gns3
```

---

## Casos de uso reales

### VirtualBox

- instalación + configuración kernel modules

### Docker

- instalación + permisos de usuario + daemon

### GNS3

- instalación + servicios + dependencias de red

---

## ⚠️ Importante

- Cada elemento debe ser un role existente en el proyecto
- Se ejecutan en el mismo contexto del play
- Se ejecutan en orden secuencial
- Comparten variables del entorno

---

## 🚫 Ejemplo de error

```yaml
software_roles:
  - gns3
  - no_existe
```

❌ El playbook fallará si el role no existe

---

# Limpieza del sistema

El role realiza limpieza automática:

### Dependencias no usadas

```bash
apt autoremove
```

---

### Caché de paquetes

```bash
apt autoclean
```

---

### Limpieza completa (opcional)

```yaml
software_full_clean: true
```

equivale a:

```bash
apt clean
```

---

# Instalación de paquetes .deb

Soporta dos tipos:

---

## Locales

```yaml
deb:
  - chrome.deb
```

Ubicación:

```text
software_deb/
```

---

## Remotos

```yaml
deb:
  - https://ejemplo.com/app.deb
```

---

## Proceso

1. Copia o descarga
2. Instalación con apt
3. Limpieza opcional

---

# Características avanzadas

---

## Combinación de perfiles

Todos los perfiles se fusionan:

- apt
- snap
- flatpak
- deb
- roles

sin duplicados (`unique`)

---

## Idempotencia

El role es sesoftware_roles:
          - virtualbox
          - docker
          - gns3guro:

- no reinstala software ya presente
- puede ejecutarse múltiples veces

---

## Arquitectura modular

Este role actúa como:

> “motor central de instalación de software”

y los roles adicionales como:

> “plugins del sistema”

---

# Ejemplo completo de aula real

```yaml
- hosts: aulaB30
  serial: 5
  become: true

  roles:
    - role: software_install
      vars:
        software_profiles:
          - software_base
          - modulo_serviciosRed
          - modulo_apWeb
          - modulo_SOR
          - software_profesor_X
          - software_evento_X

        software_full_clean: true
```
---

# Limitaciones

- Requiere conectividad a repositorios
- Snap y Flatpak pueden requerir inicialización previa
- Los roles adicionales deben existir en el proyecto
- No gestiona conflictos entre versiones de software

---

# Conclusión

Este role es el núcleo del sistema de aulas:

- convierte la instalación de software en algo declarativo  
- permite combinar perfiles  
- extiende funcionalidad mediante roles  
- centraliza toda la gestión de aplicaciones  

Es la pieza clave del sistema de automatización del aula.
