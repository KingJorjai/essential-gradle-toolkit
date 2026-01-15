# Análisis de Soporte para Nuevas Versiones de Minecraft

**Fecha:** 15 de enero de 2026  
**Versión del toolkit:** 0.6.10

## Resumen Ejecutivo

Este documento analiza el estado actual del soporte de versiones de Minecraft en Essential Gradle Toolkit y proporciona un plan para agregar soporte para nuevas versiones.

## Estado Actual

### Versión Máxima Soportada

**Minecraft 1.21.8** (código interno: `12108`)

Esta es la versión más reciente actualmente soportada en el toolkit, con soporte completo para:
- **Fabric** con Yarn mappings: `1.21.8+build.1:v2`
- **Forge**: `1.21.8-58.0.0`
- **NeoForge**: `21.8.5-beta`

### Sistema de Codificación de Versiones

El toolkit utiliza un sistema de codificación numérica para las versiones de Minecraft:

```
código = mcMajor * 10000 + mcMinor * 100 + mcPatch
```

**Ejemplos:**
- Minecraft 1.8.9 → `10809`
- Minecraft 1.21.8 → `12108`
- Minecraft 1.22.0 → `12200` (hipotético)

### Loaders Soportados

1. **Fabric** - Todas las versiones desde 1.14.4
2. **Forge** - Todas las versiones desde 1.7.10
3. **NeoForge** - Versiones modernas desde 1.20.2

### Rangos de Versiones Actualmente Soportados

#### Versiones Legacy (< 1.14.0)
- 1.7.10 (Forge)
- 1.8.0, 1.8.9 (Forge)
- 1.9.4 (Forge)
- 1.10.2 (Forge)
- 1.11.0, 1.11.2 (Forge)
- 1.12.0, 1.12.1, 1.12.2 (Forge)

#### Versiones Modernas (≥ 1.14.0)
- Serie 1.14: 1.14.4
- Serie 1.15: 1.15.2
- Serie 1.16: 1.16.1, 1.16.2, 1.16.4, 1.16.5
- Serie 1.17: 1.17.0, 1.17.1
- Serie 1.18: 1.18.1, 1.18.2
- Serie 1.19: 1.19.0, 1.19.1, 1.19.2, 1.19.3, 1.19.4
- Serie 1.20: 1.20.0, 1.20.1, 1.20.2, 1.20.3, 1.20.4, 1.20.5, 1.20.6
- Serie 1.21: 1.21.0, 1.21.1, 1.21.2, 1.21.3, 1.21.4, 1.21.5, 1.21.6, 1.21.7, 1.21.8

## Análisis de Posibles Nuevas Versiones

### Versiones Futuras Potenciales

Basándose en el patrón de lanzamientos de Minecraft:

1. **Minecraft 1.21.9+** (código: `12109+`)
   - Posibles parches adicionales de la serie 1.21
   - Requeriría: Yarn mappings, Forge, NeoForge

2. **Minecraft 1.22.x** (código: `12200+`)
   - Próxima versión menor de Minecraft
   - Requeriría: Yarn mappings, Forge (si está disponible), NeoForge
   - Posible cambio en requisitos de Java (actualmente Java 21 para ≥1.20.5)

3. **Minecraft 1.23.x y posteriores** (código: `12300+`)
   - Versiones futuras
   - Podrían requerir actualizaciones significativas

## Proceso para Agregar Soporte a una Nueva Versión

### Archivo Principal: `src/main/kotlin/gg/essential/defaults/loom.gradle.kts`

Este archivo contiene todas las configuraciones de versiones en la estructura `Revision`.

### Pasos Necesarios

#### 1. Verificar Disponibilidad de Dependencias

Antes de agregar soporte, verificar que existan:

- **Yarn mappings** (para Fabric):
  - Formato: `net.fabricmc:yarn:<version>+build.<build>:v2`
  - Ejemplo: `1.21.8+build.1:v2`
  - Fuente: https://maven.fabricmc.net/

- **Forge**:
  - Formato: `net.minecraftforge:forge:<mcVersion>-<forgeVersion>`
  - Ejemplo: `1.21.8-58.0.0`
  - Fuente: https://files.minecraftforge.net/

- **NeoForge**:
  - Formato: `net.neoforged:neoforge:<version>`
  - Ejemplo: `21.8.5-beta`
  - Fuente: https://maven.neoforged.net/

#### 2. Actualizar loom.gradle.kts

Agregar las nuevas versiones al **primer** `Revision` en el archivo:

```kotlin
revisions.add(Revision(
    yarn = mapOf(
        // ... versiones existentes ...
        12109 to "1.21.9+build.1:v2",  // Nueva versión
    ),
    forge = mapOf(
        // ... versiones existentes ...
        12109 to "1.21.9-XX.X.X",  // Nueva versión
    ),
    neoForge = mapOf(
        // ... versiones existentes ...
        12109 to "21.9.X-beta",  // Nueva versión
    ),
    // ... otros campos ...
))
```

#### 3. Actualizar Platform.kt (si es necesario)

Si la nueva versión requiere una versión diferente de Java, actualizar el método `javaVersion` en `Platform.kt`:

```kotlin
val javaVersion = when {
    mcVersion >= 12XXX -> JavaVersion.VERSION_XX  // Si hay cambios
    mcVersion >= 12005 -> JavaVersion.VERSION_21
    mcVersion >= 11800 -> JavaVersion.VERSION_17
    mcVersion >= 11700 -> JavaVersion.VERSION_16
    else -> JavaVersion.VERSION_1_8
}
```

#### 4. Incrementar Revisión

Si se modifican versiones existentes (no solo se agregan nuevas), crear una nueva revisión:

```kotlin
revisions.add(revisions.last().update(
    // Solo las versiones que cambian
))
```

### Consideraciones Importantes

1. **Agregar al primer Revision**: Las versiones nuevas se agregan al primer `Revision` para que estén disponibles para todos los usuarios.

2. **Crear nueva Revision para cambios**: Solo crear una nueva revisión cuando se modifican entradas existentes, para mantener compatibilidad con usuarios en revisiones antiguas.

3. **Fabric Loader**: La versión del fabric-loader (actualmente `0.13.3`) debe actualizarse si las nuevas versiones de Minecraft lo requieren.

4. **Mappings especiales**: 
   - Forge moderno (≥1.17.0) usa mappings oficiales de Mojang
   - Forge legacy usa MCP mappings

## Limitaciones y Restricciones

### Limitaciones Técnicas

1. **Dependencia de ecosistema externo**:
   - No se puede agregar soporte hasta que Yarn, Forge y/o NeoForge publiquen versiones compatibles
   - Los tiempos de lanzamiento varían entre plataformas

2. **Architectury Loom**:
   - El toolkit depende de architectury-loom
   - Nuevas versiones de Minecraft requieren que Loom las soporte primero

3. **Requisitos de Java**:
   - Versiones nuevas pueden requerir versiones más recientes de Java
   - Esto afecta la compatibilidad con proyectos existentes

### Consideraciones de Mantenimiento

1. **Versiones de prueba**:
   - Snapshots y pre-releases generalmente no se agregan
   - Solo versiones estables de lanzamiento

2. **Actualización de dependencias**:
   - Yarn mappings a veces recibe actualizaciones (mayor número de build)
   - Forge puede publicar versiones parcheadas
   - Estas actualizaciones pueden agregarse mediante nuevas revisiones

## Recomendaciones

### Corto Plazo (Próximos 3 meses)

1. **Monitorear lanzamientos de Minecraft**:
   - Minecraft 1.21.9 o superiores si se lanzan
   - Actualizar tan pronto como Yarn/Forge/NeoForge estén disponibles

2. **Mantener actualizado Fabric Loader**:
   - Revisar si hay versiones más recientes de fabric-loader
   - Actualizar si es necesario para compatibilidad

### Mediano Plazo (3-6 meses)

1. **Prepararse para Minecraft 1.22**:
   - Cuando se anuncie oficialmente
   - Verificar cambios en requisitos de Java
   - Actualizar Platform.kt si es necesario

2. **Revisar versiones obsoletas**:
   - Considerar deprecar soporte para versiones muy antiguas si es necesario
   - Documentar cualquier cambio de política de soporte

### Largo Plazo (6+ meses)

1. **Automatización**:
   - Considerar scripts para verificar automáticamente nuevas versiones de dependencias
   - Automatizar parte del proceso de actualización si es recurrente

2. **Documentación**:
   - Mantener este documento actualizado
   - Documentar cualquier cambio en el proceso

## Procedimiento de Actualización Propuesto

### Checklist para Agregar una Nueva Versión

- [ ] Verificar que Minecraft ha lanzado la nueva versión
- [ ] Esperar a que Yarn publique mappings (usualmente 1-2 días después)
- [ ] Esperar a que Forge publique una versión compatible (varía)
- [ ] Esperar a que NeoForge publique una versión compatible (varía)
- [ ] Calcular el código de versión (mcMajor * 10000 + mcMinor * 100 + mcPatch)
- [ ] Actualizar `loom.gradle.kts` con las nuevas entradas en el mapa
- [ ] Verificar si se necesita actualizar `Platform.kt` (requisitos de Java)
- [ ] Verificar si se necesita actualizar fabric-loader
- [ ] Probar la compilación con la nueva versión
- [ ] Incrementar la versión del toolkit
- [ ] Actualizar CHANGELOG (si existe)
- [ ] Crear commit y PR

## Archivos Clave

1. **`src/main/kotlin/gg/essential/defaults/loom.gradle.kts`**
   - Definiciones de todas las versiones soportadas
   - Mappings, Forge, NeoForge, Fabric Loader

2. **`src/main/kotlin/gg/essential/gradle/multiversion/Platform.kt`**
   - Lógica de detección de versión
   - Requisitos de Java por versión
   - Detección de loader

3. **`build.gradle.kts`**
   - Versión del toolkit
   - Dependencias del propio toolkit

## Conclusión

El Essential Gradle Toolkit actualmente soporta hasta **Minecraft 1.21.8** con un sistema bien estructurado y mantenible. Agregar soporte para nuevas versiones es un proceso directo que principalmente requiere:

1. Esperar a que las dependencias externas (Yarn, Forge, NeoForge) estén disponibles
2. Actualizar los mapas de versiones en `loom.gradle.kts`
3. Verificar requisitos de Java si es necesario

El sistema de revisiones permite mantener compatibilidad hacia atrás mientras se agregan nuevas funcionalidades, haciendo que el mantenimiento sea sostenible a largo plazo.
