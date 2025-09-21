# methods
The general methodology approached here was to first download all of the forward and reverse reads from every sample, then use spades to assemble the contigs de novo.
Blast was then used to identify the species for each genome by BLASTing ~300 base pairs of a random location in the first and largest contig assembled by spades. The logic here is that ~300 base pairs is a large enough segment to be able to identify a species using BLAST. Abricate was used on the contigs to generate antimicrobial resistance and virulence factor profiles using the  ncbi antimicrobial resistance database and vfdb. Toxins were identified via manual curation of virulence factors.

# results:
## blast results
All of the blast results show that my organism is listeria monocytogenes except one which shows  brucella (SRR27013245) 

## antimicrobial resistances and recommended treatments
These samples have shown resistances for the following antibiotics:
- Fosfomycin (all isolates)
- Lincosamide (all isolates)
Recommended treatment: 
an aminolycoside such as gentamicin or a beta lactam such as ampicillin, penicillin, or amoxicillin. 
## virulence factors and toxins
many virulence factors, but the toxins seem to be:
- phospholipase C (plcB and plcA)
- listeriolysin O and its precursors and associated proteins
Everything else seems to be a virulence factor associated with either internalization (internalins), movement (actin tail formation), competitive survival (bacteriocins), and environmental survival (bile hydrolysins), as well as some stress regulators and adhesins (fbpA).

# public health discussion
The listeria monocytogenes outbreak was most likely a case of incorrect handling of food. The outbreak was also most likely not contained quick enough due to the antimicrobial resistance of the isolates making it more difficult to eradicate the bacterium. There is a good chance that the antimicrobial resistance is now widespread amongst nosocomial strains of L monocytogenes. As such, I believe that any future treatments should include lincosamide or fosfomycin if its an environmental strain, but if there is no response to the treatment, swapping the antibiotic of choice to a beta lactam seems to be a good idea.
