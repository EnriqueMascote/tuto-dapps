# Tutorial PASO A PASO: Programa MVC en Android Studio (Kotlin)
**App: Lista de Tareas Simple**  
**Android Studio:** 2025.1.3  
**API Level:** 29 (Android 10)  
**Lenguaje:** Kotlin

> **IMPORTANTE:** Sigue cada paso EN ORDEN. No te saltes pasos. Lee todos los comentarios del código.

---

## **PASO 1: CREAR EL PROYECTO**

**¿Qué vamos a hacer?** Crear un nuevo proyecto en Android Studio con la configuración correcta.

### Instrucciones detalladas:

1. **Abrir Android Studio 2025.1.3**
2. **Hacer clic en "Create New Project"**
3. **Seleccionar "Empty Views Activity"** (NO selecciones otra opción)
4. **Llenar los datos EXACTAMENTE así:**
   - **Name:** TareasApp
   - **Package name:** com.tuuniversidad.tareasapp
   - **Save location:** Tu carpeta de preferencia
   - **Language:** Kotlin (NO Java)
   - **Minimum API level:** API 29: Android 10.0
   - **Use legacy android.support libraries:** DESMARCADO (sin palomita)

5. **Hacer clic en "Finish"**
6. **Esperar que termine de cargar** (puede tardar varios minutos la primera vez)

---

## **PASO 2: CREAR LA ESTRUCTURA DE CARPETAS MVC**

**¿Qué vamos a hacer?** Organizar nuestro código siguiendo el patrón MVC creando carpetas separadas.

### Instrucciones paso a paso:

1. **En la vista del proyecto**, navegar a: `app/src/main/kotlin/com/tuuniversidad/tareasapp/`

2. **Crear las carpetas MVC:**
   - **Clic derecho** sobre la carpeta `tareasapp`
   - **New > Package**
   - Escribir: `modelo` y presionar Enter
   - **Repetir** para crear: `vista`
   - **Repetir** para crear: `controlador`

3. **Mover MainActivity:** 
   - **Arrastrar** el archivo `MainActivity.kt` a la carpeta `vista`
   - Cuando pregunte, seleccionar **"Refactor"**

**Tu estructura debe verse así:**
```
tareasapp/
├── modelo/          ← Aquí van los datos y lógica de negocio
├── vista/           ← Aquí van las pantallas y interfaces
├── controlador/     ← Aquí va la comunicación entre modelo y vista
└── MainActivity.kt (ya no debe estar aquí)
```

---

## **PASO 3: CREAR EL MODELO - CLASE TAREA**

**¿Qué vamos a hacer?** Esta clase representará cada tarea individual. Es como una "plantilla" que define qué información tiene una tarea (título, descripción, si está completada, etc.).

### Crear el archivo:

1. **Clic derecho** en la carpeta `modelo`
2. **New > Kotlin Class/File**
3. **Nombre:** `Tarea`
4. **Tipo:** Class
5. **Copiar y pegar este código COMPLETO:**

```kotlin
package com.tuuniversidad.tareasapp.modelo

/**
 * CLASE MODELO: Tarea
 * Esta clase representa una tarea individual usando un data class de Kotlin.
 * Contiene todos los datos que necesita una tarea.
 * En el patrón MVC, esta es parte del MODELO.
 */
data class Tarea(
    // ===== PROPIEDADES (Variables que guardan la información) =====
    val id: Int,                          // Número único para identificar la tarea
    var titulo: String,                   // El nombre/título de la tarea
    var descripcion: String,              // Descripción detallada de la tarea
    var completada: Boolean = false,      // true = completada, false = pendiente
    val fechaCreacion: Long = System.currentTimeMillis() // Cuándo se creó la tarea
) {
    
    // ===== MÉTODOS ADICIONALES =====
    
    /**
     * Cambia el estado de completada (alterna entre true/false)
     */
    fun alternarCompletada() {
        completada = !completada
    }
    
    /**
     * Verifica si la tarea fue creada hoy
     * @return true si la tarea se creó hoy
     */
    fun esDeHoy(): Boolean {
        val hoy = System.currentTimeMillis()
        val unDia = 24 * 60 * 60 * 1000 // milisegundos en un día
        return (hoy - fechaCreacion) < unDia
    }
    
    /**
     * Obtiene un resumen de la tarea para mostrar en la lista
     * @return texto con formato para mostrar
     */
    fun getResumen(): String {
        val estado = if (completada) "✅" else "⏳"
        return "$estado $titulo"
    }
    
    /**
     * Verifica si el título contiene una palabra específica
     * @param palabra - palabra a buscar
     * @return true si contiene la palabra (sin importar mayúsculas/minúsculas)
     */
    fun contienePalabra(palabra: String): Boolean {
        return titulo.contains(palabra, ignoreCase = true) || 
               descripcion.contains(palabra, ignoreCase = true)
    }
    
    /**
     * Método toString personalizado para debug y logging
     * @return representación en string de la tarea
     */
    override fun toString(): String {
        return "Tarea(id=$id, titulo='$titulo', completada=$completada)"
    }
}
```

---

## **PASO 4: CREAR EL MODELO - MANAGER DE TAREAS**

**¿Qué vamos a hacer?** Esta clase será como el "administrador" de todas las tareas. Se encarga de guardar la lista de tareas, agregar nuevas, eliminar, marcar como completadas, etc. Es el "cerebro" que maneja todos los datos.

### Crear el archivo:

1. **Clic derecho** en la carpeta `modelo`
2. **New > Kotlin Class/File**
3. **Nombre:** `TareaManager`
4. **Tipo:** Class
5. **Copiar y pegar este código COMPLETO:**

```kotlin
package com.tuuniversidad.tareasapp.modelo

/**
 * CLASE MODELO: TareaManager
 * Esta clase administra TODAS las tareas de la aplicación usando características modernas de Kotlin.
 * Es como un "gerente" que se encarga de guardar, buscar, agregar y eliminar tareas.
 * En el patrón MVC, esta es parte del MODELO (maneja los datos).
 */
class TareaManager {
    // ===== PROPIEDADES =====
    private val _tareas = mutableListOf<Tarea>()    // Lista mutable privada
    val tareas: List<Tarea> get() = _tareas         // Propiedad pública de solo lectura
    private var proximoId = 1                       // Contador para IDs únicos
    
    // ===== BLOQUE INIT (Constructor) =====
    /**
     * Se ejecuta cuando se crea una nueva instancia de TareaManager
     * Inicializa con algunas tareas de ejemplo
     */
    init {
        // Agregar tareas de ejemplo para que la app no esté vacía al inicio
        agregarTarea("Estudiar MVC", "Revisar conceptos de Modelo Vista Controlador")
        agregarTarea("Hacer ejercicio", "Rutina de 30 minutos de cardio")
        agregarTarea("Completar proyecto", "Terminar la evidencia de Android Studio")
    }
    
    // ===== MÉTODOS PÚBLICOS =====
    
    /**
     * AGREGAR una nueva tarea a la lista
     * @param titulo - Título de la nueva tarea
     * @param descripcion - Descripción de la nueva tarea
     * @return la nueva tarea creada
     */
    fun agregarTarea(titulo: String, descripcion: String = ""): Tarea {
        // Crear nueva tarea con el próximo ID disponible
        val nuevaTarea = Tarea(
            id = proximoId++,
            titulo = titulo.trim(),
            descripcion = descripcion.trim()
        )
        
        // Agregar a la lista
        _tareas.add(nuevaTarea)
        
        return nuevaTarea
    }
    
    /**
     * OBTENER todas las tareas (propiedad de solo lectura)
     * @return lista inmutable de todas las tareas
     */
    fun obtenerTareas(): List<Tarea> = tareas
    
    /**
     * ELIMINAR una tarea por su ID usando función de extensión de Kotlin
     * @param id - ID de la tarea que queremos eliminar
     * @return true si se eliminó, false si no se encontró
     */
    fun eliminarTarea(id: Int): Boolean {
        return _tareas.removeIf { it.id == id }
    }
    
    /**
     * CAMBIAR el estado de completada de una tarea
     * @param id - ID de la tarea que queremos cambiar
     * @return true si se encontró y cambió, false si no existe
     */
    fun alternarCompletada(id: Int): Boolean {
        // Usar find() de Kotlin para buscar la tarea
        val tarea = _tareas.find { it.id == id }
        return if (tarea != null) {
            tarea.alternarCompletada()
            true
        } else {
            false
        }
    }
    
    /**
     * BUSCAR una tarea específica por su ID usando función find()
     * @param id - ID de la tarea que estamos buscando
     * @return la tarea encontrada, o null si no existe
     */
    fun obtenerTareaPorId(id: Int): Tarea? {
        return _tareas.find { it.id == id }
    }
    
    /**
     * OBTENER el número total de tareas usando property
     */
    val totalTareas: Int
        get() = _tareas.size
    
    /**
     * OBTENER el número de tareas completadas usando función count()
     */
    val tareasCompletadas: Int
        get() = _tareas.count { it.completada }
    
    /**
     * OBTENER el número de tareas pendientes
     */
    val tareasPendientes: Int
        get() = _tareas.count { !it.completada }
    
    /**
     * BUSCAR tareas que contengan una palabra específica
     * @param palabra - palabra a buscar
     * @return lista de tareas que contienen la palabra
     */
    fun buscarTareas(palabra: String): List<Tarea> {
        return if (palabra.isBlank()) {
            tareas
        } else {
            _tareas.filter { it.contienePalabra(palabra) }
        }
    }
    
    /**
     * OBTENER solo las tareas completadas
     * @return lista de tareas completadas
     */
    fun obtenerTareasCompletadas(): List<Tarea> {
        return _tareas.filter { it.completada }
    }
    
    /**
     * OBTENER solo las tareas pendientes
     * @return lista de tareas pendientes
     */
    fun obtenerTareasPendientes(): List<Tarea> {
        return _tareas.filter { !it.completada }
    }
    
    /**
     * LIMPIAR todas las tareas completadas
     * @return número de tareas eliminadas
     */
    fun limpiarCompletadas(): Int {
        val cantidadAntes = _tareas.size
        _tareas.removeIf { it.completada }
        return cantidadAntes - _tareas.size
    }
    
    /**
     * OBTENER estadísticas como string formateado
     * @return texto con estadísticas
     */
    fun obtenerEstadisticas(): String {
        return "Total: $totalTareas | Completadas: $tareasCompletadas | Pendientes: $tareasPendientes"
    }
    
    /**
     * VERIFICAR si hay tareas
     * @return true si no hay tareas
     */
    fun estaVacia(): Boolean = _tareas.isEmpty()
    
    /**
     * OBTENER la tarea más reciente
     * @return la última tarea agregada o null si no hay tareas
     */
    fun obtenerTareaMasReciente(): Tarea? {
        return _tareas.maxByOrNull { it.fechaCreacion }
    }
}
```

---

## **PASO 5: CREAR EL CONTROLADOR**

**¿Qué vamos a hacer?** El controlador es como el "traductor" entre la pantalla (Vista) y los datos (Modelo). Cuando el usuario presiona un botón en la pantalla, el controlador se encarga de decirle al modelo qué hacer, y luego le dice a la vista que se actualice.

### Crear el archivo:

1. **Clic derecho** en la carpeta `controlador`
2. **New > Kotlin Class/File**
3. **Nombre:** `TareaControlador`
4. **Tipo:** Class
5. **Copiar y pegar este código COMPLETO:**

```kotlin
package com.tuuniversidad.tareasapp.controlador

import com.tuuniversidad.tareasapp.modelo.Tarea
import com.tuuniversidad.tareasapp.modelo.TareaManager
import com.tuuniversidad.tareasapp.vista.MainActivity

/**
 * CLASE CONTROLADOR: TareaControlador
 * Esta clase es el "intermediario" entre la Vista (pantalla) y el Modelo (datos).
 * Usa características modernas de Kotlin como funciones lambda y propiedades.
 * En el patrón MVC, esta es el CONTROLADOR.
 */
class TareaControlador(private val vista: MainActivity) {
    // ===== PROPIEDADES =====
    private val modelo = TareaManager()    // Instancia del administrador de datos
    
    // Callback para notificar cambios a la vista
    var onDataChanged: (() -> Unit)? = null
    
    // ===== MÉTODOS PÚBLICOS (API del Controlador) =====
    
    /**
     * OBTENER la lista de todas las tareas
     * @return lista inmutable de tareas
     */
    fun obtenerListaTareas(): List<Tarea> = modelo.obtenerTareas()
    
    /**
     * AGREGAR una nueva tarea con validación
     * @param titulo - Título que escribió el usuario
     * @param descripcion - Descripción que escribió el usuario (opcional)
     * @return resultado de la operación
     */
    fun agregarNuevaTarea(titulo: String, descripcion: String = ""): ResultadoOperacion {
        return when {
            titulo.isBlank() -> {
                ResultadoOperacion.Error("El título no puede estar vacío")
            }
            titulo.length > 100 -> {
                ResultadoOperacion.Error("El título es demasiado largo (máximo 100 caracteres)")
            }
            else -> {
                try {
                    val nuevaTarea = modelo.agregarTarea(titulo, descripcion)
                    notificarCambio()
                    ResultadoOperacion.Exitoso("Tarea agregada: ${nuevaTarea.titulo}")
                } catch (e: Exception) {
                    ResultadoOperacion.Error("Error al agregar tarea: ${e.message}")
                }
            }
        }
    }
    
    /**
     * CAMBIAR el estado de una tarea (completada/pendiente)
     * @param id - ID de la tarea que se quiere cambiar
     * @return resultado de la operación
     */
    fun alternarCompletada(id: Int): ResultadoOperacion {
        return if (modelo.alternarCompletada(id)) {
            val tarea = modelo.obtenerTareaPorId(id)
            val estado = if (tarea?.completada == true) "completada" else "pendiente"
            notificarCambio()
            ResultadoOperacion.Exitoso("Tarea marcada como $estado")
        } else {
            ResultadoOperacion.Error("No se encontró la tarea")
        }
    }
    
    /**
     * ELIMINAR una tarea
     * @param id - ID de la tarea que se quiere eliminar
     * @return resultado de la operación
     */
    fun eliminarTarea(id: Int): ResultadoOperacion {
        val tarea = modelo.obtenerTareaPorId(id)
        return if (modelo.eliminarTarea(id)) {
            notificarCambio()
            ResultadoOperacion.Exitoso("Tarea eliminada: ${tarea?.titulo ?: "Desconocida"}")
        } else {
            ResultadoOperacion.Error("No se pudo eliminar la tarea")
        }
    }
    
    /**
     * OBTENER los detalles de una tarea específica
     * @param id - ID de la tarea que queremos ver
     * @return la tarea con sus detalles, o null si no existe
     */
    fun obtenerDetalleTarea(id: Int): Tarea? = modelo.obtenerTareaPorId(id)
    
    /**
     * OBTENER estadísticas formateadas
     * @return string con estadísticas actuales
     */
    fun obtenerEstadisticas(): String = modelo.obtenerEstadisticas()
    
    /**
     * BUSCAR tareas por palabra clave
     * @param busqueda - texto a buscar
     * @return lista de tareas que coinciden con la búsqueda
     */
    fun buscarTareas(busqueda: String): List<Tarea> = modelo.buscarTareas(busqueda)
    
    /**
     * LIMPIAR todas las tareas completadas
     * @return resultado de la operación con cantidad eliminada
     */
    fun limpiarCompletadas(): ResultadoOperacion {
        val cantidad = modelo.limpiarCompletadas()
        return if (cantidad > 0) {
            notificarCambio()
            ResultadoOperacion.Exitoso("$cantidad tareas completadas eliminadas")
        } else {
            ResultadoOperacion.Info("No hay tareas completadas para eliminar")
        }
    }
    
    /**
     * OBTENER solo tareas completadas
     */
    fun obtenerTareasCompletadas(): List<Tarea> = modelo.obtenerTareasCompletadas()
    
    /**
     * OBTENER solo tareas pendientes
     */
    fun obtenerTareasPendientes(): List<Tarea> = modelo.obtenerTareasPendientes()
    
    /**
     * VERIFICAR si hay tareas
     */
    fun hayTareas(): Boolean = !modelo.estaVacia()
    
    // ===== MÉTODOS PRIVADOS =====
    
    /**
     * Notifica a la vista que los datos han cambiado
     */
    private fun notificarCambio() {
        vista.actualizarLista()
        onDataChanged?.invoke()
    }
    
    // ===== CLASE SELLADA para resultados de operaciones =====
    /**
     * Representa el resultado de una operación del controlador
     */
    sealed class ResultadoOperacion(val mensaje: String) {
        class Exitoso(mensaje: String) : ResultadoOperacion(mensaje)
        class Error(mensaje: String) : ResultadoOperacion(mensaje)
        class Info(mensaje: String) : ResultadoOperacion(mensaje)
        
        val esExitoso: Boolean get() = this is Exitoso
        val esError: Boolean get() = this is Error
    }
}
```

---

## **PASO 6: CREAR LOS LAYOUTS (DISEÑOS DE PANTALLA)**

**¿Qué vamos a hacer?** Ahora vamos a diseñar cómo se verá nuestra aplicación. Crearemos el diseño de la pantalla principal y el diseño de cada elemento de la lista.

### Paso 6.1: Diseño de la pantalla principal

1. **Abrir** el archivo `res/layout/activity_main.xml`
2. **Borrar TODO el contenido** del archivo
3. **Copiar y pegar este código COMPLETO:**

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 
LAYOUT PRINCIPAL: activity_main.xml
Este archivo define cómo se ve la pantalla principal de nuestra app.
Utiliza ViewBinding para conectar con Kotlin de manera segura.
-->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    android:background="#f8f9fa">

    <!-- ===== TÍTULO DE LA APP ===== -->
    <com.google.android.material.card.MaterialCardView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="16dp"
        app:cardElevation="4dp"
        app:cardCornerRadius="12dp"
        xmlns:app="http://schemas.android.com/apk/res-auto">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="📝 Lista de Tareas - MVC Kotlin"
            android:textSize="22sp"
            android:textStyle="bold"
            android:textColor="#2c3e50"
            android:gravity="center"
            android:padding="20dp"
            android:background="#ffffff" />

    </com.google.android.material.card.MaterialCardView>

    <!-- ===== SECCIÓN PARA AGREGAR NUEVA TAREA ===== -->
    <com.google.android.material.card.MaterialCardView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="16dp"
        app:cardElevation="2dp"
        app:cardCornerRadius="8dp">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="16dp">

            <!-- Campo de texto para el título -->
            <com.google.android.material.textfield.TextInputLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="Título de la tarea"
                app:boxStrokeColor="#3498db"
                style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox">

                <com.google.android.material.textfield.TextInputEditText
                    android:id="@+id/editTextTitulo"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:inputType="text"
                    android:maxLines="2"
                    android:maxLength="100" />

            </com.google.android.material.textfield.TextInputLayout>

            <!-- Botones -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:layout_marginTop="12dp">

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/buttonAgregar"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:text="➕ Agregar Tarea"
                    android:textSize="16sp"
                    app:backgroundTint="#27ae60"
                    android:textColor="#ffffff"
                    android:layout_marginEnd="8dp" />

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/buttonLimpiar"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="🧹"
                    app:backgroundTint="#e74c3c"
                    android:textColor="#ffffff"
                    style="@style/Widget.MaterialComponents.Button.OutlinedButton" />

            </LinearLayout>

        </LinearLayout>

    </com.google.android.material.card.MaterialCardView>

    <!-- ===== ESTADÍSTICAS ===== -->
    <TextView
        android:id="@+id/textViewEstadisticas"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Total: 0 | Completadas: 0 | Pendientes: 0"
        android:textSize="14sp"
        android:textColor="#7f8c8d"
        android:gravity="center"
        android:layout_marginBottom="12dp"
        android:padding="8dp"
        android:background="#ecf0f1"
        android:textStyle="bold" />

    <!-- ===== LISTA DE TAREAS ===== -->
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerViewTareas"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:clipToPadding="false"
        android:paddingBottom="8dp" />

    <!-- ===== MENSAJE CUANDO NO HAY TAREAS ===== -->
    <LinearLayout
        android:id="@+id/layoutVacio"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="vertical"
        android:gravity="center"
        android:visibility="gone">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="📝"
            android:textSize="64sp"
            android:layout_marginBottom="16dp" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="¡No hay tareas!"
            android:textSize="20sp"
            android:textColor="#7f8c8d"
            android:textStyle="bold" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Agrega tu primera tarea arriba"
            android:textSize="16sp"
            android:textColor="#95a5a6"
            android:layout_marginTop="8dp" />

    </LinearLayout>

</LinearLayout>
```

### Paso 6.2: Diseño de cada elemento de la lista

1. **Clic derecho** en la carpeta `res/layout`
2. **New > Layout Resource File**
3. **File name:** `item_tarea`
4. **Root element:** `LinearLayout`
5. **Hacer clic OK**
6. **Borrar todo el contenido** del archivo creado
7. **Copiar y pegar este código COMPLETO:**

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 
LAYOUT DE ELEMENTO: item_tarea.xml
Este archivo define cómo se ve CADA tarea individual en la lista.
Usa Material Design con CardView para un aspecto moderno.
-->
<com.google.android.material.card.MaterialCardView 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="4dp"
    app:cardElevation="2dp"
    app:cardCornerRadius="8dp"
    android:id="@+id/cardViewTarea">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="16dp"
        android:gravity="center_vertical">

        <!-- ===== CHECKBOX (para marcar como completada) ===== -->
        <CheckBox
            android:id="@+id/checkBoxCompletada"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginEnd="16dp"
            app:buttonTint="#3498db" />

        <!-- ===== INFORMACIÓN DE LA TAREA ===== -->
        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:orientation="vertical">

            <!-- Título de la tarea -->
            <TextView
                android:id="@+id/textViewTitulo"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Título de la tarea"
                android:textSize="16sp"
                android:textStyle="bold"
                android:textColor="#2c3e50"
                android:maxLines="2"
                android:ellipsize="end" />

            <!-- Descripción de la tarea -->
            <TextView
                android:id="@+id/textViewDescripcion"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Descripción de la tarea"
                android:textSize="14sp"
                android:textColor="#7f8c8d"
                android:layout_marginTop="4dp"
                android:maxLines="3"
                android:ellipsize="end" />

            <!-- Información adicional -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:layout_marginTop="8dp">

                <!-- ID de la tarea -->
                <TextView
                    android:id="@+id/textViewId"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="ID: 1"
                    android:textSize="12sp"
                    android:textColor="#95a5a6"
                    android:background="#ecf0f1"
                    android:padding="4dp"
                    android:textStyle="bold" />

                <!-- Estado visual -->
                <TextView
                    android:id="@+id/textViewEstado"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="⏳ Pendiente"
                    android:textSize="12sp"
                    android:textColor="#e67e22"
                    android:layout_marginStart="8dp"
                    android:padding="4dp"
                    android:textStyle="bold" />

            </LinearLayout>

        </LinearLayout>

        <!-- ===== BOTÓN ELIMINAR ===== -->
        <com.google.android.material.button.MaterialButton
            android:id="@+id/buttonEliminar"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="🗑️"
            android:textSize="18sp"
            app:backgroundTint="#e74c3c"
            android:textColor="#ffffff"
            android:minWidth="48dp"
            android:minHeight="48dp"
            style="@style/Widget.MaterialComponents.Button.UnelevatedButton" />

    </LinearLayout>

</com.google.android.material.card.MaterialCardView>
```

---

## **PASO 7: CREAR EL ADAPTER CON RECYCLERVIEW (MODERNO)**

**¿Qué vamos a hacer?** En lugar del ListView tradicional, usaremos RecyclerView que es más moderno y eficiente. El Adapter en Kotlin usa características como ViewBinding y funciones lambda para un código más limpio.

### Crear el archivo:

1. **Clic derecho** en la carpeta `vista`
2. **New > Kotlin Class/File**
3. **Nombre:** `TareaAdapter`
4. **Tipo:** Class
5. **Copiar y pegar este código COMPLETO:**

```kotlin
package com.tuuniversidad.tareasapp.vista

import android.graphics.Paint
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.core.content.ContextCompat
import androidx.recyclerview.widget.DiffUtil
import androidx.recyclerview.widget.ListAdapter
import androidx.recyclerview.widget.RecyclerView
import com.google.android.material.button.MaterialButton
import com.google.android.material.card.MaterialCardView
import com.tuuniversidad.tareasapp.R
import com.tuuniversidad.tareasapp.modelo.Tarea

/**
 * CLASE VISTA: TareaAdapter
 * Adapter moderno que usa RecyclerView y DiffUtil para mejor performance.
 * Utiliza características de Kotlin como funciones lambda y propiedades.
 * En el patrón MVC, esta es parte de la VISTA.
 */
class TareaAdapter : ListAdapter<Tarea, TareaAdapter.TareaViewHolder>(TareaDiffCallback()) {
    
    // ===== INTERFACES PARA COMUNICACIÓN CON ACTIVITY =====
    
    /**
     * Interface funcional para manejar clicks en las tareas
     */
    fun interface OnTareaClickListener {
        fun onTareaClick(tarea: Tarea)
    }
    
    /**
     * Interface funcional para manejar cambios en el checkbox
     */
    fun interface OnCompletadaChangeListener {
        fun onCompletadaChange(tareaId: Int)
    }
    
    /**
     * Interface funcional para manejar eliminación de tareas
     */
    fun interface OnEliminarTareaListener {
        fun onEliminarTarea(tareaId: Int)
    }
    
    // ===== PROPIEDADES DE LISTENERS =====
    var onTareaClickListener: OnTareaClickListener? = null
    var onCompletadaChangeListener: OnCompletadaChangeListener? = null
    var onEliminarTareaListener: OnEliminarTareaListener? = null
    
    // ===== MÉTODOS DEL ListAdapter =====
    
    /**
     * Crea un nuevo ViewHolder cuando RecyclerView lo necesita
     */
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): TareaViewHolder {
        val itemView = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_tarea, parent, false)
        return TareaViewHolder(itemView)
    }
    
    /**
     * Vincula los datos de una tarea con un ViewHolder
     */
    override fun onBindViewHolder(holder: TareaViewHolder, position: Int) {
        val tarea = getItem(position)
        holder.bind(tarea)
    }
    
    // ===== CLASE ViewHolder =====
    
    /**
     * ViewHolder que contiene las vistas de cada elemento de la lista
     * Usa ViewBinding manual para mejor performance
     */
    inner class TareaViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        
        // ===== VISTAS DEL ITEM =====
        private val cardView: MaterialCardView = itemView.findViewById(R.id.cardViewTarea)
        private val checkBox: CheckBox = itemView.findViewById(R.id.checkBoxCompletada)
        private val textTitulo: TextView = itemView.findViewById(R.id.textViewTitulo)
        private val textDescripcion: TextView = itemView.findViewById(R.id.textViewDescripcion)
        private val textId: TextView = itemView.findViewById(R.id.textViewId)
        private val textEstado: TextView = itemView.findViewById(R.id.textViewEstado)
        private val buttonEliminar: MaterialButton = itemView.findViewById(R.id.buttonEliminar)
        
        /**
         * Vincula una tarea con las vistas del ViewHolder
         */
        fun bind(tarea: Tarea) {
            // ===== CONFIGURAR DATOS BÁSICOS =====
            textTitulo.text = tarea.titulo
            textDescripcion.text = if (tarea.descripcion.isNotBlank()) {
                tarea.descripcion
            } else {
                "Sin descripción"
            }
            textId.text = "ID: ${tarea.id}"
            checkBox.isChecked = tarea.completada
            
            // ===== CONFIGURAR ESTADO VISUAL =====
            configurarEstadoVisual(tarea)
            
            // ===== CONFIGURAR EVENTOS =====
            configurarEventos(tarea)
        }
        
        /**
         * Configura el aspecto visual según el estado de la tarea
         */
        private fun configurarEstadoVisual(tarea: Tarea) {
            if (tarea.completada) {
                // ===== TAREA COMPLETADA =====
                // Texto tachado
                textTitulo.paintFlags = textTitulo.paintFlags or Paint.STRIKE_THRU_TEXT_FLAG
                textDescripcion.paintFlags = textDescripcion.paintFlags or Paint.STRIKE_THRU_TEXT_FLAG
                
                // Colores para completada
                textTitulo.setTextColor(ContextCompat.getColor(itemView.context, android.R.color.darker_gray))
                textDescripcion.setTextColor(ContextCompat.getColor(itemView.context, android.R.color.darker_gray))
                
                // Estado visual
                textEstado.text = "✅ Completada"
                textEstado.setTextColor(ContextCompat.getColor(itemView.context, android.R.color.holo_green_dark))
                
                // Card con menor elevación
                cardView.cardElevation = 1f
                cardView.alpha = 0.7f
                
            } else {
                // ===== TAREA PENDIENTE =====
                // Texto normal (sin tachado)
                textTitulo.paintFlags = textTitulo.paintFlags and Paint.STRIKE_THRU_TEXT_FLAG.inv()
                textDescripcion.paintFlags = textDescripcion.paintFlags and Paint.STRIKE_THRU_TEXT_FLAG.inv()
                
                // Colores normales
                textTitulo.setTextColor(ContextCompat.getColor(itemView.context, android.R.color.black))
                textDescripcion.setTextColor(ContextCompat.getColor(itemView.context, android.R.color.darker_gray))
                
                // Estado visual
                textEstado.text = "⏳ Pendiente"
                textEstado.setTextColor(ContextCompat.getColor(itemView.context, android.R.color.holo_orange_dark))
                
                // Card con elevación normal
                cardView.cardElevation = 2f
                cardView.alpha = 1f
            }
        }
        
        /**
         * Configura todos los eventos de click e interacción
         */
        private fun configurarEventos(tarea: Tarea) {
            // ===== CLICK EN EL CARD COMPLETO =====
            cardView.setOnClickListener {
                onTareaClickListener?.onTareaClick(tarea)
            }
            
            // ===== CLICK EN EL CHECKBOX =====
            // Remover listener anterior para evitar loops
            checkBox.setOnCheckedChangeListener(null)
            // Configurar nuevo listener
            checkBox.setOnCheckedChangeListener { _, isChecked ->
                // Solo procesar si el estado cambió realmente
                if (isChecked != tarea.completada) {
                    onCompletadaChangeListener?.onCompletadaChange(tarea.id)
                }
            }
            
            // ===== CLICK EN BOTÓN ELIMINAR =====
            buttonEliminar.setOnClickListener {
                onEliminarTareaListener?.onEliminarTarea(tarea.id)
            }
            
            // ===== LONG CLICK PARA ACCIONES ADICIONALES =====
            cardView.setOnLongClickListener {
                // Aquí podrías agregar un menú contextual o acción adicional
                true // Retorna true para indicar que el evento fue manejado
            }
        }
    }
    
    // ===== MÉTODOS DE UTILIDAD =====
    
    /**
     * Obtiene una tarea por su posición
     */
    fun getTareaAt(position: Int): Tarea? {
        return if (position in 0 until itemCount) {
            getItem(position)
        } else {
            null
        }
    }
}

// ===== CLASE PARA COMPARAR TAREAS (DiffUtil) =====

/**
 * DiffCallback para optimizar las actualizaciones del RecyclerView
 * Solo actualiza los elementos que realmente cambiaron
 */
class TareaDiffCallback : DiffUtil.ItemCallback<Tarea>() {
    
    /**
     * Verifica si son el mismo elemento (mismo ID)
     */
    override fun areItemsTheSame(oldItem: Tarea, newItem: Tarea): Boolean {
        return oldItem.id == newItem.id
    }
    
    /**
     * Verifica si el contenido es el mismo
     */
    override fun areContentsTheSame(oldItem: Tarea, newItem: Tarea): Boolean {
        return oldItem == newItem
    }
    
    /**
     * Obtiene el payload para actualización parcial (opcional)
     */
    override fun getChangePayload(oldItem: Tarea, newItem: Tarea): Any? {
        return when {
            oldItem.completada != newItem.completada -> "COMPLETADA_CHANGED"
            oldItem.titulo != newItem.titulo -> "TITULO_CHANGED"
            oldItem.descripcion != newItem.descripcion -> "DESCRIPCION_CHANGED"
            else -> null
        }
    }
}
```

---

## **PASO 8: CREAR LA ACTIVITY PRINCIPAL (KOTLIN MODERNO)**

**¿Qué vamos a hacer?** Esta es la pantalla principal usando Kotlin moderno con ViewBinding, corrutinas y características avanzadas del lenguaje.

### Modificar MainActivity:

1. **Abrir** el archivo `vista/MainActivity.kt`
2. **Borrar TODO el contenido** del archivo
3. **Copiar y pegar este código COMPLETO:**

```kotlin
package com.tuuniversidad.tareasapp.vista

import android.os.Bundle
import android.view.View
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.ItemTouchHelper
import androidx.recyclerview.widget.RecyclerView
import com.google.android.material.snackbar.Snackbar
import com.tuuniversidad.tareasapp.R
import com.tuuniversidad.tareasapp.controlador.TareaControlador
import com.tuuniversidad.tareasapp.modelo.Tarea

/**
 * CLASE VISTA: MainActivity
 * Activity principal que implementa una interfaz moderna usando Kotlin.
 * Utiliza RecyclerView, Material Design y gestión moderna del estado.
 * En el patrón MVC, esta es la VISTA principal.
 */
class MainActivity : AppCompatActivity() {
    
    // ===== PROPIEDADES =====
    private lateinit var controlador: TareaControlador
    private lateinit var adapter: TareaAdapter
    
    // Views (sin ViewBinding para simplicidad en el tutorial)
    private lateinit var editTextTitulo: com.google.android.material.textfield.TextInputEditText
    private lateinit var buttonAgregar: com.google.android.material.button.MaterialButton
    private lateinit var buttonLimpiar: com.google.android.material.button.MaterialButton
    private lateinit var recyclerViewTareas: RecyclerView
    private lateinit var textViewEstadisticas: android.widget.TextView
    private lateinit var layoutVacio: android.widget.LinearLayout
    
    // ===== CICLO DE VIDA =====
    
    /**
     * Se ejecuta cuando se crea la Activity
     */
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // Inicialización en orden
        inicializarControlador()
        inicializarVistas()
        configurarRecyclerView()
        configurarEventos()
        actualizarInterfaz()
        
        // Mostrar mensaje de bienvenida
        mostrarMensaje("¡Bienvenido a tu Lista de Tareas!", TipoMensaje.INFO)
    }
    
    /**
     * Se ejecuta cuando la Activity se reanuda
     */
    override fun onResume() {
        super.onResume()
        actualizarInterfaz()
    }
    
    // ===== MÉTODOS DE INICIALIZACIÓN =====
    
    /**
     * Inicializa el controlador MVC
     */
    private fun inicializarControlador() {
        controlador = TareaControlador(this)
        
        // Configurar callback para cambios de datos
        controlador.onDataChanged = {
            actualizarEstadisticas()
        }
    }
    
    /**
     * Inicializa todas las vistas
     */
    private fun inicializarVistas() {
        editTextTitulo = findViewById(R.id.editTextTitulo)
        buttonAgregar = findViewById(R.id.buttonAgregar)
        buttonLimpiar = findViewById(R.id.buttonLimpiar)
        recyclerViewTareas = findViewById(R.id.recyclerViewTareas)
        textViewEstadisticas = findViewById(R.id.textViewEstadisticas)
        layoutVacio = findViewById(R.id.layoutVacio)
    }
    
    /**
     * Configura el RecyclerView y su adapter
     */
    private fun configurarRecyclerView() {
        // Crear y configurar adapter
        adapter = TareaAdapter().apply {
            // Configurar listeners usando SAM (Single Abstract Method)
            onTareaClickListener = TareaAdapter.OnTareaClickListener { tarea ->
                mostrarDetallesTarea(tarea)
            }
            
            onCompletadaChangeListener = TareaAdapter.OnCompletadaChangeListener { tareaId ->
                manejarCambioCompletada(tareaId)
            }
            
            onEliminarTareaListener = TareaAdapter.OnEliminarTareaListener { tareaId ->
                manejarEliminarTarea(tareaId)
            }
        }
        
        // Configurar RecyclerView
        recyclerViewTareas.apply {
            this.adapter = this@MainActivity.adapter
            layoutManager = LinearLayoutManager(this@MainActivity)
            setHasFixedSize(true)
        }
        
        // Agregar ItemTouchHelper para swipe to delete
        configurarSwipeToDelete()
    }
    
    /**
     * Configura gestos de deslizar para eliminar
     */
    private fun configurarSwipeToDelete() {
        val itemTouchHelper = ItemTouchHelper(object : ItemTouchHelper.SimpleCallback(
            0, ItemTouchHelper.LEFT or ItemTouchHelper.RIGHT
        ) {
            override fun onMove(
                recyclerView: RecyclerView,
                viewHolder: RecyclerView.ViewHolder,
                target: RecyclerView.ViewHolder
            ): Boolean = false
            
            override fun onSwiped(viewHolder: RecyclerView.ViewHolder, direction: Int) {
                val position = viewHolder.adapterPosition
                val tarea = adapter.getTareaAt(position)
                tarea?.let {
                    manejarEliminarTarea(it.id)
                }
            }
        })
        
        itemTouchHelper.attachToRecyclerView(recyclerViewTareas)
    }
    
    /**
     * Configura todos los eventos de la interfaz
     */
    private fun configurarEventos() {
        // ===== BOTÓN AGREGAR =====
        buttonAgregar.setOnClickListener {
            manejarAgregarTarea()
        }
        
        // ===== BOTÓN LIMPIAR COMPLETADAS =====
        buttonLimpiar.setOnClickListener {
            manejarLimpiarCompletadas()
        }
        
        // ===== ENTER EN CAMPO DE TEXTO =====
        editTextTitulo.setOnEditorActionListener { _, _, _ ->
            manejarAgregarTarea()
            true
        }
    }
    
    // ===== MÉTODOS DE MANEJO DE EVENTOS =====
    
    /**
     * Maneja la adición de una nueva tarea
     */
    private fun manejarAgregarTarea() {
        val titulo = editTextTitulo.text?.toString()?.trim() ?: ""
        
        when (val resultado = controlador.agregarNuevaTarea(titulo)) {
            is TareaControlador.ResultadoOperacion.Exitoso -> {
                editTextTitulo.text?.clear()
                mostrarMensaje(resultado.mensaje, TipoMensaje.EXITO)
                actualizarInterfaz()
            }
            is TareaControlador.ResultadoOperacion.Error -> {
                mostrarMensaje(resultado.mensaje, TipoMensaje.ERROR)
            }
            is TareaControlador.ResultadoOperacion.Info -> {
                mostrarMensaje(resultado.mensaje, TipoMensaje.INFO)
            }
        }
    }
    
    /**
     * Maneja el cambio de estado completado/pendiente
     */
    private fun manejarCambioCompletada(tareaId: Int) {
        when (val resultado = controlador.alternarCompletada(tareaId)) {
            is TareaControlador.ResultadoOperacion.Exitoso -> {
                mostrarMensaje(resultado.mensaje, TipoMensaje.EXITO)
                actualizarInterfaz()
            }
            is TareaControlador.ResultadoOperacion.Error -> {
                mostrarMensaje(resultado.mensaje, TipoMensaje.ERROR)
                // Revertir cambio en la interfaz
                actualizarLista()
            }
            else -> {
                actualizarInterfaz()
            }
        }
    }
    
    /**
     * Maneja la eliminación de una tarea
     */
    private fun manejarEliminarTarea(tareaId: Int) {
        // Obtener datos de la tarea antes de eliminar para el undo
        val tarea = controlador.obtenerDetalleTarea(tareaId)
        
        when (val resultado = controlador.eliminarTarea(tareaId)) {
            is TareaControlador.ResultadoOperacion.Exitoso -> {
                actualizarInterfaz()
                
                // Mostrar Snackbar con opción de deshacer
                tarea?.let { tareaEliminada ->
                    mostrarSnackbarUndo(resultado.mensaje, tareaEliminada)
                }
            }
            is TareaControlador.ResultadoOperacion.Error -> {
                mostrarMensaje(resultado.mensaje, TipoMensaje.ERROR)
            }
            else -> {
                actualizarInterfaz()
            }
        }
    }
    
    /**
     * Maneja la limpieza de tareas completadas
     */
    private fun manejarLimpiarCompletadas() {
        when (val resultado = controlador.limpiarCompletadas()) {
            is TareaControlador.ResultadoOperacion.Exitoso -> {
                mostrarMensaje(resultado.mensaje, TipoMensaje.EXITO)
                actualizarInterfaz()
            }
            is TareaControlador.ResultadoOperacion.Info -> {
                mostrarMensaje(resultado.mensaje, TipoMensaje.INFO)
            }
            is TareaControlador.ResultadoOperacion.Error -> {
                mostrarMensaje(resultado.mensaje, TipoMensaje.ERROR)
            }
        }
    }
    
    /**
     * Muestra los detalles de una tarea en un Toast
     */
    private fun mostrarDetallesTarea(tarea: Tarea) {
        val detalles = buildString {
            appendLine("📋 ${tarea.titulo}")
            if (tarea.descripcion.isNotBlank()) {
                appendLine("📝 ${tarea.descripcion}")
            }
            appendLine("🆔 ID: ${tarea.id}")
            appendLine("📅 ${if (tarea.esDeHoy()) "Creada hoy" else "Creada anteriormente"}")
            append("✅ ${if (tarea.completada) "Completada" else "Pendiente"}")
        }
        
        Toast.makeText(this, detalles, Toast.LENGTH_LONG).show()
    }
    
    // ===== MÉTODOS PÚBLICOS (LLAMADOS DESDE EL CONTROLADOR) =====
    
    /**
     * Actualiza la lista de tareas (llamado desde el controlador)
     */
    fun actualizarLista() {
        val tareas = controlador.obtenerListaTareas()
        adapter.submitList(tareas.toList()) // Crear nueva lista para trigger DiffUtil
    }
    
    /**
     * Actualiza toda la interfaz
     */
    private fun actualizarInterfaz() {
        actualizarLista()
        actualizarEstadisticas()
        actualizarVisibilidadVistas()
    }
    
    /**
     * Actualiza las estadísticas mostradas
     */
    private fun actualizarEstadisticas() {
        textViewEstadisticas.text = controlador.obtenerEstadisticas()
    }
    
    /**
     * Actualiza la visibilidad de vistas según el estado
     */
    private fun actualizarVisibilidadVistas() {
        val hayTareas = controlador.hayTareas()
        
        recyclerViewTareas.visibility = if (hayTareas) View.VISIBLE else View.GONE
        layoutVacio.visibility = if (hayTareas) View.GONE else View.VISIBLE
        buttonLimpiar.isEnabled = controlador.obtenerTareasCompletadas().isNotEmpty()
    }
    
    // ===== MÉTODOS DE UTILIDAD =====
    
    /**
     * Muestra un mensaje usando diferentes métodos según el tipo
     */
    private fun mostrarMensaje(mensaje: String, tipo: TipoMensaje) {
        when (tipo) {
            TipoMensaje.EXITO -> {
                Toast.makeText(this, "✅ $mensaje", Toast.LENGTH_SHORT).show()
            }
            TipoMensaje.ERROR -> {
                Toast.makeText(this, "❌ $mensaje", Toast.LENGTH_LONG).show()
            }
            TipoMensaje.INFO -> {
                Toast.makeText(this, "ℹ️ $mensaje", Toast.LENGTH_SHORT).show()
            }
        }
    }
    
    /**
     * Muestra Snackbar con opción de deshacer
     */
    private fun mostrarSnackbarUndo(mensaje: String, tareaEliminada: Tarea) {
        Snackbar.make(findViewById(android.R.id.content), mensaje, Snackbar.LENGTH_LONG)
            .setAction("DESHACER") {
                // Funcionalidad para restaurar tarea (implementación básica)
                controlador.agregarNuevaTarea(tareaEliminada.titulo, tareaEliminada.descripcion)
                mostrarMensaje("Tarea restaurada", TipoMensaje.INFO)
            }
            .show()
    }
    
    // ===== ENUM PARA TIPOS DE MENSAJE =====
    
    /**
     * Enum que define los tipos de mensaje para mostrar
     */
    private enum class TipoMensaje {
        EXITO, ERROR, INFO
    }
}
```

---

## **PASO 9: ACTUALIZAR DEPENDENCIAS**

**¿Qué vamos a hacer?** Agregar las dependencias necesarias para Material Design y RecyclerView en el archivo build.gradle.

### Actualizar build.gradle (Module: app):

1. **Abrir** el archivo `app/build.gradle.kts` (o `build.gradle` si es Groovy)
2. **Buscar** la sección `dependencies`
3. **Agregar estas líneas** dentro del bloque dependencies:

```kotlin
dependencies {
    // Dependencias existentes...
    
    // Material Design Components
    implementation("com.google.android.material:material:1.12.0")
    
    // RecyclerView
    implementation("androidx.recyclerview:recyclerview:1.3.2")
    
    // ConstraintLayout (si no está)
    implementation("androidx.constraintlayout:constraintlayout:2.1.4")
    
    // Core KTX
    implementation("androidx.core:core-ktx:1.12.0")
    
    // Activity KTX
    implementation("androidx.activity:activity-ktx:1.8.2")
}
```

4. **Sincronizar** el proyecto: Click en "Sync Now" cuando aparezca el banner

---

## **PASO 10: PROBAR LA APLICACIÓN**

**¿Qué vamos a hacer?** Ejecutar y probar todas las funcionalidades de nuestra app con patrón MVC en Kotlin.

### Lista de verificación:

1. **Compilar sin errores:**
   - Build > Clean Project
   - Build > Rebuild Project
   - Verificar que no hay errores en rojo

2. **Ejecutar la aplicación:**
   - Click en Run (▶️) o Shift + F10
   - Seleccionar dispositivo o emulador
   - Esperar instalación

3. **Probar funcionalidades:**
   - ✅ **Agregar tarea**: Escribir título y presionar botón
   - ✅ **Marcar completada**: Click en checkbox (ver cambio visual)
   - ✅ **Eliminar tarea**: Click en botón eliminar o deslizar
   - ✅ **Ver detalles**: Click en card de la tarea
   - ✅ **Limpiar completadas**: Botón limpiar
   - ✅ **Estadísticas**: Verificar números actualizados
   - ✅ **Mensaje vacío**: Eliminar todas las tareas

---

## **PASO 11: CREAR LA DOCUMENTACIÓN**

**¿Qué vamos a hacer?** Crear documentación profesional explicando el patrón MVC implementado en Kotlin.

### Contenido del documento PDF:

1. **PORTADA:**
   - Tu nombre y matrícula
   - "Evidencia MVC - Lista de Tareas en Kotlin"
   - Fecha actual

2. **INTRODUCCIÓN:**
   ```
   Esta aplicación demuestra la implementación del patrón MVC (Modelo-Vista-Controlador)
   usando Kotlin moderno y las mejores prácticas de Android. La app permite gestionar
   una lista de tareas con funcionalidades CRUD completas.
   ```

3. **ARQUITECTURA MVC EN KOTLIN:**
   
   **MODELO:**
   - `Tarea.kt`: Data class que representa una tarea individual
   - `TareaManager.kt`: Clase que administra todas las tareas
   - **Características Kotlin utilizadas:**
     - Data classes para inmutabilidad
     - Propiedades computadas (get())
     - Funciones de extensión
     - Null safety
   
   **VISTA:**
   - `MainActivity.kt`: Activity principal con interfaz moderna
   - `TareaAdapter.kt`: Adapter para RecyclerView
   - Layouts XML con Material Design
   - **Características modernas:**
     - RecyclerView con DiffUtil
     - Material Design Components
     - ViewHolder pattern mejorado
   
   **CONTROLADOR:**
   - `TareaControlador.kt`: Intermediario entre Modelo y Vista
   - **Características Kotlin:**
     - Sealed classes para resultados
     - Funciones lambda para callbacks
     - Smart casting

4. **VENTAJAS DE KOTLIN EN MVC:**
   ```
   ✅ CÓDIGO MÁS CONCISO: Menos líneas de código que Java
   ✅ NULL SAFETY: Previene NullPointerException
   ✅ DATA CLASSES: Generación automática de equals, hashCode, toString
   ✅ PROPIEDADES: Sintaxis más limpia para getters/setters
   ✅ FUNCIONES LAMBDA: Callbacks más elegantes
   ✅ SEALED CLASSES: Mejor manejo de estados y resultados
   ```

5. **CAPTURAS DE PANTALLA:**
   - Pantalla inicial con tareas
   - Agregando nueva tarea
   - Tarea completada (con estilo visual)
   - Deslizar para eliminar
   - Pantalla vacía

6. **CONCLUSIONES PERSONALES:**
   Escribir tu experiencia con Kotlin vs Java, qué te pareció más fácil, etc.

---

## **PASO 12: CREAR VIDEO DEMOSTRATIVO**

**¿Qué mostrar en el video (3-4 minutos):**

1. **Introducción (30 seg):**
   - Presentación personal
   - Explicar que es una app MVC en Kotlin

2. **Demostración completa (2.5 min):**
   - Mostrar pantalla inicial
   - Agregar 3 tareas diferentes
   - Marcar algunas como completadas (mostrar cambio visual)
   - Click en tarea para ver detalles
   - Eliminar tarea con botón
   - Eliminar tarea deslizando
   - Usar botón limpiar completadas
   - Mostrar estadísticas actualizándose
