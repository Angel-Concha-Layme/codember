## Descripción del Reto 🕵️‍♂️🔍

### Objetivo 🎯

Un espía ha estado transmitiendo mensajes secretos que necesitamos descifrar. Estos mensajes consisten en una serie de palabras codificadas, y tu labor es desarrollar un algoritmo capaz de analizar estos mensajes y encontrar los patrones ocultos.

### Especificaciones del Programa 📝

El programa debe realizar lo siguiente:

- **Analizar el Texto**: Procesar una cadena de texto que contiene palabras separadas por espacios. Por ejemplo:

    gato perro perro coche Gato peRRo sol


- **Normalizar Mayúsculas y Minúsculas**: Identificar palabras sin importar la diferencia entre mayúsculas y minúsculas, tratándolas como iguales.

- **Contabilizar Apariciones**: Calcular la frecuencia con la que cada palabra única aparece en el mensaje.

- **Formato de Salida**: Generar una cadena de texto que contenga cada palabra seguida del número de veces que aparece. La cadena debe mantener el orden de la primera aparición de cada palabra en el mensaje original. Por ejemplo:


    gato2perro3coche1sol1


### Ejemplos Adicionales 📊

Aquí se muestran algunos ejemplos más del funcionamiento esperado del programa:

- Entrada: `llaveS casa CASA casa llaves`
Salida: `llaves2casa3`

- Entrada: `taza ta za taza`
Salida: `taza2ta1za1`

- Entrada: `casas casa casasas`
Salida: `casas1casa1casas1`



## Solución del Reto 🤓
Este problema se puede solucionar de diversas maneras, pero una estrategia particularmente eficaz para lidiar con el procesamiento de texto y la agregación de datos es el modelo MapReduce. Este modelo se alinea bien con tareas que implican la manipulación y conteo de elementos como las palabras en un texto. Aunque nuestra aplicación no requiere la escala para la cual MapReduce fue originalmente diseñado, adoptar este patrón ofrece claridad conceptual, potencial de escalabilidad y una solución estructurada que podría ser extendida a entornos de procesamiento distribuido si fuera necesario.

### Por qué MapReduce es Adecuado
MapReduce es un paradigma poderoso cuando necesitamos analizar grandes cantidades de datos y extraer información valiosa de ellos. En el caso de los mensajes encriptados de un espía, la simplicidad de paralelización, escalabilidad, tolerancia a fallos y el manejo eficiente de pares clave-valor lo hacen una elección acertada incluso para conjuntos de datos menos voluminosos.



### Fases del MapReduce

La solución adopta el modelo de MapReduce para dividir el proceso de análisis del texto en dos fases principales, cada una con sus propias responsabilidades y operaciones específicas. A continuación, detallo cada fase y explico con mayor profundidad las tareas involucradas:

#### Fase de Mapeo (Map Phase)

En la fase de mapeo, el objetivo es preparar los datos para el proceso de agregación. Aquí se realiza el trabajo preliminar de organizar la información de entrada de tal manera que pueda ser procesada eficientemente en la siguiente fase.

- **Tokenización**: Esta operación implica escanear el texto de entrada y separarlo en elementos individuales, llamados tokens, basados en un criterio definido, que en este caso es el espacio en blanco entre palabras. Cada token representa una palabra que será sujeto de conteo.

- **Normalización**: Una vez que tenemos los tokens, se procede a normalizarlos. Esto significa que se eliminan las diferencias de formato que no son relevantes para el conteo, como las mayúsculas y minúsculas. Al convertir todos los tokens a minúsculas, aseguramos que 'Palabra', 'palabra' y 'PALABRA' se cuenten como la misma entidad.

- **Contabilización**: Finalmente, cada token normalizado se contabiliza. Se crea un par clave-valor donde la clave es la palabra normalizada y el valor es el número de veces que esa palabra ha aparecido hasta el momento. Este proceso se lleva a cabo utilizando una estructura de datos que mantiene el orden de inserción (en Java, `LinkedHashMap`), para que podamos luego reconstruir el mensaje en el mismo orden en que las palabras aparecieron originalmente.

```java 
    public static class Mapper {
        public Map<String, Integer> map(String input) {
            Map<String, Integer> wordCount = new LinkedHashMap<>();
            String[] words = input.split(" ");
            for (String word : words) {
                word = word.toLowerCase();
                wordCount.put(word, wordCount.getOrDefault(word, 0) + 1);
            }
            return wordCount;
        }
    }
```



#### Fase de Reducción (Reduce Phase)

La fase de reducción toma la salida del mapeo, que es un conjunto de pares clave-valor, y los combina para producir el resultado final.

- **Agregación**: En esta etapa, procesamos la colección de pares clave-valor para consolidar los resultados. Dado que el proceso de mapeo ya ha contabilizado las apariciones de cada palabra, en realidad no hay una 'agregación' en el sentido tradicional de combinar múltiples valores. En su lugar, lo que hacemos es prepararnos para la salida final al asegurarnos de que tenemos un conteo completo y final para cada palabra.

- **Construcción de la Salida**: Con los conteos finales en mano, construimos la cadena de salida. Iteramos sobre el `LinkedHashMap`, que nos da las palabras en el orden de su primera aparición, y para cada palabra, anexamos su contador al final. Este proceso genera una cadena que es la representación agregada del mensaje original, con el formato requerido de "palabraconteo".

```java
    public static class Reducer {
        public String reduce(Map<String, Integer> mappedData) {
            StringBuilder result = new StringBuilder();
            for (Map.Entry<String, Integer> entry : mappedData.entrySet()) {
                result.append(entry.getKey()).append(entry.getValue());
            }
            return result.toString();
        }
    }
```





Cada una de estas fases se diseñó para operar de manera eficiente sobre el conjunto de datos y, aunque en esta implementación trabajan de manera secuencial en un solo hilo, están conceptualmente preparadas para ser escaladas a un entorno distribuido donde la tokenización y la contabilización podrían ocurrir en paralelo a través de múltiples nodos en un clúster, y la reducción se encargaría de unir los resultados intermedios en la respuesta final.