  <!-- badges: start -->
  [![Launch Rstudio Binder](http://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/IgnasiLucas/T7_BLAST/soca?urlpath=lab)
  <!-- badges: end -->

# Tema 7. BLAST

## Continguts

Aquest repositori inclou materials didàctics per la realització d'exercicis pràctics
de bioinformàtica mitjançant la plataforma mybinder.org. Han estat confeccionats
seguint el tutorial [*From Zero to Binder in R!*](https://github.com/alan-turing-institute/the-turing-way/blob/master/workshops/boost-research-reproducibility-binder/workshop-presentations/zero-to-binder-r.md)
de l'Alan Turing Institute.

La pràctica del tema 7 està orientada a executar cerques avançades amb BLAST-delta

Els continguts inclouen, a banda d'aquest README.md, els arxius següents:

| Arxiu                | Descripció                                                            | 
| -------------------- |:--------------------------------------------------------------------- |
| AAPB.txt             | Identificadors de taxons de bacteris aerobis anoxigènics fototròfics. |
| guion.ipynb          | Guió de pràctiques.                                                   |
| LPOR_Ath.fas         | Seqüència de l'enzim LPOR d'*Arabidopsis thaliana*.                   |
| LPOR_candidates.txt  | Llista de LPOR descrits per Chernomor et al. (2020; taula S1).        |
| runtime.txt          | Indicació de la data de referència per la versió d'R.                 |
| swissprot.*          | Arxius de la base de dades swissprot per al BLAST (veure secció 2).   |

## Base de dades

L'execució de cerques BLAST des d'una instància de MyBinder té algune dificultats:

- No és possible utilitzar l'eina `update_blastdb.pl` de blast+ des de MyBinder,
  perquè, si no m'equivoque, depén d'una connexió ftp, que en MyBinder estàn limitades.
- No és fàcil incloure una base de dades en un repositori de Github, on la mida dels
  arxius està limitada a 100M (mida màxima recomanada: 50M).
- No funciona l'opció `-remote` dels programes blast. Almenys jo no l'he poguda fer
  funcionar, i he assumit que està també limitada en MyBinder (ftp?).

La solució és incloure la base de dades en el repositori de Github, però reconstruïda
amb una mida màxima d'arxiu (opció `-max_file_sz` de `makeblastdb`). Però atenció, perquè
cal haver descarregat i tenir disponible un mapa de taxonomia per tal d'incloure la
informació taxonòmica de cada seqüència de la base de dades en la reconstrucció. Altrament
les opcions `-taxids` i `-taxidlist` dels programes blast no funcionaran.

Finalment, el que he fet ha sigut:

```
mkdir tmp
cd tmp
update_blastdb.pl swissprot`
cd ..
blastdbcmd -db tmp/swissprot -entry all -out swissprot.fas`
```

La descàrrega dels identificadors ha durat més de 4 hores:

```
wget https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/idmapping/idmapping_selected.tab.gz
gunzip idmapping_selected.tab.gz
cut -f 1,13 idmapping_selected.tab > taxid_map.txt
makeblastdb -in swissprot.fas -dbtype prot -title swissprot \
            -parse_seqids -out swissprot -max_file_sz 50M
mv tmp/taxdb* ./
rm idmapping_selected.tab
rm -r tmp
```

El codi anterior no l'he executat tal qual, i pot incloure algun error.

## Instal·lació de blast

Com que el kernel que necessite en MyBinder és el d'R, la forma de definir l'entorn
és mitjançant l'script d'R `install.R`, que ha d'instal·lar els paquets especificats.
Per tal d'instal·lar paquets fora d'R he intentat incloure un document .yml amb un
entorn de conda que inclou blast, però no ha funcionat.

Entenc que en MyBinder, el notebook amb kernel d'R s'executa en un entorn de conda
concret ("notebook"?) i que no està previst modificar eixe entorn mitjançant un
document yaml. Per tal d'instal·lar blast+ des de la sessió oberta en MyBinder,
el que hem fet és utilitzar conda des de la terminal:

`conda install -c bioconda blast=2.10.1`

Això hauria de funcionar. Si no, es podria instal·lar la versió per defecte i
després intentar actualitzar. El cas és que no és possible crear un entorn nou,
perquè aleshores afectem el funcionament del notebook. El kernel d'R sembla
desaparéixer.

