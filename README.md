# Generador de Crucigramas

Aplicación web que permite capturar palabras y sus descripciones para generar automáticamente un crucigrama dentro de una cuadrícula de 25 × 25.

## 2. Requerimientos

Para ejecutar la aplicación se requiere:

* Un navegador web moderno, como Google Chrome, Microsoft Edge o Mozilla Firefox.
* Los archivos `index.html`, `estilos.css` y `script.js`.
* No se requiere servidor web.
* No se requieren bases de datos.
* No se utilizan librerías externas.
* La aplicación utiliza HTML, CSS y JavaScript.

## 3. Descripción paso a paso del algoritmo

El algoritmo del generador de crucigramas recibe las palabras y sus descripciones, las valida y posteriormente intenta acomodarlas dentro de una matriz de 25 × 25. Las palabras se ordenan por longitud y se realizan diferentes intentos para obtener una distribución que permita colocar la mayor cantidad de palabras posible.

### Paso 1. Captura de palabras y descripciones

El usuario introduce una palabra y su descripción. El programa verifica que ambos datos hayan sido ingresados antes de almacenarlos.

```javascript
function agregarPalabra() {
    const original = $entradaPalabra.value.trim();
    const desc = $entradaDesc.value.trim();

    if (!original) {
        alert("Debes escribir una palabra.");
        return;
    }

    if (!desc) {
        alert("Debes escribir la descripción.");
        return;
    }
```

Una vez que los datos son válidos, la palabra y su descripción se almacenan para utilizarlas posteriormente durante la generación del crucigrama.

### Paso 2. Normalización de las palabras

Antes de utilizar las palabras en el algoritmo, el programa las normaliza. Las convierte a mayúsculas, conserva la letra Ñ, elimina los acentos y elimina caracteres que no sean letras.

```javascript
function normalizarPalabra(palabra) {
    return palabra.trim().toUpperCase()
        .replace(/Ñ/g, "\uFFFF")
        .normalize("NFD")
        .replace(/[\u0300-\u036f]/g, "")
        .replace(/\uFFFF/g, "Ñ")
        .replace(/[^A-ZÑ]/g, "");
}
```

Esto permite trabajar con las palabras en un formato uniforme durante la generación del crucigrama.

### Paso 3. Validación de las entradas

Antes de iniciar la generación, el programa verifica que las palabras cumplan con las condiciones necesarias. Se comprueba que exista una cantidad mínima de palabras, que ninguna supere el tamaño de la cuadrícula y que no existan palabras repetidas.

```javascript
function validarEntradas(palabras) {
```

Esta validación evita que datos incorrectos interfieran con el algoritmo de generación.

### Paso 4. Crear la matriz de 25 × 25

El programa crea una matriz de 25 filas por 25 columnas. Inicialmente todas las posiciones tienen el valor `null`, lo que representa una casilla vacía.

```javascript
const TAMANO_CUADRICULA = 25;

function crearCuadricula(tamano) {
    const c = [];

    for (let f = 0; f < tamano; f++) {
        c[f] = [];

        for (let col = 0; col < tamano; col++) {
            c[f][col] = null;
        }
    }

    return c;
}
```

La matriz funciona como el espacio donde se colocarán las letras de las palabras.

### Paso 5. Ordenar las palabras

Las palabras se ordenan de mayor a menor longitud. Esto permite utilizar primero la palabra más larga como base para construir el crucigrama.

```javascript
copia.sort((a, b) => {
    if (b.palabra.length !== a.palabra.length)
        return b.palabra.length - a.palabra.length;

    return a.palabra.localeCompare(b.palabra);
});
```

De esta manera, la palabra con mayor cantidad de letras queda en la primera posición de la lista.

### Paso 6. Colocar la primera palabra

Después de ordenar las palabras, el algoritmo toma la primera palabra de la lista. Debido al orden realizado anteriormente, esta corresponde a una de las palabras de mayor longitud.

La primera palabra se utiliza como punto inicial para construir el crucigrama y se coloca cerca del centro de la cuadrícula de 25 × 25.

La posición inicial se calcula de la siguiente manera:

```javascript
const fIni = Math.floor(TAMANO_CUADRICULA / 2);
const cIni = Math.floor((TAMANO_CUADRICULA - primera.palabra.length) / 2);
```

La fila se obtiene tomando la mitad de la cuadrícula, mientras que la columna se calcula considerando la longitud de la palabra para que quede centrada.

La dirección de la primera palabra puede ser horizontal o vertical dependiendo del intento que se esté realizando:

```javascript
const dir1 = semilla % 2 === 0 ? "horizontal" : "vertical";
```

Finalmente, la palabra se coloca en la cuadrícula mediante la función `colocarPalabra()`.

```javascript
colocarPalabra(cuadricula, primera.palabra, fIni, cIni, dir1);
```

Después de colocarla, el programa guarda sus datos, como fila, columna y dirección, y la agrega a la lista de palabras colocadas.

### Paso 7. Separar palabras colocadas y pendientes

Una vez colocada la primera palabra, el algoritmo divide el resto de las palabras en dos grupos:

* Palabras colocadas:aquellas que ya fueron ubicadas dentro de la cuadrícula.
* Palabras pendientes: aquellas que todavía necesitan encontrar una posición válida.

Esto se realiza mediante:

```javascript
let colocadas  = [primera];
let pendientes = copia.slice(1);
```

La primera palabra queda dentro de `colocadas`, mientras que las demás pasan a `pendientes`.

### Paso 8. Buscar posiciones para las palabras pendientes

El algoritmo recorre las palabras pendientes una por una y busca posiciones donde puedan colocarse.
Primero intenta encontrar posiciones que permitan que la nueva palabra se cruce con alguna palabra que ya se encuentre en la cuadrícula:

```javascript
let posiciones = buscarPosiciones(
    cuadricula,
    elem.palabra,
    true
);
```

La función `buscarPosiciones()` recorre las filas y columnas de la cuadrícula y analiza las letras existentes.
Cuando encuentra una letra que coincide con una letra de la palabra que se quiere colocar, calcula una posible posición horizontal y otra vertical.

Por ejemplo, si una letra coincide, el algoritmo puede intentar colocar la palabra de esta forma:

* Horizontalmente, desplazando la palabra hasta hacer coincidir la letra.
* Verticalmente, desplazando la palabra hasta hacer coincidir la letra.

De esta manera se buscan diferentes posibilidades de cruce.

### Paso 9. Validar cada posición encontrada

Cada posición propuesta se verifica mediante la función `validarPosicion()` antes de colocar la palabra.

La validación comprueba diferentes condiciones:

1. Que la palabra permanezca dentro de los límites de la cuadrícula.
2. Que no existan letras diferentes en las posiciones donde se quiere colocar.
3. Que las letras que coincidan sean iguales.
4. Que las celdas perpendiculares no provoquen choques incorrectos.
5. Que los extremos de la palabra no estén ocupados incorrectamente.
6. Que exista un cruce cuando el algoritmo lo requiere.

La función devuelve información sobre si la posición es válida y cuántos cruces produce:

```javascript
return {
    valido: true,
    cruces: cruces
};
```

Si se encuentra una letra diferente en una posición que ya está ocupada, la posición se descarta:

```javascript
if (celda !== null && celda !== palabra[i])
    return false;
```

Esto evita que dos palabras se coloquen de manera incompatible.

### Paso 10. Buscar una posición sin cruce

Si no se encuentra ninguna posición válida que permita cruzar la palabra con las palabras existentes, el algoritmo realiza una segunda búsqueda.

```javascript
if (!posiciones.length)
    posiciones = buscarPosicionesSinCruce(
        cuadricula,
        elem.palabra
    );
```

Esta función busca posiciones libres tanto horizontal como verticalmente.
Esto permite que el algoritmo tenga otra posibilidad para colocar una palabra cuando todavía existe espacio disponible, aunque no pueda realizar un cruce.

### Paso 11. Seleccionar y colocar una posición

Cuando existen varias posiciones válidas, estas se ordenan de acuerdo con la cantidad de cruces que producen.

```javascript
resp.sort((a, b) => b.cruces - a.cruces);
```

Las posiciones con mayor cantidad de cruces quedan primero.
Posteriormente, el algoritmo toma las tres mejores posiciones:

```javascript
const top = posiciones.slice(0, 3);
```

De esas tres posibilidades selecciona una de manera aleatoria:

```javascript
const pos =
    top[Math.floor(Math.random() * top.length)];
```

Esto permite obtener diferentes distribuciones del crucigrama cuando se vuelve a generar.
Finalmente, la palabra se coloca en la cuadrícula:

```javascript
colocarPalabra(
    cuadricula,
    elem.palabra,
    pos.fila,
    pos.columna,
    pos.direccion
);
```

También se guardan su fila, columna y dirección para poder utilizar posteriormente esa información.

### Paso 12. Repetir el proceso y conservar el mejor resultado

El algoritmo realiza varias pasadas sobre las palabras pendientes:

```javascript
for (
    let pasada = 0;
    pasada < 10 && pendientes.length > 0;
    pasada++
)
```

En cada pasada intenta colocar las palabras que todavía no han sido ubicadas.
Si una palabra no puede colocarse, permanece en la lista de pendientes y se vuelve a intentar en una siguiente pasada.

El proceso termina cuando:
* Se colocaron todas las palabras.
* Se realizaron las 10 pasadas permitidas.
* O una pasada completa no consiguió colocar ninguna palabra.

Después de generar un intento, el programa guarda:

```javascript
return {
    cuadricula: cuadricula,
    palabras: copia,
    colocadas: colocadas,
    pendientes: pendientes
};
```

La función `generarAlgoritmo()` realiza hasta cinco intentos diferentes:

```javascript
for (let i = 0; i < 5; i++) {
    const r = generarUnIntento(palabras, i);
```

Después compara la cantidad de palabras colocadas en cada intento.

```javascript
if (r.colocadas.length > mejorCant) {
    mejorCant = r.colocadas.length;
    mejor = r;
}
```

Por lo tanto, el resultado final es el intento que consiguió colocar la mayor cantidad de palabras.

Si se logra colocar todas las palabras, el algoritmo puede detener los intentos antes de llegar a cinco:

```javascript
if (mejorCant === palabras.length)
    break;
```

De esta manera, el algoritmo busca generar una distribución adecuada utilizando diferentes posibilidades y conserva el resultado que logra colocar más palabras.


### Paso 13. Numerar las palabras

Después de colocar las palabras, el programa asigna un número a cada palabra. La numeración se realiza tomando como referencia la posición inicial de cada palabra dentro de la cuadrícula.

Primero se obtienen las posiciones iniciales de las palabras colocadas y posteriormente se ordenan de arriba hacia abajo y de izquierda a derecha.

```javascript
inicios.sort((a, b) =>
    a.fila !== b.fila
        ? a.fila - b.fila
        : a.columna - b.columna
);
```

Cuando dos palabras comienzan en la misma casilla, comparten el mismo número.

```javascript
const clave = ini.fila + "-" + ini.columna;

if (numeros[clave] === undefined)
    numeros[clave] = sig++;

ini.ref.numero = numeros[clave];
```

De esta manera, cada palabra queda identificada con un número que posteriormente se utiliza para mostrar sus pistas.

### Paso 14. Calcular los límites visibles

La cuadrícula original tiene un tamaño de 25 × 25, pero no es necesario mostrar todas las filas y columnas si están vacías.

La función `calcularLimites()` identifica la posición mínima y máxima ocupada por las palabras.

```javascript
for (const p of datos.colocadas) {
    for (let i = 0; i < p.palabra.length; i++) {
        const f = p.direccion === "horizontal"
            ? p.fila
            : p.fila + i;

        const c = p.direccion === "horizontal"
            ? p.columna + i
            : p.columna;
```

Después se agrega una casilla de margen alrededor del crucigrama para que las palabras no queden pegadas al límite visual.
Esto permite mostrar solamente la parte de la matriz que contiene el crucigrama.

### Paso 15. Generar visualmente el crucigrama

La función `mostrarCuadricula()` transforma la información obtenida por el algoritmo en elementos HTML que pueden visualizarse en el navegador.

Primero obtiene la numeración y los límites:

```javascript
const numeros = numerarPalabras(datos);
const limites = calcularLimites(datos);
```

Después recorre las filas y columnas correspondientes a la zona ocupada.

Cuando una posición está vacía, se genera una casilla vacía:

```javascript
if (letra === null) {
    html += '<div class="celda vacia"></div>';
}
```

Cuando existe una letra, se genera una casilla jugable con un campo de entrada:

```javascript
html += `
    <div class="celda jugable">
        <input type="text">
    </div>
`;
```

También se coloca el número correspondiente cuando la casilla es el inicio de una palabra.

Además de la cuadrícula, esta función genera:

* Cantidad de palabras colocadas.
* Cantidad de palabras pendientes.
* Lista de pistas.
* Panel lateral de interacción.
* Botón para generar nuevamente.
* Botón para descargar el crucigrama.
* Botón para descargar la solución.
* Botón para comprobar las respuestas.

Finalmente, se activan las funciones necesarias para que el crucigrama sea interactivo.

### Paso 16. Identificar la palabra seleccionada

Cuando el usuario selecciona una casilla, el programa determina qué palabra corresponde a esa posición.

La función `obtenerPalabrasEnCelda()` revisa si la casilla pertenece a una palabra horizontal, vertical o a ambas.

Esto permite trabajar correctamente con las casillas donde se cruzan dos palabras.

La función `determinarDireccion()` decide si se debe trabajar con la palabra horizontal o vertical.

Cuando una casilla pertenece a las dos direcciones, el usuario puede cambiar entre ellas.

### Paso 17. Mostrar la pista de la palabra

Una vez determinada la palabra, la función `mostrarPistaActual()` muestra la información correspondiente en el panel lateral.

Se muestra:

* Número de la palabra.
* Dirección: horizontal o vertical.
* Cantidad de letras.
* Descripción de la palabra.

Por ejemplo, la información puede presentarse como:

```text
1 Horizontal
Número de letras: 9

Descripción:
Conjunto ordenado de instrucciones...
```

Esto permite que el usuario resuelva el crucigrama utilizando las descripciones proporcionadas al momento de capturar las palabras.

### Paso 18. Escribir las respuestas mediante el panel de interacción

El programa utiliza la función `abrirModalEclipse()` para mostrar una interfaz similar al funcionamiento de EclipseCrossword.

Cuando el usuario selecciona una palabra, aparece un panel donde puede consultar la pista y escribir su respuesta.

Antes de aceptar la respuesta se realizan diferentes validaciones.

Se verifica que:

1. El usuario haya escrito una respuesta.
2. Solamente se utilicen letras permitidas.
3. La cantidad de letras coincida con la longitud de la palabra.
4. No se introduzcan menos letras de las necesarias.
5. No se introduzcan más letras de las permitidas.

Si la cantidad de letras no coincide, el programa muestra un mensaje indicando el problema y permite corregir la respuesta.
Cuando la respuesta es válida, sus letras se colocan en las casillas correspondientes del crucigrama.
También existe la opción de cancelar la operación.

### Paso 19. Controlar la navegación dentro del crucigrama

La función `activarCasillas()` permite interactuar directamente con las casillas.

El usuario puede seleccionar una casilla mediante el mouse y utilizar el teclado para desplazarse.

Se contemplan diferentes teclas:

* Flecha arriba.
* Flecha abajo.
* Flecha izquierda.
* Flecha derecha.
* Enter.
* Backspace.
* Delete.
* Tab.

Cuando una palabra tiene una intersección horizontal y vertical, la dirección puede cambiarse para seleccionar la palabra correspondiente.

También se utiliza el resaltado para indicar visualmente la palabra que está seleccionada.

### Paso 20. Comprobar el crucigrama

Cuando el usuario termina de responder, puede utilizar el botón para comprobar el crucigrama.

La función principal encargada de esta acción es:

```javascript
comprobarCrucigrama()
```

Primero obtiene las letras introducidas por el usuario y las organiza según la posición de cada casilla.
Después compara las respuestas introducidas con las palabras originales almacenadas en el crucigrama.
Cada letra se compara con la letra correspondiente de la palabra.
Si coincide, la casilla se marca como correcta.
Si no coincide, se marca como incorrecta.

### Paso 21. Determinar las palabras acertadas

Una palabra se considera acertada cuando todas sus letras coinciden con la respuesta original.

El programa revisa cada palabra colocada:

```javascript
for (const p of crucigramaActual.colocadas) {
```

Si todas sus posiciones contienen las letras correctas, se incrementa el número de aciertos.
Si contiene alguna letra incorrecta, se registra como una palabra con error.
También se identifican las palabras que todavía están incompletas.

De esta manera se pueden distinguir tres situaciones:

* Palabras correctas.
* Palabras con errores.
* Palabras incompletas.

### Paso 22. Calcular la calificación

La calificación se obtiene a partir de la cantidad de palabras acertadas.

La función utilizada es:

```javascript
function calificar(aciertos, errores, vacias, total) {
    const nota = total > 0
        ? Math.round((aciertos / total) * 10)
        : 0;
```

La fórmula utilizada es:

```text
Calificación = (Aciertos / Total de palabras) × 10
```

Por ejemplo, si el crucigrama tiene 5 palabras y el usuario acierta las 5:

```text
Aciertos: 5 de 5
Calificación: 10
```

Si solamente acierta 4 de 5:

```text
Aciertos: 4 de 5
Calificación: 8
```

### Paso 23. Mostrar el resultado

Después de realizar la comprobación, el programa muestra un mensaje con los resultados obtenidos.

Cuando todas las palabras son correctas, se muestra un mensaje indicando que el crucigrama fue completado correctamente, junto con la cantidad de aciertos y la calificación.

Cuando todavía existen errores o palabras sin completar, se muestra un mensaje indicando que el crucigrama aún no está completo.

También se muestran los datos correspondientes a:

* Aciertos.
* Palabras con error.
* Palabras incompletas.
* Calificación obtenida.

### Paso 24. Generar nuevamente el crucigrama

El programa permite generar otra distribución utilizando las mismas palabras capturadas anteriormente.

La función `activarRegenerar()` utiliza la lista guardada en:

```javascript
ultimaListaPalabras
```

Posteriormente vuelve a ejecutar:

```javascript
generarAlgoritmo(ultimaListaPalabras)
```

Esto permite obtener una nueva distribución de las palabras sin tener que capturarlas nuevamente.

### Paso 25. Preparar el crucigrama para descargar

La aplicación permite descargar el crucigrama como un archivo HTML independiente.

La función:

```javascript
construirHtmlIndependiente(datos)
```

genera el contenido necesario para que el crucigrama pueda abrirse posteriormente en un navegador sin depender de los archivos originales del proyecto.

El archivo descargado contiene:

* La cuadrícula.
* Las casillas.
* Las pistas.
* La interacción.
* La comprobación de respuestas.
* El cálculo de la calificación.

El archivo se genera mediante un `Blob` y posteriormente se crea un enlace de descarga.

### Paso 26. Ocultar las respuestas en el archivo HTML

Para evitar que las respuestas aparezcan directamente en el código HTML generado, el programa utiliza una función de codificación:

```javascript
codificarTexto(p.palabra)
```

Los datos de las palabras se preparan mediante:

```javascript
buildDatosOfuscadosExport(datos)
```

En lugar de colocar directamente la respuesta correcta dentro de atributos como:

```html
data-correcta="..."
```

el archivo exportado contiene una representación ofuscada de la información.

Posteriormente, el propio programa puede recuperar esa información durante la comprobación.

Esto evita que las respuestas estén almacenadas de forma directa y fácilmente visible en el HTML generado.

### Paso 27. Descargar la solución

Además del crucigrama normal, la aplicación permite generar una versión con las respuestas visibles.

La función utilizada es:

```javascript
construirHtmlSolucion(datos)
```

En esta versión las casillas contienen las letras correspondientes y se muestran como campos de solo lectura.

Esto permite obtener una versión de referencia del crucigrama ya resuelto.

El archivo se descarga con un nombre diferente:

```text
solucion-crucigrama.html
```

Mientras que la versión normal se descarga como:

```text
crucigrama.html
```

### Paso 28. Ejecutar todo el proceso desde el botón Generar

El proceso completo comienza cuando el usuario presiona el botón **Generar crucigrama**.

El programa realiza las siguientes acciones:

```javascript
$btnGenerar.addEventListener("click", function () {
    let palabras = obtenerPalabras();

    const errores = validarEntradas(palabras);

    if (errores.length) {
        mostrarErroresValidacion(errores);
        return;
    }

    palabras = eliminarDuplicadas(palabras);

    ultimaListaPalabras = palabras.map(...);

    crucigramaActual = generarAlgoritmo(palabras);

    mostrarCuadricula(crucigramaActual);
});
```

                           ┌─────────────┐
                           │   INICIO    │
                           └──────┬──────┘
                                  ↓
                 ┌──────────────────────────────┐
                 │ Capturar palabras y          │
                 │ descripciones                │
                 └──────────────┬───────────────┘
                                ↓
                    ┌──────────────────────┐
                    │ Normalizar palabras │
                    │ (mayúsculas, Ñ,     │
                    │ eliminar acentos)   │
                    └──────────┬───────────┘
                               ↓
                    ┌──────────────────────┐
                    │ ¿Datos válidos?      │
                    └───────┬────────┬─────┘
                         NO │        │ SÍ
                            ↓        ↓
                   ┌────────────┐  ┌────────────────────┐
                   │ Mostrar    │  │ Eliminar palabras  │
                   │ errores    │  │ duplicadas         │
                   └─────┬──────┘  └──────────┬─────────┘
                         ↓                     ↓
                       FIN            ┌────────────────────┐
                                      │ Ordenar palabras   │
                                      │ por longitud       │
                                      └──────────┬─────────┘
                                                 ↓
                                      ┌────────────────────┐
                                      │ Crear matriz       │
                                      │ de 25 × 25         │
                                      └──────────┬─────────┘
                                                 ↓
                                      ┌────────────────────┐
                                      │ Colocar primera    │
                                      │ palabra al centro  │
                                      └──────────┬─────────┘
                                                 ↓
                                      ┌────────────────────┐
                                      │ Crear lista de     │
                                      │ palabras pendientes│
                                      └──────────┬─────────┘
                                                 ↓
                                      ┌────────────────────┐
                                      │ Buscar posiciones  │
                                      │ con cruces         │
                                      └──────────┬─────────┘
                                                 ↓
                                      ┌────────────────────┐
                                      │ ¿Existe posición   │
                                      │ válida?            │
                                      └───────┬──────┬─────┘
                                           NO│      │SÍ
                                             ↓      ↓
                              ┌──────────────────┐  ┌──────────────────┐
                              │ Buscar posición  │  │ Seleccionar una │
                              │ sin cruce        │  │ de las mejores  │
                              └────────┬─────────┘  └────────┬─────────┘
                                       ↓                     ↓
                              ┌────────────────────────────────────┐
                              │ ¿Existe una posición disponible?   │
                              └───────────────┬──────────────┬─────┘
                                           NO│              │SÍ
                                             ↓              ↓
                                      ┌────────────┐  ┌────────────────┐
                                      │ Mantener   │  │ Colocar palabra│
                                      │ pendiente  │  │ en la matriz    │
                                      └─────┬──────┘  └───────┬────────┘
                                            │                 │
                                            └────────┬────────┘
                                                     ↓
                                      ┌──────────────────────────┐
                                      │ ¿Quedan palabras         │
                                      │ pendientes?              │
                                      └───────────┬────────┬─────┘
                                               SÍ│        │NO
                                                 ↓        ↓
                                      ┌────────────────┐  │
                                      │ Repetir hasta  │  │
                                      │ 10 pasadas     │  │
                                      └───────┬────────┘  │
                                              │           │
                                              └─────┐     │
                                                    ↓     ↓
                                      ┌──────────────────────────┐
                                      │ ¿Se realizaron 5 intentos│
                                      │ de generación?           │
                                      └───────────┬────────┬─────┘
                                               NO│        │SÍ
                                                 ↓        ↓
                                      ┌────────────────┐  │
                                      │ Generar nuevo  │  │
                                      │ intento        │  │
                                      └───────┬────────┘  │
                                              │           │
                                              └─────┐     │
                                                    ↓     ↓
                                      ┌──────────────────────────┐
                                      │ Conservar el intento con │
                                      │ más palabras colocadas  │
                                      └────────────┬─────────────┘
                                                   ↓
                                      ┌──────────────────────────┐
                                      │ Numerar las palabras     │
                                      └────────────┬─────────────┘
                                                   ↓
                                      ┌──────────────────────────┐
                                      │ Calcular límites visibles│
                                      └────────────┬─────────────┘
                                                   ↓
                                      ┌──────────────────────────┐
                                      │ Mostrar crucigrama y     │
                                      │ pistas                   │
                                      └────────────┬─────────────┘
                                                   ↓
                                      ┌──────────────────────────┐
                                      │ Usuario resuelve         │
                                      │ el crucigrama             │
                                      └────────────┬─────────────┘
                                                   ↓
                                      ┌──────────────────────────┐
                                      │ Comprobar respuestas     │
                                      └────────────┬─────────────┘
                                                   ↓
                                      ┌──────────────────────────┐
                                      │ Calcular aciertos,       │
                                      │ errores y calificación   │
                                      └────────────┬─────────────┘
                                                   ↓
                                      ┌──────────────────────────┐
                                      │ Descargar crucigrama    │
                                      │ HTML o solución          │
                                      └────────────┬─────────────┘
                                                   ↓
                                             ┌─────────┐
                                             │   FIN   │
                                             └─────────┘

### Paso 29. Resultado final del algoritmo

El algoritmo busca generar automáticamente una distribución de palabras dentro de una matriz de 25 × 25.

Su funcionamiento se basa principalmente en:

1. Recibir las palabras y sus descripciones.
2. Normalizar y validar los datos.
3. Ordenar las palabras por longitud.
4. Colocar una primera palabra como punto de partida.
5. Buscar coincidencias entre las letras de las palabras.
6. Validar las posiciones disponibles.
7. Colocar las palabras que puedan cruzarse.
8. Intentar colocar las palabras pendientes nuevamente.
9. Realizar varios intentos de generación.
10. Conservar el intento que haya colocado la mayor cantidad de palabras.
11. Numerar las palabras.
12. Mostrar el crucigrama generado.
13. Permitir que el usuario lo resuelva.
14. Comprobar las respuestas.
15. Calcular la calificación.
16. Permitir descargar el crucigrama en formato HTML.

De esta manera, el programa no utiliza una distribución fija, sino que calcula las posiciones de las palabras mediante diferentes validaciones y varios intentos de generación.
