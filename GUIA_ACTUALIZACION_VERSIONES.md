# Guía Rápida: Cómo Agregar Soporte para Nuevas Versiones de Minecraft

## Resumen

**Versión máxima actualmente soportada:** Minecraft 1.21.8 (código: 12108)

Esta guía explica cómo agregar soporte para versiones más recientes de Minecraft cuando estén disponibles.

## ¿Hasta qué versión se puede subir?

El toolkit puede soportar **cualquier versión futura de Minecraft** siempre y cuando existan las dependencias necesarias:

- ✅ **Yarn mappings** (para Fabric) - Disponible en https://maven.fabricmc.net/
- ✅ **Forge** (para Forge) - Disponible en https://files.minecraftforge.net/
- ✅ **NeoForge** (para NeoForge) - Disponible en https://maven.neoforged.net/

### Versiones Potenciales

| Versión | Código | Estado | Notas |
|---------|--------|--------|-------|
| 1.21.9 | 12109 | Esperando lanzamiento | Siguiente parche posible |
| 1.21.10 | 12110 | Esperando lanzamiento | Parche posterior |
| 1.22.0 | 12200 | Esperando lanzamiento | Próxima versión menor |

## Pasos para Agregar una Nueva Versión

### 1. Verificar Disponibilidad

Esperar a que estén disponibles:
- ✅ Lanzamiento oficial de Minecraft
- ✅ Yarn mappings (1-2 días después del lanzamiento)
- ✅ Forge compatible (tiempo variable)
- ✅ NeoForge compatible (tiempo variable)

### 2. Calcular Código de Versión

```
código = (major * 10000) + (minor * 100) + patch
```

**Ejemplo para Minecraft 1.21.9:**
```
código = (1 * 10000) + (21 * 100) + 9 = 12109
```

### 3. Editar `loom.gradle.kts`

Abrir: `src/main/kotlin/gg/essential/defaults/loom.gradle.kts`

Agregar las nuevas versiones en el **primer** bloque `Revision`:

```kotlin
revisions.add(Revision(
    yarn = mapOf(
        // ... versiones existentes ...
        12109 to "1.21.9+build.1:v2",  // ← AGREGAR AQUÍ
    ),
    forge = mapOf(
        // ... versiones existentes ...
        12109 to "1.21.9-XX.X.X",  // ← AGREGAR AQUÍ (con versión correcta)
    ),
    neoForge = mapOf(
        // ... versiones existentes ...
        12109 to "21.9.X-beta",  // ← AGREGAR AQUÍ (con versión correcta)
    ),
    // ... resto de la configuración ...
))
```

### 4. Verificar Requisitos de Java

Si la nueva versión requiere una versión diferente de Java, editar `Platform.kt`:

Abrir: `src/main/kotlin/gg/essential/gradle/multiversion/Platform.kt`

```kotlin
val javaVersion = when {
    // Agregar nueva línea si cambia requisito de Java
    mcVersion >= XXXXX -> JavaVersion.VERSION_XX
    mcVersion >= 12005 -> JavaVersion.VERSION_21  // Actual para 1.20.5+
    mcVersion >= 11800 -> JavaVersion.VERSION_17
    mcVersion >= 11700 -> JavaVersion.VERSION_16
    else -> JavaVersion.VERSION_1_8
}
```

### 5. Actualizar Versión del Toolkit

Editar `build.gradle.kts` (raíz del proyecto):

```kotlin
version = "0.6.11"  // Incrementar versión
```

### 6. Probar

```bash
./gradlew build
```

## Ejemplo Completo: Agregar Minecraft 1.21.9

Supongamos que queremos agregar soporte para Minecraft 1.21.9:

### Paso 1: Obtener información

- **Versión de Minecraft:** 1.21.9
- **Código:** 12109
- **Yarn mappings:** `1.21.9+build.1:v2` (ejemplo)
- **Forge:** `1.21.9-58.1.0` (ejemplo)
- **NeoForge:** `21.9.1-beta` (ejemplo)
- **Java requerido:** 21 (mismo que 1.20.5+, sin cambios necesarios)

### Paso 2: Editar loom.gradle.kts

```kotlin
revisions.add(Revision(
    yarn = mapOf(
        12109 to "1.21.9+build.1:v2",  // ← NUEVA LÍNEA
        12108 to "1.21.8+build.1:v2",
        12107 to "1.21.7+build.6:v2",
        // ... resto de versiones ...
    ),
    mcp = mapOf(
        // Sin cambios, versiones antiguas
    ),
    fabricLoader = "0.13.3",
    forge = mapOf(
        12109 to "1.21.9-58.1.0",  // ← NUEVA LÍNEA
        12108 to "1.21.8-58.0.0",
        12107 to "1.21.7-57.0.2",
        // ... resto de versiones ...
    ),
    neoForge = mapOf(
        12109 to "21.9.1-beta",  // ← NUEVA LÍNEA
        12108 to "21.8.5-beta",
        12107 to "21.7.11-beta",
        // ... resto de versiones ...
    )
))
```

### Paso 3: Actualizar build.gradle.kts

```kotlin
version = "0.6.11"
```

### Paso 4: Commit

```bash
git add .
git commit -m "Add support for Minecraft 1.21.9"
```

## Cambios Importantes según la Versión

### Cambios en Requisitos de Java

| Versión MC | Java Requerido | Cambio necesario en Platform.kt |
|------------|----------------|----------------------------------|
| 1.7.10 - 1.16.5 | Java 8 | No |
| 1.17.0 - 1.17.1 | Java 16 | No |
| 1.18.0 - 1.19.4 | Java 17 | No |
| 1.20.5+ | Java 21 | No |
| 1.22.0+ | ¿Java 21? | **Verificar al lanzarse** |

### Cambios en Mappings

- **Forge ≥ 1.17.0:** Usa mappings oficiales de Mojang automáticamente
- **Forge < 1.17.0:** Usa MCP mappings (ya configurado)
- **Fabric:** Siempre usa Yarn mappings

## Preguntas Frecuentes

### ¿Puedo agregar soporte para snapshots?

No es recomendable. El toolkit solo soporta versiones estables de lanzamiento.

### ¿Qué hago si Forge aún no está disponible?

Puedes agregar solo el soporte para Fabric/NeoForge y actualizar Forge más tarde creando una nueva revisión.

### ¿Necesito actualizar Fabric Loader?

Solo si las nuevas versiones de Minecraft lo requieren. Consulta la documentación de Fabric.

### ¿Cómo sé qué versiones de Yarn/Forge/NeoForge usar?

- **Yarn:** https://maven.fabricmc.net/net/fabricmc/yarn/
- **Forge:** https://files.minecraftforge.net/net/minecraftforge/forge/
- **NeoForge:** https://maven.neoforged.net/releases/net/neoforged/neoforge/

### ¿Qué es el sistema de "revisiones"?

Las revisiones permiten actualizar versiones existentes sin romper proyectos que usan versiones antiguas del toolkit. 

- **Agregar nueva versión:** Modificar el primer `Revision`
- **Actualizar versión existente:** Crear nueva revisión con `.update()`

## Archivos a Modificar

| Archivo | Cuándo modificar | Qué modificar |
|---------|------------------|---------------|
| `loom.gradle.kts` | Siempre | Agregar mappings/versiones |
| `Platform.kt` | Si cambia req. Java | Actualizar `javaVersion` |
| `build.gradle.kts` | Siempre | Incrementar versión del toolkit |

## Checklist Completo

- [ ] Nueva versión de Minecraft lanzada
- [ ] Yarn mappings disponible
- [ ] Forge disponible (opcional, puede agregarse después)
- [ ] NeoForge disponible (opcional, puede agregarse después)
- [ ] Código de versión calculado
- [ ] `loom.gradle.kts` actualizado
- [ ] `Platform.kt` actualizado (si necesario)
- [ ] `build.gradle.kts` versión incrementada
- [ ] Build exitoso (`./gradlew build`)
- [ ] Commit creado
- [ ] PR abierto

## Soporte

Para más detalles técnicos, consulta: `MINECRAFT_VERSION_SUPPORT_ANALYSIS.md`
