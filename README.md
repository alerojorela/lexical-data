# Spanish lexical data

> **Este documento en castellano: [`README.es.md`](README.es.md).**

Two tab-separated text tables with the inflection and the part of speech of
Spanish.

| | |
|---|---|
| `form-lemma-root-pos.tsv.gz` | **form, lemma, root and part of speech**. From a form you reach its lemmas (*fui* → *ser*, *ir*) and from a lemma all its forms |
| `variant-lemma.tsv` | the **cross-references**: which spelling leads to which lemma (*cinc* → *zinc*, *trasfusor* → *transfusor*) |

The first one is **gzipped**: in plain text it is 80 MB. You do not need to
unpack it to use it —`zcat` on the command line, and pandas opens it as is—
but you do need to read it **as text**:

```python
pd.read_csv('form-lemma-root-pos.tsv.gz', sep='\t',
            dtype=str, keep_default_na=False)
```

⚠️ Without that you lose **85 rows**: the root of *nana*, *nanear* and *nano*
is `nan`, and pandas turns it into a missing value with no warning.

## The columns

| | |
|---|---|
| `pos` | the part of speech: `NOUN`, `ADJ`, `VERB`, `ADV`, `INTERJ`, `PRON`, `PREP`, `CONJ`, `DET` |
| `lemma` | the lemma, without the feminine ending and without a homograph number |
| `form` | the form, and **always a single word**. The lemma is a form of itself too |
| `gender` | `m.`, `f.` or `m. y f.` |
| `number` | **nominal** number: `sg` or `pl` on the noun, the adjective, the pronoun and the determiner. **Empty on the verb**, and not for lack of data: the number a verb form carries is the number of **its person** —agreement— and it lives in `person_pronoun` and `person_id`. It is not the same category: verbal number, in the languages that mark it, says the action happened more than once, and Spanish does not mark that |
| `mood` | `indicativo`, `subjuntivo` or `imperativo`; empty on the non-finite forms |
| `person_pronoun` | `yo`, `tú`, `vos`, `usted`, `él`, `nosotros`, `vosotros`, `ustedes`, `ellos` |
| `person_id` | the same person as a key: `1s`, `2s_tu`, `2s_vos`, `2s_usted`, `3s`, `1p`, `2p_vosotros`, `2p_ustedes`, `3p` |
| `tense` | `Infinitivo`, `Participio`, `Gerundio`, `Presente`, `Pretérito imperfecto o Copretérito`, `Pretérito perfecto simple o Pretérito`, `Futuro simple o Futuro`, `Condicional simple o Pospretérito`, `Pretérito imperfecto o Pretérito`, `Imperativo` |
| `tense_id` | the number of the tense, stable across editions |
| `root` | the root, by *stemming*. See below: **it is not a morpheme** |

## What to know before using it

- **The compound tenses are not here**, and they are not needed: *hubiera
  cantado* is two forms the table already has separately, one of `haber` and the
  participle of `cantar`.
- **Every row is one word.** Nothing multiword: no idioms, no `acid house`, no
  `agnus Dei`. If it is not written joined, it is not a form.
- **In a reflexive verb, where the pronoun is written joined that is the only
  form that enters**, and the bare one does not exist for that lemma:

  | ✅ in | ❌ out |
  |---|---|
  | `apozarse` | `apozar` |
  | `apozándose` | `apozando` |
  | `apózate`, `apozate` (*voseo*), `apócese`, `apozaos`, `apócense` | `apoza` as an imperative |

  These are the seven slots where Spanish writes enclisis —infinitive, gerund
  and affirmative imperative— across 620 lemmas. The verb is `apozarse`, and
  `apozar` is not a word of its own.
- **And where the pronoun is a separate word, the pronoun is dropped**: from
  `te apozas` the table takes `apozas`, which leads to `apozarse` all the same.
  The participle never carries a pronoun: `apozado`.
- ⚠️ **What is not here is Spanish enclisis**: `dámelo` and `escribiéndoselas`
  are missing. What is complete is the reflexive paradigm.
- **Homonyms cannot be told apart.** `ribera` the riverbank and `ribera` the
  wine share a lemma, which is why one same combination of part of speech,
  lemma and form can show up with two genders.
- **A repeated form is not an error**: it is syncretism. `cante` is both first
  and third person of the present subjunctive, and each one is a row.
- **The feminine and the plural are derived by rule**, not copied from a list.
  The plural rules are below in full, with their exceptions; what you need to
  know when using them: *jabalí* has **two plurals** because both are correct,
  an invariable plural appears **twice** —`crisis` as `sg` and as `pl`—, and
  **the pronoun and the determiner have no plural** in the table because theirs
  is suppletive.
- **The root is not a morpheme**, and it is worth reading what it is before
  grouping by it: see below.

## How the plural is derived

The plural is **not copied from a list: it is derived**, and here are the rules,
so that a form can be audited without having to take the table on trust. They
are rules a)–k) of [the «plural» entry of the *Diccionario panhispánico de
dudas*](https://www.rae.es/dpd/plural), which is the norm; what is added here is
how the written accent is rewritten and what stays out.

**Nouns and adjectives only.** The pronoun and the determiner do have a plural,
but a suppletive one —*yo/nosotros*, *el/los*—, and no rule yields it. Affixes,
multiword entries (`acid house`), uppercase acronyms (`ADN`, invariable in
writing) and **a lemma that is already plural** are not pluralised either.

1. **Unstressed vowel, or stressed `-á`, `-é`, `-ó`** → `+s`: `casa → casas`,
   `tribu → tribus`, `sofá → sofás`, `café → cafés`.
2. **Stressed `-í`, `-ú`** → **both forms**, `+es` and `+s`, because both are
   correct and the table does not choose: `jabalí → jabalíes, jabalís`.
3. **`-y`**: after a vowel, `+es` —`rey → reyes`—; after a consonant, the `y`
   becomes `i` and `+s` —`dandy → dandis`—.
4. **`-s`, `-x`**: with the stress on the last syllable they grow —`país →
   países`, `mes → meses`—; otherwise they **do not change**, and then the same
   form appears twice, as `sg` and as `pl`: `crisis`, `lunes`, `tórax`.
5. **`-l`, `-r`, `-n`, `-d`, `-z`, `-j`** → `+es`, with `z` becoming `c`:
   `color → colores`, `luz → luces`, `reloj → relojes`.
6. **But if there is another consonant before that one** —a consonant cluster—
   → `+s`: `récord → récords`, `western → westerns`. This rule comes before the
   previous one, because the ending is the same and the rule is the opposite.
7. **Any other consonant** → `+s`: `cómic → cómics`, `chip → chips`,
   `mamut → mamuts`.
8. **The written accent is rewritten** so that the stress does not move:
   - it drops when it is no longer needed: `canción → canciones`,
     `inglés → ingleses`;
   - it **appears** when the word becomes stressed on the third-last syllable:
     `joven → jóvenes`, `imagen → imágenes`, `lápiz → lápices`;
   - and a hiatus accent always stays: `país → países`, `raíz → raíces`.
9. **The irregular ones**, in a table, because no rule yields them: the three
   that **move the stress to another syllable** (`carácter → caracteres`,
   `régimen → regímenes`, `espécimen → especímenes`), the three demonstratives
   (`este → estos`) and `jean → jeans`, which the `-n` rule would turn into
   `jéanes`.

<!-- RAMAS -->
<!-- figures measured on the table: do not edit by hand -->

### How many words fall in each rule

Counted over the **96,533 singular nominal forms** of the table, and the sum matches the total, which is the check that no case is left unclassified:

| rule | forms | e.g. |
|---|---:|---|
| a) and b) vowel → `+s` | 77,322 | `Adviento` |
| g) and i) `-l -r -n -d -z -j`, `-ch` → `+es` | 15,329 | `Alcorán` |
| f) unstressed `-s`, `-x` → **invariable** | 1,713 | `Corpus` |
| h) any other consonant → `+s` | 699 | `accésit` |
| f) stressed `-s`, `-x` → `+es` | 504 | `Pentecostés` |
| c) stressed `-í`, `-ú` → **two plurals** | 443 | `abadí` |
| d) and e) `-y` | 187 | `abey` |
| no plural: affix or oddity | 158 | `-sarcoma` |
| no plural: acronym | 98 | `ABS` |
| j) consonant cluster → `+s` | 66 | `algund` |
| no plural: no vowels | 7 | `Rh` |
| irregular, in a table | 7 | `aquel` |
| **total** | **96,533** | |

So **96.0 % comes from two rules** —the vowel and the single consonant—, and what is written by hand is **7 words**. The other 'exceptions' are rules with their own orthographic reason, not lists.

<!-- /RAMAS -->

### What the derivation does not get right

| | |
|---|---|
| **Latinisms** | `álbum → álbums` by the general rule, while the established plural is *álbumes*. They are 128 lemmas in `-m`, nearly all loanwords, and they are settled one by one, not by rule |
| **a feminine the lemma does not pair** | the feminine is derived from the ending the lemma carries (`deitano, na` → `deitana`); where there is none there is nothing to derive, so `actriz` enters as a lemma of its own and not as a form of `actor` |

## What the `root` column is, and what it is not

It is the result of applying the **Snowball stemmer for Spanish** to the
**lemma**: a rule-based ending trimmer. Four things to know before using it:

- **It comes from the lemma, not from the form.** Every row of a lemma carries
  the same root: `luces` carries the one of `luz`. It is not a segmentation of
  the form.
- **It is not a morpheme nor an etymological root**, it is a **truncation**:
  `gato` → `gat`, `niño` → `niñ`, `cantar` → `cant`.
- **It loses the written accents**, which is how the algorithm ends: `canción` →
  `cancion`, `vacío` → `vaci`, `álbum` → `album`.
- **It is useful for grouping families, and it also overgroups.** Right:
  `color`, `coloración`, `colorado`, `colorante` and `colorar` share `color`.
  Wrong: `par` (the pair) and `parar` (to stop) share `par`, and they are not a
  family. And it undergroups: `cabeza` → `cabez` but `cabezal` → `cabezal`.

And what it does overall is in the figures below: **three out of four roots have
a single lemma**, meaning that for most of them the column groups nothing. It is
an approximate search aid, not a morphological analysis.

<!-- ESTADISTICAS -->
<!-- figures measured on the table: do not edit by hand -->

## What is inside

| | |
|---|---:|
| rows | 1,098,090 |
| lemmas | 92,086 |
| distinct forms | 747,416 |
| distinct roots | 63,653 |
| roots with a single lemma | 48,736 |
| cross-references in `variant-lemma.tsv` | 2,990 |

## By part of speech

| `pos` | rows | lemmas | forms |
|---|---:|---:|---:|
| VERB | 869,300 | 12,315 | 564,123 |
| NOUN | 136,537 | 60,002 | 131,601 |
| ADJ | 89,173 | 26,179 | 88,695 |
| ADV | 2,318 | 2,212 | 2,283 |
| INTERJ | 486 | 458 | 479 |
| PRON | 163 | 100 | 131 |
| CONJ | 54 | 52 | 54 |
| PREP | 52 | 50 | 51 |
| DET | 7 | 4 | 6 |

## Gender and number

| `gender` | rows |
|---|---:|
| — | 888,294 |
| m. | 107,029 |
| f. | 97,320 |
| m. y f. | 5,447 |

| `number` | rows |
|---|---:|
| — (the verb does not carry it: its number is its person's) | 863,277 |
| sg | 121,690 |
| pl | 113,123 |

## Verb inflection

**869,300 rows** over 12,315 lemmas.

| `mood` · `tense` | rows |
|---|---:|
| subjuntivo · Pretérito imperfecto o Pretérito | 170,090 |
| indicativo · Presente | 95,886 |
| subjuntivo · Presente | 85,307 |
| indicativo · Futuro simple o Futuro | 85,091 |
| indicativo · Condicional simple o Pospretérito | 85,091 |
| indicativo · Pretérito perfecto simple o Pretérito | 85,065 |
| indicativo · Pretérito imperfecto o Copretérito | 85,059 |
| subjuntivo · Futuro simple o Futuro | 85,045 |
| imperativo · Imperativo | 53,249 |
| Participio | 10,680 |
| Infinitivo | 10,663 |
| Gerundio | 10,655 |
| — | 7,419 |

| `person_pronoun` | rows |
|---|---:|
| usted | 106,406 |
| ustedes | 106,386 |
| tú | 106,384 |
| vosotros | 106,268 |
| él | 96,026 |
| ellos | 95,816 |
| yo | 95,726 |
| nosotros | 95,640 |
| — (non-finite form, or the lemma alone) | 39,417 |
| vos | 21,231 |

<!-- /ESTADISTICAS -->

## Licence

**CC0 1.0: public domain, and no need to credit anyone.** Copy, redistribute,
transform and sell it, without asking permission and without attribution. The
text is in [`LICENSE`](LICENSE).

And the reason it asks for no attribution is the one it has: **there is no work
of authorship here**. This is the inflection of a language applied by rule, and
neither the rules nor the words belong to anyone. Charging a citation for
copying a table of plurals would be charging a toll on the language.

