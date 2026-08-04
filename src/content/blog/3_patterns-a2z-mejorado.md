---
title: 'Guía completa para resolver los patrones de la A2Z Sheet (impresión de figuras)'
description: 'Recorrido por los 22 patrones de la Striver A2Z DSA Sheet — cómo reconocer la fórmula detrás de cada figura, agrupados por técnica.'
heroImage: '../../assets/posts/3.png'
pubDate: '2026-07-31'
updatedDate: '2026-08-02'
---

## Introducción

Antes de llegar a árboles, grafos o programación dinámica, hay un cimiento mucho menos glamoroso que hay que dominar primero: manejar loops anidados y entender bien la relación entre el índice de la fila (`i`) y lo que se imprime en cada columna (`j`). Los patrones de impresión de la A2Z Sheet existen justamente para entrenar eso — y cuando los empecé, confieso que los subestimé. Pensé que iba a ser un trámite rápido antes de llegar a "lo interesante". Terminé pasando más tiempo del que esperaba, no porque las figuras fueran difíciles de entender visualmente, sino porque calcular el rango exacto de cada loop a mano, sin errores de índice, resultó ser un ejercicio en sí mismo.

Así que decidí armar esta guía recorriendo los 22 patrones agrupados por técnica, con la lógica y las fórmulas que usé para resolver cada uno — pensada tanto para mí como referencia futura, como para cualquiera que esté arrancando y quiera reconocer el patrón matemático detrás de una figura nueva que no esté en esta lista.

Una aclaración sobre el formato: cuando una técnica se repite entre patrones (como el manejo de espejos, o sumar un entero a un `char`), la explico la primera vez que aparece, y en los siguientes patrones solo señalo qué cambia respecto a esa primera vez — no todo el razonamiento de nuevo.

## El principio general

Con el tiempo, terminé reduciendo cada patrón a responderme dos preguntas por cada fila `i`:

1. **¿Cuántas veces se repite algo en esta fila?** (el rango del loop interno)
2. **¿Qué valor se imprime en cada repetición?** (un carácter fijo, un contador, o una letra calculada)

La mayoría de los errores que cometí no vinieron de no entender la figura — visualmente todas son bastante intuitivas — sino de calcular mal el rango exacto del loop interno (`<` vs `<=`) o de olvidar en qué dirección debía iterar (`++` vs `--`). Lo que mejor me funcionó, después de tropezar varias veces con lo mismo, fue simular a mano las primeras 2-3 filas con un `n` pequeño (n=4 o n=5) antes de escribir una sola línea de código, anotando cuántas repeticiones y qué valores corresponden a cada fila. Suena obvio dicho así, pero es el paso que más se salta cuando uno tiene apuro por llegar al código.

## Bloque 1 — Figuras simples con un solo loop interno (Patterns 1-6)

Estos seis patrones fueron los primeros que resolví, y comparten la misma estructura de fondo: un loop externo por fila, un loop interno que imprime algo `k` veces, donde `k` depende de `i` de forma directa. Una vez que entendí esto, dejaron de sentirse como seis problemas distintos y empezaron a sentirse como una única plantilla con parámetros distintos.

| Patrón                  | Rango del loop interno                      | Qué se imprime                |
| ----------------------- | ------------------------------------------- | ----------------------------- |
| 1 — Cuadrado            | `j` de 0 a n-1 (fijo, no depende de `i`)    | `*`                           |
| 2 — Triángulo creciente | `j` de 0 a `i`                              | `*`                           |
| 3 — Números crecientes  | `j` de 1 a `i+1`                            | `j` (el propio contador)      |
| 4 — Número repetido     | `j` de 0 a `i`                              | `i+1` (constante en la fila)  |
| 5 — Triángulo invertido | `j` de `n` a `i+1` (decreciente)            | `*`                           |
| 6 — Números decrecientes| `j` de 1 a `n-i` (decreciente)              | `j`                           |

**Lo que más me costó internalizar acá:** cuando el número de repeticiones crece con `i` (patrones 2-4), el loop interno va de `0` a `i` inclusive — o sea, `i+1` repeticiones, no `i`. Es un off-by-one clásico y lo cometí más de una vez. Cuando decrece (patrones 5-6), el loop interno depende de `n-i`, no de un valor fijo de `n` — otra fuente típica de confusión si se copia la estructura del patrón anterior sin ajustar el signo del conteo.

## Bloque 2 — Pirámides con espacios (Patterns 7-9)

Acá el nivel subió un escalón: cada fila necesita **dos o tres loops internos secuenciales** — primero los espacios, luego los caracteres visibles (y a veces un tercer loop para el espejo). La primera vez que intenté esto sin pensarlo antes en papel, terminé con figuras "desalineadas" que casi tenían la forma correcta pero no del todo — y ese "casi" fue justamente lo que me enseñó a simular las filas a mano antes de tocar el teclado.

**Pattern 7 (pirámide):** fila `i` tiene `n-1-i` espacios seguidos de `2i+1` asteriscos.

**Pattern 8 (pirámide invertida):** fila `i` tiene `i` espacios seguidos de `2(n-i-1)+1` asteriscos.

**Pattern 9 (diamante):** acá tuve una especie de "ajá" — no hace falta inventar una fórmula nueva. Es la concatenación directa de Pattern 7 (mitad superior) y Pattern 8 (mitad inferior), una debajo de la otra, reusando exactamente lo que ya había resuelto.

**Técnica clave:** cuando una figura tiene una mitad superior y una inferior, me resultó mucho más fácil resolver primero cada mitad como un patrón independiente (ya conocido) y luego unirlas, en vez de perseguir una fórmula única que cubra ambas a la vez. Y algo que se me pasó más de una vez: verificar que cada fila termine con su salto de línea — es fácil de olvidar cuando hay varios loops internos antes del `\n`, y el resultado visual queda todo apilado en una sola línea gigante.

## Bloque 3 — Números en espejo y booleanos alternos (Patterns 10-11)

**Pattern 10 (rombo de asteriscos sin espacios):** mitad superior con `i+1` asteriscos crecientes (de 1 a n), mitad inferior con `n-1-i` asteriscos decrecientes (de n-1 a 1) — el detalle en el que me trabé fue el rango del segundo bloque: va de `0` a `n-2`, no hasta `n-1`, para no repetir la fila más ancha dos veces.

**Pattern 11 (binario alterno 1/0):** este fue el primero que me hizo parar a pensar en serio. Se necesita una variable booleana (`mark`) que alterna entre 1 y 0 en cada carácter impreso. Lo delicado no es alternar dentro de la fila — eso es directo — sino **cuándo resetear** esa variable para que la fila siguiente empiece con el valor correcto: si la fila `i` es par, la fila `i+1` es impar y debe empezar en `0`. El reset se calcula en función de la paridad de la fila que *sigue*, no de la que acaba de terminar, y esa inversión de a quién le corresponde el reset fue justamente lo que me hizo perder más tiempo del que quisiera admitir.

## Bloque 4 — Cuando cambian los números por letras (Patterns 12-18)

**Pattern 12 (números con espacio central):** tres loops — números crecientes (1 a i+1), espacios centrales (`2*(n-1-i)`), números decrecientes (i+1 a 1).

**Pattern 13 (contador continuo):** en esencia es el mismo Pattern 3, pero en vez de reiniciar el contador en cada fila, se usa una variable declarada *fuera* del loop externo que sigue subiendo fila tras fila sin resetearse. Es el mismo patrón de siempre, solo que ahora el contador "recuerda" el valor entre filas — un cambio de una línea de código, pero que si no lo esperás, te hace dudar si el patrón anterior estaba mal.

**Pattern 14 (letras crecientes):** es Pattern 2 pero con letras en vez de asteriscos. El loop interno sigue yendo de `0` a `i`, solo que ahora se imprime `char('A' + j)` en lugar de `*`.

**Pattern 15 (letras decrecientes):** el espejo de Pattern 14, con la misma lógica que Patterns 5-6 pero con letras: el loop interno recorre de `n-1` hasta `i` (decreciente), imprimiendo `char('A' + j)` en cada paso.

**Pattern 16 (letra repetida):** es Pattern 4, cambiando el número fijo por una letra: el loop interno va de `0` a `i` imprimiendo siempre `char('A' + i)`, la misma letra repetida `i+1` veces.

**Pattern 17 (letras en pirámide espejo):** espacios decrecientes (`n-1-i`), letras ascendentes de A hasta la letra `i` (`char('A'+j)`), y letras descendentes en espejo sin repetir la letra del pico (`char('A'+j-1)` con `j` de `i` a `1`).

**Pattern 18 (letras invertidas):** fila `i` imprime las letras desde `char('A' + n-1-i)` hasta `char('A' + n-1)`, siempre terminando en la misma letra final.

**Lo que más me sirvió de este bloque:** darme cuenta de que para trabajar con letras sin necesitar un array, C++ permite sumar un entero directamente a un `char`: `char('A' + k)` da la k-ésima letra después de la A. Es mucho más simple y menos propenso a errores que declarar un array de 26 letras — y, como se ve en 13-16, casi todos estos patrones terminan siendo simplemente los primeros seis pero cambiando *qué* se imprime, no *cómo* se cuenta. Ese fue el momento en el que este bloque completo dejó de sentirse como siete problemas nuevos y pasó a sentirse como reciclar cuatro fórmulas que ya tenía.

## Bloque 5 — Figuras compuestas por cuadrantes (Patterns 19-21)

**Pattern 19 (reloj de arena):** dos bloques — mitad superior con asteriscos exteriores decrecientes (`n-i`) y espacio central creciente (`2i`), y mitad inferior en espejo exacto de la superior.

**Pattern 20 (mariposa):** mitad superior con asteriscos crecientes (`i+1`) y espacio central decreciente (`2*(n-i-1)`); mitad inferior igual pero recorriendo `i` de `1` a `n-1` (no desde 0), para no duplicar la fila completa de asteriscos.

**Pattern 21 (marco hueco):** este fue, para mi sorpresa, el más simple de todo el bloque una vez que dejé de pensarlo como "figura" y empecé a pensarlo como condición: la fila superior e inferior son completas de asteriscos, y en las filas intermedias, un simple condicional (`if (j==0 || j==n-1)`) decide si el carácter es `*` o espacio, según si `j` está en el borde de la fila.

**Técnica clave:** cuando una figura tiene simetría entre mitad superior e inferior, me convino resolver una mitad primero y anotar exactamente qué cambia en la otra (el rango de `i`, o si se repite o no la fila central) en vez de escribir cada mitad desde cero por separado, como hice ingenuamente las primeras veces.

## Bloque 6 — El patrón más avanzado: la caja espiral numérica (Pattern 22)

Este fue, sin dudas, el que más me hizo replantear el enfoque de "fila por fila" que había usado en los 21 anteriores. Pattern 22 es una matriz cuadrada donde el valor de cada celda depende de qué tan cerca está del borde — y tratar de resolverlo con loops encadenados por fila, al estilo de los patrones anteriores, se vuelve mucho más complicado de lo necesario. Se resuelve muchísimo más simple con una fórmula por celda:

```cpp
class Solution {
public:
    void pattern22(int n) {
        int totalRows = 2 * n - 1;
        for (int i = 0; i < totalRows; i++) {
            for (int j = 0; j < totalRows; j++) {
                int top = i;
                int bottom = totalRows - 1 - i;
                int left = j;
                int right = totalRows - 1 - j;
                int minDist = min({top, bottom, left, right});
                cout << (n - minDist) << " ";
            }
            cout << "\n";
        }
    }
};
```

**La idea:** cada celda `(i,j)` de una matriz `(2n-1)×(2n-1)` imprime `n - distancia_al_borde_más_cercano`, calculando esa distancia como el mínimo entre la distancia al borde superior, inferior, izquierdo y derecho. Es la técnica más general de todo el bloque, y la que más me quedó grabada: en vez de pensar "fila por fila", pensar "celda por celda" y su relación con los cuatro bordes de la figura. Un cambio de perspectiva chico, pero que cambia completamente cómo se aborda el problema.

## Complejidad de toda la sección

Los 22 patrones comparten la misma complejidad: **O(n²) temporal** (dos o tres loops anidados, todos acotados por `n`) y **O(1) espacial** (no se usa memoria adicional proporcional a `n`, más allá de un par de variables auxiliares).

## Cómo abordo un patrón nuevo que no está en esta lista

Después de los 22, terminé con una especie de checklist mental que uso cada vez que me aparece una figura que no reconozco:

1. Dibujo la figura para un `n` pequeño (4 o 5) y numero las filas desde 0.
2. Para cada fila, identifico cuántos espacios y cuántos caracteres visibles hay, y escribo esas cantidades como función de `i` y `n`.
3. Si la figura tiene simetría (pirámide + pirámide invertida, o espejo izquierda-derecha), resuelvo primero una mitad conocida y adapto la fórmula para la otra.
4. Si el valor a imprimir cambia por celda según su posición (no solo por fila), lo pienso como distancia a los bordes, igual que en Pattern 22.
5. Reviso siempre el salto de línea al final de cada fila — sigue siendo, para mí, la causa más común de que una lógica correcta termine dando un resultado que no se parece en nada a lo esperado.

## Qué me llevo de este bloque

Si algo me queda claro después de los 22 patrones es que ninguno de ellos era realmente difícil por separado — el desafío estaba en no perder la paciencia con los primeros seis, que se sienten repetitivos, y confiar en que esa base iba a pagar dividendos más adelante. Y pagó: para cuando llegué a Pattern 22, ya no estaba pensando en "cómo imprimo esta figura" sino en "cómo describo esta figura en términos de fila, columna y distancia" — que es, en el fondo, el mismo tipo de razonamiento que después se necesita para matrices, DP 2D, y bastante más de lo que hace pensar un ejercicio de asteriscos.

Que la Fuerza (del `git commit`) los acompañe. Nos vemos en el próximo post.
