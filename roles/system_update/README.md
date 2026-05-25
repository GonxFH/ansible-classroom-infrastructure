# system_update

Role Ansible para actualización automática del sistema y reinicio controlado en caso necesario.

---

## Descripción

Este role realiza mantenimiento completo del sistema mediante APT y gestiona reinicios automáticos cuando son requeridos tras actualizaciones críticas.

Está diseñado para entornos de aulas donde los equipos deben mantenerse actualizados sin intervención manual.

Este role realiza tareas básicas de mantenimiento del sistema:

- actualiza el índice de paquetes
- instala actualizaciones disponibles
- elimina dependencias obsoletas

Está pensado para mantenimiento periódico de aulas Ubuntu/Lubuntu y equipos clonados.

---


## Objetivo

Mantener los equipos actualizados de forma homogénea y automatizada evitando intervención manual.

Permite:

- aplicar actualizaciones de seguridad
- mantener paquetes al día
- eliminar dependencias innecesarias
- reducir tareas de mantenimiento repetitivas

---

## Funcionamiento

El role ejecuta el siguiente flujo:

1. Actualización de índices APT
2. Comprobación de validez de caché
3. Actualización del sistema (APT)
4. Eliminación de paquetes obsoletos
5. Comprobación de reinicio requerido
6. Reinicio automático si está habilitado

---

## Variables disponibles

| Variable | Descripción | Obligatoria | Valor por defecto |
|---|---:|---:|---|
| system_update_upgrade_type | Tipo de actualización APT | No | dist |
| system_update_cache_valid_time | Validez de caché APT (segundos) | No | 3600 |
| system_update_autoremove | Elimina dependencias no usadas | No | true |
| system_update_auto_reboot | Reinicio automático si es necesario | No | true |
| system_update_reboot_timeout | Tiempo de espera del reinicio (seg) | No | 600 |

---

## Tipos de actualización (system_update_upgrade_type)

La variable `system_update_upgrade_type` controla el tipo de actualización que realiza APT sobre el sistema.

Es uno de los parámetros más importantes del role, ya que determina el nivel de cambios que se aplican.

### safe

```text
safe → apt upgrade
```

Realiza actualizaciones seguras del sistema:

- actualiza paquetes instalados
- no elimina paquetes
- no instala dependencias nuevas que cambien la estructura del sistema

Uso recomendado:
- entornos muy estables
- servidores críticos
- cuando se quiere minimizar cualquier cambio del sistema

### full

```text
full → apt full-upgrade
```

Permite actualizaciones más completas que `safe`:

- puede instalar nuevas dependencias
- puede eliminar paquetes conflictivos
- resuelve cambios de dependencias automáticamente

Uso recomendado:
- mantenimiento general
- equipos de usuario
- sistemas con cambios moderados

### dist (valor por defecto)

```text
dist → apt dist-upgrade
```

Realiza la actualización más completa:

- permite cambios profundos en dependencias
- puede reemplazar paquetes del sistema
- puede eliminar o instalar componentes necesarios para la nueva versión de paquetes

Uso recomendado:
- mantenimiento automatizado de aulas
- entornos donde se quiere mantener el sistema completamente actualizado
- sistemas clonados que deben mantenerse homogéneos

---

### Comparativa de tipos de actualización

| Tipo  | Comando APT equivalente | Nivel de cambios | Instalación/eliminación de paquetes | Riesgo | Uso recomendado |
|-------|--------------------------|------------------|--------------------------------------|--------|-----------------|
| safe  | apt upgrade              | Bajo             | No instala ni elimina paquetes      | Bajo   | Sistemas críticos, máxima estabilidad |
| full  | apt full-upgrade         | Medio            | Puede instalar dependencias nuevas  | Medio  | Uso general, mantenimiento habitual |
| dist  | apt dist-upgrade         | Alto             | Puede instalar o eliminar paquetes  | Alto   | Aulas, entornos automatizados, sistemas clonados |

---

### Recomendación general

Para este role se recomienda:

```yaml
system_update_upgrade_type: dist
```

ya que permite mantener los equipos de aula completamente actualizados sin intervención manual.


### Recomendación según escenario

#### Aula en uso (clases en horario lectivo)

Recomendación:

```yaml
system_update_upgrade_type: safe
system_update_auto_reboot: false
```

Motivo:
- evita cambios agresivos en el sistema
- reduce riesgo de interrupciones
- no provoca reinicios inesperados

#### Mantenimiento programado (fuera de horario lectivo)

Recomendación:

```yaml
system_update_upgrade_type: full
system_update_auto_reboot: true
```

Motivo:
- actualiza el sistema de forma más completa
- permite reinicios controlados
- mantiene equilibrio entre estabilidad y actualización

#### Despliegue automático de aulas (caso principal del proyecto)

Recomendación:

```yaml
system_update_upgrade_type: dist
system_update_auto_reboot: true
```

Motivo:
- asegura que todos los equipos estén completamente actualizados
- resuelve dependencias automáticamente
- ideal para equipos clonados y homogéneos
- minimiza intervención manual

---

## Ejemplo de uso

```yaml
- hosts: aula
  become: true

  roles:
    - role: system_update
```

---

## Ejemplo con control de ejecución por lotes (serial)

En entornos de aulas no es recomendable actualizar todos los equipos al mismo tiempo, especialmente cuando el role puede provocar reinicios.

Para evitar caídas simultáneas de todos los equipos, se puede usar `serial`:

```yaml
- hosts: aulaB30
  serial: 5
  become: true

  roles:
    - system_update
```

### ¿Qué hace `serial: 5`?

Ansible ejecuta el playbook en grupos de 5 equipos a la vez en lugar de hacerlo sobre todos simultáneamente.

Por ejemplo, si el inventario tiene 30 equipos:

```text
30 equipos → 6 tandas de 5 equipos
```

Cada lote se completa antes de pasar al siguiente.

---

### ¿Por qué es importante en este role?

Este role:

- actualiza el sistema
- puede reiniciar automáticamente equipos
- puede dejar temporalmente equipos fuera de servicio

Usar `serial` permite:

- evitar que toda el aula se reinicie a la vez
- mantener parte del aula operativa durante la actualización
- reducir impacto en sesiones de clase
- controlar mejor la carga de red y repositorios APT

---

### Ejemplo alternativo (más conservador)

```yaml
- hosts: aulaB30
  serial: 10
  become: true

  roles:
    - system_update
```

En este caso se actualizan 10 equipos por tanda en lugar de 5.

---

### Recomendación

- `serial: 5` → entornos críticos o aulas en uso
- `serial: 10` → mantenimiento más rápido con menor riesgo
- sin `serial` → actualización simultánea (no recomendado en aulas con reinicio automático)


## Ejemplo con configuración personalizada

```yaml
- hosts: aula
  become: true

  roles:
    - role: system_update
      vars:
        system_update_auto_reboot: false
        system_update_upgrade_type: safe
```

---

## Archivos modificados

Este role no modifica ficheros de configuración.

Interviene sobre:

- paquetes del sistema
- dependencias instaladas
- estado del sistema (reinicio si es necesario)

---

## Requisitos

- Sistemas Debian / Ubuntu
- Acceso sudo
- Conectividad a repositorios APT

---

## Reinicio automático

Ubuntu genera el archivo:

```text
/run/reboot-required
```

cuando es necesario reiniciar el sistema tras actualizaciones críticas.

Este role:

- detecta ese archivo
- ejecuta reinicio automático si está habilitado
- espera a que el equipo vuelva a estar accesible por SSH

---


## Consideraciones

El parámetro:

```yaml
system_update_upgrade_type: dist
```

permite cambios de dependencias durante la actualización.

Equivale aproximadamente a:

```bash
apt-get dist-upgrade
```

Esto permite actualizaciones más completas, pero puede instalar o eliminar paquetes automáticamente.

---

## Consideraciones importantes

### Entornos de aula

En entornos con múltiples equipos:

- el reinicio puede interrumpir sesiones activas
- es recomendable ejecutar este role en horarios controlados

### Seguridad operativa

El reinicio automático evita:

- sistemas parcialmente actualizados
- kernels antiguos activos tras actualización

pero debe usarse con planificación en entornos educativos.

---

## Limitaciones

Este role no distingue entre:

- usuarios conectados
- sesiones activas
- procesos críticos en ejecución

El reinicio se basa exclusivamente en la señal del sistema operativo.


## Consideraciones de mantenimiento

El role ejecuta opcionalmente:

```yaml
autoremove: true
```

Esto elimina dependencias instaladas automáticamente que APT considera innecesarias.

Aunque normalmente es seguro, conviene revisar este comportamiento si existen roles adicionales que instalen paquetes opcionales o configuraciones poco habituales.

En infraestructuras complejas puede ser recomendable desactivar:

```yaml
system_update_autoremove: false
```

y gestionar la limpieza mediante procesos específicos.
