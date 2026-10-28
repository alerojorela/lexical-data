# Datos léxicos del castellano

> **Read this document in English: [`README.md`](README.md).**

Dos tablas en texto separado por tabuladores, con la flexión y la categoría
gramatical del castellano.

| | |
|---|---|
| `form-lemma-root-pos.tsv.gz` | **forma, lema, raíz y categoría**. De una forma se llega a sus lemas (*fui* → *ser*, *ir*) y de un lema a todas sus formas |
| `variant-lemma.tsv` | las **remisiones**: qué grafía lleva a qué lema (*cinc* → *zinc*, *trasfusor* → *transfusor*) |

La primera va **comprimida**: en claro son 80 MB. No hace falta descomprimirla
para usarla —`zcat` en la consola, y pandas la abre tal cual—, pero sí leerla
**como texto**:

```python
pd.read_csv('form-lemma-root-pos.tsv.gz', sep='\t',
            dtype=str, keep_default_na=False)
```

⚠️ Sin eso se pierden **85 filas**: la raíz de *nana*, *nanear* y *nano* es
`nan`, y pandas la convierte en ausente sin avisar.

## Las columnas

| | |
|---|---|
| `pos` | la categoría: `NOUN`, `ADJ`, `VERB`, `ADV`, `INTERJ`, `PRON`, `PREP`, `CONJ`, `DET` |
| `lemma` | el lema, sin la terminación de femenino y sin número de homógrafo |
| `form` | la forma, y **siempre una sola palabra**. El lema es también forma de sí mismo |
| `gender` | `m.`, `f.` o `m. y f.` |
| `number` | el número **nominal**: `sg` o `pl` en el nombre, el adjetivo, el pronombre y el determinante. **Vacío en el verbo**, y no por falta de dato: el número que lleva una forma verbal es el de **su persona** —concordancia— y vive en `person_pronoun` y `person_id`. No es la misma categoría: el número del verbo, en las lenguas que lo tienen, dice que la acción ocurrió más de una vez, y eso el castellano no lo marca |
| `mood` | `indicativo`, `subjuntivo` o `imperativo`; vacío en las formas no personales |
| `person_pronoun` | `yo`, `tú`, `vos`, `usted`, `él`, `nosotros`, `vosotros`, `ustedes`, `ellos` |
| `person_id` | la misma persona en clave: `1s`, `2s_tu`, `2s_vos`, `2s_usted`, `3s`, `1p`, `2p_vosotros`, `2p_ustedes`, `3p` |
| `tense` | `Infinitivo`, `Participio`, `Gerundio`, `Presente`, `Pretérito imperfecto o Copretérito`, `Pretérito perfecto simple o Pretérito`, `Futuro simple o Futuro`, `Condicional simple o Pospretérito`, `Pretérito imperfecto o Pretérito`, `Imperativo` |
| `tense_id` | el número del tiempo, estable entre ediciones |
| `root` | la raíz, por *stemming*. Ver más abajo: **no es un morfema** |

## Qué conviene saber antes de usarla

- **Los tiempos compuestos no están**, y no hacen falta: *hubiera cantado* son
  dos formas que la tabla ya trae por separado, una de `haber` y el participio
  de `cantar`.
- **Cada fila es una palabra.** No hay nada pluriverbal: ni locuciones, ni
  `acid house`, ni `agnus Dei`. Si no se escribe junto, no es una forma.
- **En el verbo pronominal, donde el pronombre va adherido es la única forma
  que entra**, y la desnuda no existe para ese lema:

  | ✅ entra | ❌ no entra |
  |---|---|
  | `apozarse` | `apozar` |
  | `apozándose` | `apozando` |
  | `apózate`, `apozate` (voseo), `apócese`, `apozaos`, `apócense` | `apoza` como imperativo |

  Son las siete ranuras en las que el castellano escribe enclisis —infinitivo,
  gerundio e imperativo afirmativo— en 620 lemas. El verbo es `apozarse`, y
  `apozar` no es una palabra suya.
- **Y donde el pronombre es otra palabra, se descarta el pronombre**: de
  `te apozas` entra `apozas`, que remite igual a `apozarse`. El participio no
  lleva pronombre nunca: `apozado`.
- ⚠️ **Lo que no hay es la enclisis del castellano**: `dámelo` y
  `escribiéndoselas` no están. Lo que está completo es el paradigma reflexivo.
- **Los homónimos son indistinguibles.** `ribera` la orilla y `ribera` el vino
  comparten lema, y por eso una misma combinación de categoría, lema y forma
  puede aparecer con dos géneros.
- **Una forma repetida no es un error**: es sincretismo. `cante` es primera y
  tercera persona del presente de subjuntivo, y cada una es una fila.
- **El femenino y el plural están derivados por regla**, no copiados de una
  lista. El del plural está más abajo entero, con sus excepciones; lo que hay
  que saber al usarlo: *jabalí* trae **dos plurales** porque los dos son
  correctos, el plural invariable aparece **dos veces** —`crisis` como `sg` y
  como `pl`—, y **el pronombre y el determinante no tienen plural** en la tabla
  porque el suyo es supletivo.
- **La raíz no es un morfema**, y conviene leer lo que es antes de agrupar por
  ella: está más abajo.

## Cómo se deriva el plural

El plural **no está copiado de una lista: se deriva**, y aquí están las reglas,
para que se pueda auditar una forma sin tener que creerse la tabla. Son las
reglas a)–k) de [la entrada «plural» del *Diccionario panhispánico de
dudas*](https://www.rae.es/dpd/plural), que es la norma; lo que aquí se añade es
cómo se reescribe el acento y qué se queda fuera.

**Solo el nombre y el adjetivo.** El pronombre y el determinante tienen plural,
pero supletivo —*yo/nosotros*, *el/los*—, y no sale de ninguna regla. Tampoco se
pluralizan los afijos, los pluriverbales (`acid house`), las siglas en
mayúsculas (`ADN`, que por escrito no varía) ni **el lema que ya es plural**.

1. **Vocal átona, o `-á`, `-é`, `-ó` tónicas** → `+s`: `casa → casas`,
   `tribu → tribus`, `sofá → sofás`, `café → cafés`.
2. **`-í`, `-ú` tónicas** → **las dos formas**, `+es` y `+s`, porque las dos son
   correctas y la tabla no elige: `jabalí → jabalíes, jabalís`.
3. **`-y`**: tras vocal, `+es` —`rey → reyes`—; tras consonante, la `y` se vuelve
   `i` y `+s` —`dandy → dandis`—.
4. **`-s`, `-x`**: con el acento en la última sílaba crecen —`país → países`,
   `mes → meses`—; si no, **no varían**, y entonces la misma forma aparece dos
   veces, como `sg` y como `pl`: `crisis`, `lunes`, `tórax`.
5. **`-l`, `-r`, `-n`, `-d`, `-z`, `-j`** → `+es`, con la `z` que se vuelve `c`:
   `color → colores`, `luz → luces`, `reloj → relojes`.
6. **Pero si antes de esa consonante hay otra** —grupo consonántico— → `+s`:
   `récord → récords`, `western → westerns`. Esta regla va antes que la anterior
   porque el final es el mismo y la regla, la contraria.
7. **Cualquier otra consonante** → `+s`: `cómic → cómics`, `chip → chips`,
   `mamut → mamuts`.
8. **El acento se reescribe** para que el golpe de voz no se mueva:
   - cae cuando deja de hacer falta: `canción → canciones`, `inglés → ingleses`;
   - **aparece** cuando la palabra se vuelve esdrújula: `joven → jóvenes`,
     `imagen → imágenes`, `lápiz → lápices`;
   - y la tilde de hiato se queda siempre: `país → países`, `raíz → raíces`.
9. **Los irregulares**, en una tabla, porque ninguna regla los da: los tres que
   **mueven el acento de sílaba** (`carácter → caracteres`,
   `régimen → regímenes`, `espécimen → especímenes`), los tres demostrativos
   (`este → estos`) y `jean → jeans`, que por la regla de la `-n` daría
   `jéanes`.

<!-- RAMAS -->
<!-- cifras medidas sobre la tabla: no editar a mano -->

### Cuántas palabras caen en cada regla

Contado sobre las **96.533 formas nominales en singular** de la tabla, y la suma cuadra con el total, que es la comprobación de que no hay un caso sin clasificar:

| regla | formas | p. ej. |
|---|---:|---|
| a) y b) vocal → `+s` | 77.322 | `Adviento` |
| g) e i) `-l -r -n -d -z -j`, `-ch` → `+es` | 15.329 | `Alcorán` |
| f) `-s`, `-x` átona → **invariable** | 1.713 | `Corpus` |
| h) otra consonante → `+s` | 699 | `accésit` |
| f) `-s`, `-x` tónica → `+es` | 504 | `Pentecostés` |
| c) `-í`, `-ú` tónicas → **dos plurales** | 443 | `abadí` |
| d) y e) `-y` | 187 | `abey` |
| sin plural: afijo o rareza | 158 | `-sarcoma` |
| sin plural: sigla | 98 | `ABS` |
| j) grupo consonántico → `+s` | 66 | `algund` |
| sin plural: sin vocales | 7 | `Rh` |
| irregulares, en tabla | 7 | `aquel` |
| **total** | **96.533** | |

O sea que **el 96,0 % sale por dos reglas** —la vocal y la consonante simple—, y lo que hay escrito a mano son **7 palabras**. Las demás «excepciones» son reglas con su propio motivo ortográfico, no listas.

<!-- /RAMAS -->

### Lo que la derivación no acierta

| | |
|---|---|
| **los latinismos** | `álbum → álbums` por la regla general, y lo asentado es *álbumes*. Son 128 lemas en `-m`, casi todos préstamos, y se resuelven uno a uno, no por regla |
| **el femenino sin par en el lema** | el femenino se deriva de la terminación que trae el lema (`deitano, na` → `deitana`); donde no la hay no hay nada que derivar, así que `actriz` entra como lema propio y no como forma de `actor` |

## Qué es la columna `root`, y qué no es

Es el resultado de aplicar al **lema** el algoritmo de *stemming* de **Snowball
para el castellano**: un recortador de terminaciones por reglas. Cuatro cosas que
hay que saber antes de usarla:

- **Sale del lema, no de la forma.** Todas las filas de un lema llevan la misma
  raíz: `luces` lleva la de `luz`. No es una segmentación de la forma.
- **No es un morfema ni una raíz etimológica**, es una **truncación**: `gato` →
  `gat`, `niño` → `niñ`, `cantar` → `cant`.
- **Pierde las tildes**, que es cómo acaba el algoritmo: `canción` → `cancion`,
  `vacío` → `vaci`, `álbum` → `album`.
- **Sirve para agrupar familias, y también agrupa de más.** Bien:
  `color`, `coloración`, `colorado`, `colorante` y `colorar` comparten `color`.
  Mal: `par` (la pareja) y `parar` (detener) comparten `par`, y no son familia.
  Y agrupa de menos: `cabeza` → `cabez` pero `cabezal` → `cabezal`.

Y lo que hace en conjunto está en las cifras de abajo: **tres de cada cuatro
raíces tienen un solo lema**, o sea que para la mayoría la columna no agrupa
nada. Es una ayuda de búsqueda aproximada, no un análisis morfológico.

<!-- ESTADISTICAS -->
<!-- cifras medidas sobre la tabla: no editar a mano -->

## Qué hay dentro

| | |
|---|---:|
| filas | 1.098.090 |
| lemas | 92.086 |
| formas distintas | 747.416 |
| raíces distintas | 63.653 |
| raíces con un solo lema | 48.736 |
| remisiones en `variant-lemma.tsv` | 2.990 |

## Por categoría

| `pos` | filas | lemas | formas |
|---|---:|---:|---:|
| VERB | 869.300 | 12.315 | 564.123 |
| NOUN | 136.537 | 60.002 | 131.601 |
| ADJ | 89.173 | 26.179 | 88.695 |
| ADV | 2.318 | 2.212 | 2.283 |
| INTERJ | 486 | 458 | 479 |
| PRON | 163 | 100 | 131 |
| CONJ | 54 | 52 | 54 |
| PREP | 52 | 50 | 51 |
| DET | 7 | 4 | 6 |

## El género y el número

| `gender` | filas |
|---|---:|
| — | 888.294 |
| m. | 107.029 |
| f. | 97.320 |
| m. y f. | 5.447 |

| `number` | filas |
|---|---:|
| — (el verbo no lo lleva: su número es el de la persona) | 863.277 |
| sg | 121.690 |
| pl | 113.123 |

## La flexión verbal

**869.300 filas** de 12.315 lemas.

| `mood` · `tense` | filas |
|---|---:|
| subjuntivo · Pretérito imperfecto o Pretérito | 170.090 |
| indicativo · Presente | 95.886 |
| subjuntivo · Presente | 85.307 |
| indicativo · Futuro simple o Futuro | 85.091 |
| indicativo · Condicional simple o Pospretérito | 85.091 |
| indicativo · Pretérito perfecto simple o Pretérito | 85.065 |
| indicativo · Pretérito imperfecto o Copretérito | 85.059 |
| subjuntivo · Futuro simple o Futuro | 85.045 |
| imperativo · Imperativo | 53.249 |
| Participio | 10.680 |
| Infinitivo | 10.663 |
| Gerundio | 10.655 |
| — | 7.419 |

| `person_pronoun` | filas |
|---|---:|
| usted | 106.406 |
| ustedes | 106.386 |
| tú | 106.384 |
| vosotros | 106.268 |
| él | 96.026 |
| ellos | 95.816 |
| yo | 95.726 |
| nosotros | 95.640 |
| — (forma no personal o solo el lema) | 39.417 |
| vos | 21.231 |

<!-- /ESTADISTICAS -->

## Licencia

**CC0 1.0: dominio público, y no hace falta citar a nadie.** Copiar,
redistribuir, transformar y vender, sin pedir permiso y sin atribución. El texto
está en [`LICENSE`](LICENSE).

Y la razón de que no pida atribución es la que tiene: **aquí no hay obra
propia**. Esto es flexión del castellano aplicada con reglas, y las reglas y las
palabras no son de nadie. Exigir una cita por copiar una tabla de plurales sería
cobrar peaje por el idioma.
