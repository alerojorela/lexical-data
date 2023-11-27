# Lexical data for spanish language

> This repository content is still under review. Enhancements will be applied.

Lexical data for **spanish**

- forms-lemma-root-pos relationships
  - lemmas-forms (N:N relationship):
    - retrieve lemmas from forms: *fui* -> *ser, ir*
    - retrieve forms from lemmas: ser -> soy, eres ... sea ... sed

  - roots:
    - retrieve root from lemmas (via lemma from forms)
    - retrieve lemmas from roots
    - retrieve forms from roots

  - POS (part of speech)




## TO-DO

- [ ] add roots

  - [x] applying SnowballStemmer to lemmas (not to forms)

  - [ ] check automatic generation

- [ ] femenine gender generation
  - [ ] check automatic generation
  - tildes: héroe, ína
  - diverged forms: emperador, triz *from lat. *imperātor**
  - mambí, sa
- [ ] generate plural forms (rules on https://www.rae.es/dpd/plural)
  - [ ] check automatic generation
- [ ] add exceptions
  - [ ] corrections
