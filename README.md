# mxp (Memoria por programa - Monitor de Uso de Memoria)
![MXP](mxp.webp)  
## Descripción

Este script bash permite monitorizar el uso de memoria de uno o varios programas en ejecución. Muestra información detallada sobre el consumo de memoria física (RSS) y virtual (VSZ) de cada proceso, con resultados formateados en columnas y resaltados con colores para facilitar la lectura.

## Características

- Monitoriza múltiples programas en una sola ejecución
- Muestra información detallada de cada proceso: PID, %MEM, RSS, VSZ y nombre del comando
- Calcula subtotales para programas con múltiples instancias
- Calcula el total global cuando se monitoriza más de un programa
- Formatea los valores de memoria en unidades legibles (KB, MB, GB, etc.)
- Presenta los resultados en columnas alineadas para mejor visualización
- Utiliza colores para destacar información importante (cuando se ejecuta en terminal)

## Requisitos

- Sistema operativo tipo Unix/Linux
- Bash shell
- Utilidades estándar: `ps`, `pgrep`, `numfmt`, `awk`

## Instalación

1. Descarga el script
2. Dale permisos de ejecución:

```bash
chmod +x mxp
```

## Uso

```bash
./mxp <programa1> [programa2] [programa3] ...
```

### Ejemplos

Para monitorizar un solo programa:
```bash
./mxp firefox
```

Para monitorizar varios programas a la vez:
```bash
./mxp firefox chrome spotify
```

## Herramientas utilizadas

El script utiliza varias herramientas estándar de Unix/Linux:

### `ps` (Process Status)

`ps` es una utilidad que muestra información sobre los procesos activos.

**Cómo se usa en el script:**
- `ps -p $pid -o pid,%mem,rss,vsz,comm --no-headers`: Obtiene información específica (PID, porcentaje de memoria, RSS, VSZ y nombre del comando) para un proceso específico sin mostrar encabezados.
- El parámetro `-o` permite seleccionar exactamente qué columnas de información queremos obtener.
- `--no-headers` elimina la línea de encabezados para facilitar el procesamiento.

### `pgrep` (Process Grep)

`pgrep` busca procesos activos según un patrón y devuelve sus PIDs.

**Cómo se usa en el script:**
- `pgrep -x "$PROGRAMA"`: Busca procesos cuyo nombre exacto coincida con el valor de la variable `$PROGRAMA`.
- La opción `-x` asegura que coincida exactamente con el nombre completo del proceso, no con subconjuntos.
- Se utiliza para verificar si un programa está en ejecución y para obtener los PIDs asociados a un programa específico.

### `numfmt`

`numfmt` es una utilidad para convertir números entre diferentes formatos, incluyendo unidades de tamaño legibles por humanos.

**Cómo se usa en el script:**
- `numfmt --to=iec --suffix=B --format="%.2f" $((rss * 1024))`: Convierte un valor en KB a un formato legible (KiB, MiB, GiB, etc.).
- `--to=iec`: Utiliza el estándar IEC para las unidades (KiB, MiB, GiB).
- `--suffix=B`: Añade "B" (bytes) al final de cada valor.
- `--format="%.2f"`: Muestra los valores con dos decimales.
- `$((rss * 1024))`: Convierte KB a bytes para que numfmt pueda procesarlo correctamente.

### `awk`

`awk` es un lenguaje de programación diseñado para procesar texto, especialmente adecuado para manipular datos estructurados en columnas.

**Cómo se usa en el script:**
- Para extraer campos específicos de la salida de `ps`:
  ```bash
  mem_info=$(ps -p $pid -o pid=,%mem=,rss=,vsz=,comm=)
  pid=$(echo $mem_info | awk '{print $1}')
  ```
- Para calcular sumas en subtotales:
  ```bash
  total_rss=$(ps -p $(echo $pids | tr ' ' ',') -o rss --no-headers | awk '{sum+=$1} END {print sum}')
  ```
  Aquí, `awk` suma todos los valores de la columna `$1` (RSS) y muestra el resultado al final.

## Flujo de ejecución del script

1. **Verificación de argumentos**: Comprueba si se ha proporcionado al menos un nombre de programa.
2. **Configuración de colores**: Define colores para terminales compatibles.
3. **Definición de formatos**: Establece formatos para alinear correctamente las columnas.
4. **Procesamiento de programas**:
   - Para cada programa especificado, verifica si está en ejecución usando `pgrep`.
   - Obtiene los PIDs asociados al programa.
   - Para cada PID, obtiene y muestra información detallada de memoria usando `ps`.
   - Si hay múltiples instancias, calcula subtotales usando `awk`.
5. **Cálculo de totales globales**: Si se han encontrado varios programas, muestra el total global de memoria utilizada.

## Salida

El script proporciona la siguiente información:

- **PID**: Identificador del proceso
- **%MEM**: Porcentaje de memoria física utilizada
- **RSS**: Resident Set Size - Memoria física realmente utilizada por el proceso (en formato legible)
- **VSZ**: Virtual Memory Size - Memoria virtual asignada al proceso (en formato legible)
- **COMANDO**: Nombre del proceso

Para programas con múltiples instancias, se muestra un subtotal del RSS y VSZ. Si se monitoriza más de un programa, se muestra un total global de la memoria consumida por todos los programas analizados.

## Explicación de los valores de memoria

- **RSS (Resident Set Size)**: La cantidad de memoria RAM física que está utilizando un proceso. Este es el valor más relevante para determinar el impacto real de un proceso en el sistema.

- **VSZ (Virtual Memory Size)**: La cantidad total de memoria virtual asignada a un proceso, incluyendo memoria compartida, memoria intercambiada (swap) y archivos mapeados en memoria. Este valor suele ser mayor que RSS y no necesariamente representa el uso real de recursos.

## Ejemplo de salida

```
Uso de memoria para los programas ...

PID      %MEM           RSS           VSZ COMANDO
------------------------------------------------
Programa: firefox

3721     3.2%       511.25MiB      1.72GiB firefox
3823     1.8%       287.42MiB    955.67MiB firefox
4022     0.7%       112.18MiB    780.45MiB firefox
  Subtotal para 'firefox':
  RSS total:     910.85MiB
  VSZ total:     3.43GiB
------------------------------------------------
Programa: chrome

8561     1.5%       245.33MiB      1.15GiB chrome
8602     0.6%        98.75MiB    670.22MiB chrome
  Subtotal para 'chrome':
  RSS total:     344.08MiB
  VSZ total:     1.80GiB
------------------------------------------------
TOTAL GLOBAL DE MEMORIA UTILIZADA POR TODOS LOS PROGRAMAS:
RSS total global:     1.23GiB
VSZ total global:     5.23GiB
----------------------------------------------------------

RSS (Resident Set Size): Memoria física utilizada
VSZ (Virtual Memory Size): Memoria virtual asignada
```

## Notas

- Si un programa especificado no está en ejecución, el script lo notificará.
- Los colores sólo se muestran cuando el script se ejecuta en un terminal interactivo.
- Para obtener información más detallada sobre el uso de memoria de un proceso específico, considere usar herramientas como `top`, `htop` o `smem`.

## Autor
Un par de chatbots de IA y Daniel Horacio Braga  
README generado por Anthropic Claude - https://www.anthropic.com/claude
## Licencia

Este script es de libre uso y distribución.
