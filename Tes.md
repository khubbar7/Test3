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




   
   
