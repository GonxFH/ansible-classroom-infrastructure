# hostname_by_ip

Role Ansible para generar automáticamente el hostname de los equipos de un aula a partir de su dirección IP.

## Descripción

Este role construye automáticamente el nombre del equipo utilizando:

- Nombre identificador del aula
- Último octeto de la dirección IPv4 principal

Formato generado:

```text
AULA + PC + XXX
```

Ejemplo:

```text
APRUEBAPC005
```

Si un equipo tiene:

```text
IP: 192.168.1.5
Aula: APrueba
```

El hostname generado será:

```text
APRUEBAPC005
```

---

## Objetivo

En aulas con equipos clonados resulta frecuente que múltiples equipos compartan el mismo hostname.

Este role permite:

- Generar nombres únicos automáticamente
- Mantener nomenclatura homogénea
- Evitar intervención manual tras clonar equipos
- Facilitar inventario y administración remota

---

## Funcionamiento

El role ejecuta los siguientes pasos:

1. Valida parámetros obligatorios
2. Obtiene la IPv4 principal
3. Extrae el último octeto
4. Convierte el número a formato de tres cifras
5. Construye el hostname final
6. Aplica el hostname
7. Actualiza `/etc/hosts`

---

## Variables disponibles

| Variable | Descripción | Obligatoria | Valor por defecto |
|---|---:|---:|---|
| hostname_classroom | Identificador del aula | Sí | DEFAULT |
| hostname_hosts_ip | IP usada en /etc/hosts | No | 127.0.1.1 |

---

## Ejemplo de uso

```yaml
- hosts: aula
  become: true

  roles:
    - role: hostname_by_ip
      vars:
        hostname_classroom: APrueba
```

---

## Ejemplo de resultado

| IP | Hostname generado |
|---|---|
| 192.168.1.2 | APRUEBAPC002 |
| 192.168.1.5 | APRUEBAPC005 |
| 192.168.1.27 | APRUEBAPC027 |
| 192.168.1.135 | APRUEBAPC135 |

---

## Tags disponibles

Permiten ejecutar partes concretas del role:

```bash
--tags validate
```

Ejecuta únicamente validaciones.

```bash
--tags build
```

Construye el hostname sin aplicarlo.

```bash
--tags set
```

Aplica hostname al sistema.

```bash
--tags update
```

Actualiza `/etc/hosts`.

---

## Requisitos

- Ubuntu / Debian
- IPv4 configurada
- facts de Ansible habilitados
- permisos sudo

---

## Consideraciones

Este role utiliza:

```yaml
ansible_default_ipv4.address
```

Por tanto:

- requiere recopilación de facts
- necesita una interfaz principal correctamente detectada
- no está pensado para equipos exclusivamente IPv6

La numeración se obtiene únicamente del último octeto de la IP.

Por ejemplo:

```text
192.168.1.23 → PC023
```

Cambiar la IP del equipo provocará un cambio automático del hostname en la siguiente ejecución.

### Resolución local mediante /etc/hosts

El role actualiza automáticamente:

```text
/etc/hosts
```

Esto permite que el propio equipo resuelva su hostname aunque:

- la red aún no esté disponible
- cambie la IP
- falle DNS
- existan problemas temporales de conectividad

Ubuntu utiliza habitualmente:

```text
127.0.1.1 hostname
```

para este propósito.


## Archivos modificados

El role modifica:

| Archivo | Descripción |
|---|---|
| hostname del sistema | Nombre del equipo |
| /etc/hosts | Resolución local del hostname |

---

## Ejemplo antes/después

Estado inicial:

```text
IP: 192.168.1.5
Hostname: ubuntu
```

Contenido de:

```text
/etc/hosts
```

```text
127.0.0.1 localhost
127.0.1.1 ubuntu
```

Tras ejecutar el role:

```text
Hostname: APRUEBAPC005
```

Nuevo contenido:

```text
127.0.0.1 localhost
127.0.1.1 APRUEBAPC005
```
## Limitaciones y consideraciones técnicas

### Generación basada únicamente en el último octeto

La numeración del equipo se obtiene exclusivamente del último octeto de la IPv4 principal.

Ejemplo:

```text
192.168.1.25 → PC025
10.0.0.25    → PC025
```

Por tanto, equipos pertenecientes a subredes distintas pueden generar el mismo identificador numérico.

Esto normalmente no supone un problema si cada aula utiliza un identificador distinto:

```text
APRUEBAPC025
AINFO1PC025
```

Sin embargo, si varias aulas reutilizan el mismo valor de:

```yaml
hostname_classroom
```

podrían producirse hostnames duplicados.

Se recomienda utilizar nombres de aula únicos dentro de la organización.

---

### Dependencia de la interfaz IPv4 principal detectada por Ansible

El role utiliza:

```yaml
ansible_default_ipv4.address
```

para obtener la dirección IP del equipo.

En sistemas con múltiples interfaces de red puede ocurrir que Ansible detecte una interfaz distinta a la esperada.

Ejemplos habituales:

- Ethernet + Wi-Fi simultáneamente
- Interfaces Docker
- VPN activas
- Adaptadores virtuales
- Múltiples tarjetas de red

Esto podría generar un hostname basado en una IP inesperada.

Para verificar qué dirección utilizará el role puede ejecutarse:

```bash
ansible equipo -m setup -a "filter=ansible_default_ipv4"
```

Ejemplo de salida:

```yaml
ansible_default_ipv4:
  address: 192.168.1.23
```

Si el entorno utiliza múltiples interfaces puede ser recomendable adaptar el role para seleccionar una interfaz concreta.
