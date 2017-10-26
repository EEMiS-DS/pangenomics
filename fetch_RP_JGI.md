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

Delete all Archeal seqs from alignments but two (outgroups)

```python
from Bio import SeqIO
import glob

ark2keep = ["Archaea_Thaumarchaeota_Thaumarchaeota_archaeon_strain_BS4", "Archaea_Euryarchaeota_Methanosarcinales_5m_scaffold_1030"]

def removeArk(seqFile, arkList, ark2keep):
    seqHandle = SeqIO.parse(seqFile, 'fasta')
    outHandle = open("./noArk/" + seqFile.replace('.fasta', '.noArk.fasta'), 'w')
    for s in seqHandle:
        if s.id in arkList and s.id not in ark2keep:
            continue
        else:
            outHandle.write('>%s\n%s\n' % (s.id, str(s.seq)))
    outHandle.close()

arkList = [i.split()[0] for i in open("arkID", 'r').readlines()]

seqFiles = glob.glob("*_NR_alignment.fasta")
for seqFile in seqFiles:
    removeArk(seqFile, arkList, ark2keep)
```

Add sequences to alignment (run from interactive development node)

```bash
# salloc -p devel -t 1:00:00 -A b2016308

alignmentFolder="/pica/v9/b2016308_nobackup/projects/TOL/Anantharaman_et.al_NatComm_RP_alignments/noArk"
wdir="/pica/v9/b2016308_nobackup/projects/JGI_CSP_data/SAG"
finalOutDir="/pica/v9/b2016308_nobackup/projects/JGI_CSP_analyses/phylogenySAG"
mkdir -p $finalOutDir

module load bionfo-tools
module load MAFFT

mafft --thread 16 \
--inputorder \
--add ${wdir}/batch1.screened_Ribosomal_L14.fa \
--auto ${alignmentFolder}/L14_NR_alignment.fasta > ${finalOutDir}/batch1.screened_Ribosomal_L14.afa
```

Concatenate MSAs

```bash
sbatch -p core -t 5:00:00 -A b2016308 \
-J MSA -o MSA.out -e MSA.err \
--mail-type=ALL --mail-user=domenico.simone@lnu.se,annalisa.16@hotmail.it<<'EOF'
#!/bin/bash
 
export PATH=/proj/b2016308/glob/:$PATH
concatenateMSA.py
EOF
```

And remove residual Archaeal sequences :)
