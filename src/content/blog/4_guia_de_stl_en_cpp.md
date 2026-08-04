---
title: 'STL en C++: la guía que me hubiera gustado tener'
description: 'Un recorrido completo por los containers y algoritmos más usados de la Standard Template Library — sintaxis, complejidades y los errores que casi todos cometemos al empezar.'
heroImage: '../../assets/posts/4.png'
pubDate: '2026-08-02'
---

## Introducción

Cuando empecé a resolver problemas de algoritmos en C++, mi primer instinto siempre fue el mismo: declarar un array, un par de variables auxiliares, y reinventar a mano cualquier estructura que necesitara. Funcionaba, pero era lento de escribir y fácil de romper — un `sort` manual con bubble sort, un stack simulado con un array y un índice, un "diccionario" hecho con dos vectores paralelos buscando linealmente.

La **STL** (Standard Template Library) es la respuesta de C++ a ese problema: una colección de containers (estructuras de datos genéricas) y algoritmos, ya implementados, probados y optimizados, que cubren el 90% de lo que necesitás para resolver un problema típico de programación competitiva o una entrevista técnica. En vez de escribir tu propio árbol balanceado o tu propia hash table, la STL te da `set` y `unordered_map`. En vez de un ordenamiento casero, te da `sort()` con complejidad garantizada.

Últimamente estuve repasando a fondo esta parte de C++ como preparación para seguir avanzando en algoritmos, y quise dejar todo documentado en un solo lugar — tanto para mí en el futuro como por si le sirve a alguien más que esté en el mismo punto del camino. Este post es justamente eso: un recorrido por los containers y algoritmos más usados, con su sintaxis, su complejidad, y (lo más importante, para mí) los errores típicos que te terminan mordiendo si no los conocés de antemano.

Una aclaración sobre el formato: cuando una función se repite entre containers (como `push`, `pop`, `find`) la explico en detalle la primera vez que aparece, y en las siguientes solo la menciono con su complejidad — salvo que cambie algo respecto a la primera definición.

---

## Containers secuenciales y utilidades básicas

### `pair`

Agrupa dos valores, potencialmente de tipos distintos, en una sola unidad. Es la base sobre la que se construyen varios containers de la STL (`map` internamente almacena `pair<clave, valor>`).

Syntax: `pair<tipo1, tipo2> p;`

Declarar e inicializar: `pair<int, string> p = {1, "uno"};` | O(1)

Acceder: `p.first`, `p.second` | O(1)

Comparación: lexicográfica — compara `.first` primero, y usa `.second` como desempate si `.first` es igual.

```cpp
pair<int, int> nested = {10, 20};
```

### `vector`

Array dinámico que crece automáticamente. Es el container por defecto en C++: si no tenés una razón puntual para usar otra cosa, usás `vector`.

Syntax: `vector<tipo> v;`

Insertar al final: `v.push_back(x);` | O(1) amortizado

Acceder por índice: `v[i]` | O(1)

Acceso con chequeo de límites: `v.at(i)` | O(1), lanza excepción si `i` está fuera de rango

Primer y último elemento: `v.front()`, `v.back()` | O(1)

Quitar el último: `v.pop_back();` | O(1)

Insertar en posición arbitraria: `v.insert(v.begin() + i, x);` | O(n)

Borrar en posición arbitraria: `v.erase(v.begin() + i);` | O(n)

Iteradores al inicio y al final: `v.begin()`, `v.end()` | O(1) — `begin()` apunta al primer elemento, `end()` apunta a la posición *después* del último (no es un elemento válido, es un centinela para saber dónde parar de iterar).

Tamaño: `v.size();` | O(1)

Vacío: `v.empty();` | O(1) — equivalente a `v.size() == 0`, pero más idiomático (y en algunos containers, más eficiente).

Vaciar todo: `v.clear();` | O(n) — destruye todos los elementos; el container queda con `size() == 0`.

```cpp
vector<int> v = {1, 2, 3};
for (int x : v) cout << x << " ";
for (auto it = v.begin(); it != v.end(); ++it) cout << *it << " ";
```

**Gotcha:** insertar/borrar en el medio (o cualquier operación que dispare *reallocation*) invalida iteradores, punteros y referencias existentes hacia el vector.

### `list`

Lista doblemente enlazada. Se usa cuando necesitás insertar/borrar en cualquier posición sin pagar el costo de desplazar elementos — pero perdés el acceso por índice.

Syntax: `list<tipo> l;`

Insertar al inicio: `l.push_front(x);` | O(1)

Insertar/borrar con iterador ya posicionado: O(1) — porque solo hay que reenlazar punteros, sin desplazar nada.

Ordenar: `l.sort();` | O(n log n) — `std::sort` no funciona acá porque `list` no tiene iteradores de acceso aleatorio.

`l.begin()`, `l.end()`, `l.size()`, `l.empty()`, `l.clear()` | O(1), O(1), O(1), O(1), O(n) — mismas complejidades que en `vector`.

**Gotcha:** no existe `l[i]`. Si necesitás acceso por índice frecuente, `list` es el container equivocado.

### `deque`

Array dinámico organizado en bloques de memoria, que permite inserción/borrado eficiente en **ambos** extremos (a diferencia de `vector`, que solo es eficiente al final), manteniendo acceso por índice O(1).

Syntax: `deque<tipo> d;`

Insertar al inicio: `d.push_front(x);` | O(1) amortizado

Quitar del inicio: `d.pop_front();` | O(1)

El resto de las operaciones (`push_back`, `pop_back`, `[i]`, `insert`, `erase`, `begin()`, `end()`, `size()`, `empty()`, `clear()`) se comportan igual que en `vector`, con las mismas complejidades.

**Gotcha:** no es un bloque contiguo de memoria como `vector` — no podés pasar `&d[0]` esperando un array de C.

---

## Adapters: `stack`, `queue`, `priority_queue`

Los adapters no son estructuras de datos propias — envuelven otro container (`deque` por defecto) y exponen una interfaz restringida a propósito.

### `stack`

Interfaz LIFO (último en entrar, primero en salir). Útil para backtracking, DFS iterativo, evaluación de expresiones, el patrón "monotonic stack".

Syntax: `stack<tipo> s;`

Insertar: `s.push(x);` | O(1) amortizado

Ver el tope: `s.top();` | O(1)

Quitar el tope: `s.pop();` | O(1) — **no devuelve el valor**, hay que guardarlo con `top()` antes.

Vacío / tamaño: `s.empty();`, `s.size();` | O(1)

**Gotcha:** no tiene `begin()`/`end()` — no se puede iterar sin destruir la estructura. Y llamar `top()`/`pop()` sobre un stack vacío es comportamiento indefinido.

### `queue`

Interfaz FIFO (primero en entrar, primero en salir). El uso más común: BFS.

Syntax: `queue<tipo> q;`

Insertar (siempre al final): `q.push(x);` | O(1) amortizado

Ver el frente: `q.front();` | O(1)

Ver el final: `q.back();` | O(1)

Quitar del frente: `q.pop();` | O(1) — tampoco devuelve el valor.

```cpp
queue<int> q;
q.push(1); q.push(2);
cout << q.front(); // 1
q.pop();
```

**Gotcha:** en BFS, marcar "visitado" recién al hacer `pop()` (en vez de al hacer `push()`) hace que el mismo nodo se encole varias veces, inflando la complejidad real.

### `priority_queue`

Implementa un heap — siempre da acceso al elemento de mayor prioridad, sin importar el orden de inserción. Max-heap por defecto.

Syntax (max-heap): `priority_queue<tipo> pq;`

Syntax (min-heap): `priority_queue<tipo, vector<tipo>, greater<tipo>> pq;`

Insertar: `pq.push(x);` | O(log n)

Ver el máximo (o mínimo, en un min-heap): `pq.top();` | O(1)

Quitar el máximo/mínimo: `pq.pop();` | O(log n)

Construir desde un rango existente: `priority_queue<int> pq(v.begin(), v.end());` | O(n) — usa *heapify*, no O(n log n).

`pq.size()`, `pq.empty()` | O(1) — igual que en `stack`/`queue`. No tiene `begin()`/`end()` ni `clear()`, por la misma razón que `stack`: la interfaz está restringida a propósito.

**Gotcha:** `less<int>` da max-heap (comportamiento por defecto), `greater<int>` da min-heap — es contraintuitivo la primera vez. Y no hay forma de modificar un elemento arbitrario ya insertado (no hay *decrease-key* directo).

---

## Containers asociativos: la familia `set`

Todos internamente son árboles balanceados (excepto los `unordered_*`, que usan hash table). La diferencia entre las cuatro variantes es únicos vs. duplicados, y ordenado vs. sin orden.

### `set`

Valores únicos, ordenados automáticamente.

Syntax: `set<tipo> s;`

Insertar: `s.insert(x);` | O(log n) — si `x` ya existe, se ignora silenciosamente.

Buscar: `s.find(x);` | O(log n) — devuelve iterador, `s.end()` si no existe.

Contar ocurrencias: `s.count(x);` | O(log n) — siempre 0 o 1 en `set`.

Borrar: `s.erase(x);` | O(log n)

Mínimo / máximo: `*s.begin()`, `*s.rbegin()` | O(1)

Primer elemento ≥ x: `s.lower_bound(x);` | O(log n)

Primer elemento > x: `s.upper_bound(x);` | O(log n)

`s.begin()`, `s.end()`, `s.size()`, `s.empty()`, `s.clear()` | O(1), O(1), O(1), O(1), O(n) — mismas complejidades que en `vector`/`list`, con la diferencia de que iterar de `begin()` a `end()` en `set` siempre recorre los elementos en orden ascendente.

**Gotcha:** usar las funciones libres `lower_bound`/`upper_bound` de `<algorithm>` sobre un `set` (en vez del método miembro) degrada la búsqueda a O(n), porque el iterador de `set` no es de acceso aleatorio.

### `multiset`

Igual que `set`, pero permite duplicados.

`count(x)` ahora cuesta O(log n + k), con k = ocurrencias de x, porque hay que contarlas todas.

**Gotcha, el más importante de esta variante:** `ms.erase(x)` borra **todas** las ocurrencias de x. Para borrar solo una: `ms.erase(ms.find(x));`.

### `unordered_set`

Hash table — inserción, borrado y búsqueda en O(1) promedio (O(n) en el peor caso, por colisiones). Sin orden garantizado al iterar.

`us.begin()`, `us.end()`, `us.size()`, `us.empty()`, `us.clear()` | mismas complejidades que en `set`, pero iterar de `begin()` a `end()` **no** da ningún orden garantizado.

**Gotcha crítico en jueces competitivos:** el hash por defecto de GCC es vulnerable a ataques de colisión deliberada — un test case malicioso puede degradar tu solución de O(n) a O(n²) real. La mitigación es un hash custom (combinando el valor con `chrono` para aleatorizar la semilla).

### `unordered_multiset`

Combina las dos anteriores: hash table + duplicados. Hereda el gotcha de `erase(x)` borrando todo, y el riesgo anti-hash. En la práctica, es la variante menos usada de las cuatro — la mayoría de los casos se resuelven más simple con `unordered_map<valor, frecuencia>`.

---

## Containers asociativos: la familia `map`

### `map`

Pares clave-valor únicos, ordenados por clave.

Syntax: `map<clave, valor> m;`

Insertar o actualizar: `m[clave] = valor;` | O(log n) — **inserta silenciosamente si la clave no existía**.

Insertar explícito: `m.insert({clave, valor});` | O(log n)

Acceso seguro: `m.at(clave);` | O(log n), lanza excepción si no existe.

`m.begin()`, `m.end()`, `m.size()`, `m.empty()`, `m.clear()` | O(1), O(1), O(1), O(1), O(n) — igual que en `set`, iterar de `begin()` a `end()` recorre los pares en orden ascendente de clave.

Patrón de conteo de frecuencias:

```cpp
map<int, int> freq;
for (int x : nums) freq[x]++;
```

**Gotcha más importante de todo `map`:** usar `operator[]` para solo *verificar* si una clave existe (`if (m[clave] == algo)`) la inserta si no estaba, con valor por defecto. Para verificar sin insertar, usar `m.find(clave) != m.end()`.

### `multimap`

Igual que `map`, pero permite claves duplicadas — una clave puede tener varios valores asociados.

**No existe `operator[]`** (no tendría sentido con claves repetidas) — hay que usar `insert()`.

Obtener todos los valores de una clave: `auto range = mm.equal_range(clave);` | O(log n) para encontrar el rango, luego O(k) para recorrerlo.

En la práctica, casi siempre resulta más simple usar `map<clave, vector<valor>>` en vez de `multimap`.

### `unordered_map`

Como `map`, pero con hash table: O(1) promedio en vez de O(log n). Probablemente el container con más impacto real en la complejidad de una solución — la diferencia entre O(n log n) y O(n) total.

`um.begin()`, `um.end()`, `um.size()`, `um.empty()`, `um.clear()` | mismas complejidades que en `map`, pero sin orden garantizado al iterar — igual que pasa entre `set` y `unordered_set`.

**Gotcha:** hereda el riesgo anti-hash de `unordered_set`. Y si la clave es un tipo compuesto (como `pair<int,int>`), no compila sin proveer un hash custom — a diferencia de `map`, que solo necesita `operator<`.

### `unordered_multimap`

La combinación final: hash table + claves duplicadas. Hereda todos los gotchas de sus dos "padres". Es la variante que menos vas a usar de las ocho — casi siempre `unordered_map<clave, vector<valor>>` es la opción más simple y legible.

---

## Algoritmos más usados de `<algorithm>`

### `sort()`

Ordena un rango in-place.

Syntax: `sort(v.begin(), v.end());` | O(n log n)

Descendente: `sort(v.begin(), v.end(), greater<int>());`

Comparador custom: `sort(v.begin(), v.end(), [](auto& a, auto& b){ return a.second < b.second; });`

**Gotcha:** el comparador tiene que ser un orden estricto — usar `<=` en vez de `<` es comportamiento indefinido, y a veces "parece funcionar" en inputs chicos, lo que hace el bug difícil de detectar. Y `sort()` no es estable — para eso existe `stable_sort()`, con la misma complejidad.

### `min_element()` / `max_element()`

Devuelven un **iterador** al menor/mayor elemento de un rango, sin requerir que esté ordenado.

Syntax: `auto it = min_element(v.begin(), v.end());` | O(n)

```cpp
int minimo = *min_element(v.begin(), v.end());
```

Ambos en una sola pasada: `auto [itMin, itMax] = minmax_element(v.begin(), v.end());` | O(n), pero con menos comparaciones que llamar a las dos por separado.

**Gotcha:** devuelven un iterador, no el valor — hay que desreferenciar. Y si el rango puede estar vacío, hay que chequearlo antes: devuelven `end()`, y desreferenciar eso es UB.

### `next_permutation()`

Transforma un rango en su siguiente permutación en orden lexicográfico.

Syntax: `next_permutation(v.begin(), v.end());` | O(n) por llamada

Generar todas las permutaciones:

```cpp
sort(v.begin(), v.end()); // arrancar desde la primera permutación
do {
    // procesar v
} while (next_permutation(v.begin(), v.end()));
```

**Gotcha:** devuelve `bool` — `false` cuando el rango estaba en la última permutación posible (y lo reordena de vuelta a la primera). Generar todas las permutaciones es O(n! × n) total — inviable para n mayor a 10-12.

### `__builtin_popcount()`

Extensión de GCC/Clang (no es parte del estándar) que cuenta bits en 1 de un entero. Se usa mucho en bit manipulation y DP sobre bitmask.

Syntax: `__builtin_popcount(x);` | O(1) en la práctica

Para `long long`: `__builtin_popcountll(x);`

**Gotcha:** usar la versión de 32 bits con un `long long` trunca el valor silenciosamente — para 64 bits, siempre `__builtin_popcountll`.

---

## Cierre

Esto no es, ni de cerca, toda la STL — quedan afuera cosas como `tuple`, `bitset`, o el resto de `<algorithm>` (`accumulate`, `binary_search`, `count`, etc.) que probablemente terminen en un post futuro a medida que los vaya necesitando. Pero entre los containers y los algoritmos de acá arriba está, en la práctica, el 90% de lo que uso al resolver un problema típico.

Si hay algo que me llevo de este repaso es que la mayoría de los bugs que meto con STL no son de lógica del algoritmo, sino de no conocer el contrato exacto de la herramienta que estoy usando — un `erase(valor)` que borra más de lo que pensaba, un `operator[]` que inserta cuando solo quería consultar, un comparador con `<=` que compila perfecto y explota en producción. Vale la pena aprenderse estos detalles una vez, en frío, en vez de descubrirlos a mitad de un contest con el reloj corriendo.

Que la Fuerza (del `git commit`) los acompañe. Nos vemos en el próximo post.
