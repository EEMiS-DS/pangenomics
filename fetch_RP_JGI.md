Softlink archives here and extract

```bash
for i in $(cat batch1.screened); do echo $i; ln -s $i .; done
for i in $(ls *.tar.gz); do tar -xzf $i; done
```

Get dictionary taxID : JGI_sample_name, eg 2754412842 : JGI_SA12_F08_054

```python
a = open('batch1.screened.IDs', 'r')
outf = open('assembly_table_JGI', 'w')

for l in a:
    l = l.split('\t')
    taxID = l[0].split('/')[-1].split('.')[0]
    JGI_ID = l[1].split()[-1]
    outf.write('%s\t%s\n' % (taxID, JGI_ID))

outf.close()
```

Generate list of proteins compliant to Pfam names, eg Ribosomal_<proteinName>.

proteins to download: L2, L3, L4, L5, L6, L14, L15, L16, L18, L22, L24, S3, S8, S10, S17, S19

```bash
parseRibosomalProteins.JGI.py "*/*.genes.faa" batch1.screened batch1.screened.IDs
```

Add sequences to alignment

```bash
alignmentFolder="/home/domeni/projects_b2016308/TOL/Anantharaman_et.al_NatComm_RP_alignments"

mafft --thread 16 \
--inputorder \
--add batch1.screened_Ribosomal_L14.fa \
--auto $alignmentFolder/L14_NR_alignment.fasta > batch1.screened_Ribosomal_L14.afa
```