# Test 3

## By Katelin Hubbard :smile:

Lets nest some things!

Making two directories and moving in! :)

```

mkdir test3
cd test3
mkdir question2a
cd question2a

```

mirror in needed files:

```

ln -s /pickett_shared/teaching/EPP622_Fall2022/raw_data/citrus_test2/Maustralasica_genome.4.fasta
ln -s /pickett_shared/teaching/EPP622_Fall2022/raw_data/citrus_test2/microcitrus_australasica_pacbio.4.fastq
ln -s /pickett_shared/teaching/EPP622_Fall2022/raw_data/citrus_test2/microcitrus_australasica_nanopore.4.fastq

```

and then list to make sure theyre all there and teal colored!


## Now on to mapping! 🥳


We are going to map the two reads against the reference genome independently. 

Nanopore first! 


```

/sphinx_local/software/minimap2-2.24_x64-linux/minimap2 \
        -ax map-ont \
        --eqx \
        -o GenomeNanopore.sam \
        -t 2 \
        Maustralasica_genome.4.fasta \
        microcitrus_australasica_nanopore.4.fastq \
        >& minimap2_output
        
   ```
   
   
   Yay! Now lets do PacBio! Do not ask me why I decided to name the file "test3genome". It was stupid, I do not know why. I will fix it in sniffles.
   
   
   ```
   
/sphinx_local/software/minimap2-2.24_x64-linux/minimap2 \
        -ax map-hifi \
        --eqx \
        -o Test3genome.sam \
        -t 2 \
        Maustralasica_genome.4.fasta \
        microcitrus_australasica_pacbio.4.fastq \
        >& minimap2_output
        
   ```
   
   
   Now we need to call structural varaints with sniffles. (PacBio first because it ran faster even though I ran it five mins after the first one.
   
   
   
   ```
   
   spack load /r67sol
   samtools view -b -@ 2 Test3genome.sam > PacBiosniffiles.bam
   samtools sort -@ 2 PacBiosniffiles.bam -o PacBiosniffles.sorted.bam
   samtools index PacBiosniffles.sorted.bam
   
   conda create -n sniffles
   conda activate sniffles
   conda install sniffles=2.0

   sniffles --input PacBiosniffles.sorted.bam --vcf pacbiosniffles.vcf -t 2
   conda deactivate
   
   ```
   
   
   Now lets do this with Nano because she decided to grace us with the sam file by finishing the run. lol Also without any capitalization this time because, again, I can be dumb but I also learn. haha
   
   Also, I ran the whole prompt even though I was pretty sure I didnt need to run the create or install lines but I was running everything parallel on different terminals and I wanted to be sure everything was loaded correctly. 
   
   
   ```
   
   spack load /r67sol
   samtools view -b -@ 2 GenomeNanopore.sam > nanoporesniffiles.bam
   samtools sort -@ 2 nanoporesniffiles.bam -o nanoporesniffles.sorted.bam
   samtools index nanoporesniffles.sorted.bam
   conda activate sniffles
   conda install sniffles=2.0

   sniffles --input nanoporesniffles.sorted.bam --vcf nanoporesniffles.vcf -t 2
   conda deactivate
   
   ```
   
   
   Lets run the stats on the VCF:
   
   PacBio first since shes playing nice.
   
   
   
   ```
   
   conda create -n tabix -c bioconda tabix 
   conda activate tabix
   bgzip pacbiosniffles.vcf
   tabix pacbiosniffles.vcf.gz
conda deactivate

spack load bcftools
bcftools stats pacbiosniffles.vcf.gz > pacbiosniffles.vchk
plot-vcfstats pacbiosniffles.vchk -p plots/

```

And now, NanoPore. 

   ```
   conda activate tabix
   bgzip nanoporesniffles.vcf
   tabix nanoporesniffles.vcf.gz
conda deactivate

spack load bcftools
bcftools stats nanoporesniffles.vcf.gz > nanoporesniffles.vchk
plot-vcfstats nanoporesniffles.vchk -p plots/

```

Okay now lets pull these to my computer to look at them:
on a terminal running on my computer...

```
scp khubbar7@sphinx.ag.utk.edu:/home/khubbar7/test3/question2a/PacBiosniffles.sorted.bam.bai .
scp khubbar7@sphinx.ag.utk.edu:/home/khubbar7/test3/question2a/PacBiosniffles.sorted.bam .
scp khubbar7@sphinx.ag.utk.edu:/home/khubbar7/test3/question2a/pacbiosniffles.vcf .
scp khubbar7@sphinx.ag.utk.edu:/home/khubbar7/test3/question2a/pacbiosniffles.vcf.gz.tbi .
scp khubbar7@sphinx.ag.utk.edu:/pickett_shared/teaching/EPP622_Fall2022/raw_data/citrus_test2/Maustralasica_genome.4.fasta .
```


## Back on my terminal, lets start counting! 🥳

Finding out how many deletions:

```
grep -c '<DEL>' pacbiosniffles.vcf
```

Answer: 200 deletions

Finding out how many duplications:

```
grep -c '<DUP>' pacbiosniffles.vcf
```
Answer: 30

Finding out how many insertions

```
 grep -c '<INS>' pacbiosniffles.vcf
 
```


Answer: 19

Finding out how many inversions:

```
grep -c '<INV>' pacbiosniffles.vcf
        
```

Answer: 126

Finding out how many translocations:

```
grep -c '<BND>' pacbiosniffles.vcf
```

Answer: 0

![image](https://user-images.githubusercontent.com/115577500/199860501-6a0fe358-953b-47f0-a548-6dcd25e8e26e.png)


## Lets start comparing!!

Because I am lazy im just going to grep for them which will look like this, but change out the letters in the pointy parenthesis to match the correct insertion (INS) deletion (DEL), ect

```
grep '<INS>' pacbiosniffles.vcf
```


Deletion

Scaffold_4      26116645        Sniffles2.DEL.2EDES0    N       <DEL>   60      PASS    PRECISE;SVTYPE=DEL;SVLEN=-400;END=26117045;SUPPORT=30;COVERAGE=73,72,71,70,73;STRAND=+-;AF=0.423;STDEV_LEN=0.000;STDEV_POS=0.000        GT:GQ:DR:DV     0/1:60:41:30
        
 So this code says its is a deletion, (SVTYPE), it starts at 26116645, ends at 26117045, is 400 nucleotides long (SVLEN). Which is exactly what we see because when I load it into IGV and look up the ID (Scaffold_4:26,116,343-26,117,416) I see the 400 bp deletion at the bottom. 
        
        
![image](https://user-images.githubusercontent.com/115577500/199862492-5c11419c-4c70-4313-ace1-70433f992dc6.png)

        

Duplication

Scaffold_4      26142922        Sniffles2.DUP.306CS0    N       <DUP>   60      PASS    PRECISE;SVTYPE=DUP;SVLEN=4785;END=26147707;SUPPORT=9;COVERAGE=66,32,42,None,None;STRAND=+-;AF=0.231;STDEV_LEN=0.000;STDEV_POS=0.000     GT:GQ:DR:DV     0/1:6:30:9

This is a 4785 bp duplication starting at 26142922 and ending at 26147707. This is what we see in IGV 

Insertion
        
Scaffold_4      745766  Sniffles2.INS.E6S0      N       <INS>   57      PASS    PRECISE;SVTYPE=INS;SVLEN=1315;END=745766;SUPPORT=13;COVERAGE=62,20,20,20,52;STRAND=+-;AF=0.650;STDEV_LEN=0.000;STDEV_POS=0.000;SUPPORT_LONG=0   GT:GQ:DR:DV     0/1:33:7:13
        
Inversion
        
Scaffold_4      22800790        Sniffles2.INV.33D9S0    N       <INV>   60      PASS    PRECISE;SVTYPE=INV;SVLEN=410447;END=23211237;SUPPORT=25;COVERAGE=40,4,81,0,62;STRAND=-;AF=0.397;STDEV_LEN=0.000;STDEV_POS=0.000 GT:GQ:DR:DV     0/1:60:38:25
        







   
   
