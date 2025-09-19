# pseudo code

First thing to do is to produce my genomes... 
I have to:
1. do a quality check
	1. use fastq for this
2. trim the reads
	1. use fastp for this
3. do a quality check again
	1. use fastq
4. use spades to make a denovo genome
	1. remember to check the genome after using bandage and quast
5. use BLAST to find out the organism by looking for its 16s rna
	1. I need to annotate the genome first...
	2. I can instead get a random piece of the first contig and I can do that by breaking the contigs.fasta into an array delimitted by ">" and then getting a random number between 100 and the number of bases in the first contig and then reading from that number and 300 base pairs down (just remember to remove all empty spaces)
6. then find the antimicrobial resistome by checking through all of them using abricate
7. identify toxic genes using toxinome database online and then just use BLAST for now...

Then after that I can write up the report!
1. break it into:
	1. methods, results, discussion
2. annotate the code to clarify what's going on


`mapfile -d \> pootis < "assembly/contigs.fasta"`
`contig_length=$(echo "${pootis[1]}" | wc -c)`
`random_base=$(shuf -i 100-"$contig_length" -n 1)`
```
first_element="${pootis[1]}"
first_element_substring=${first_element:$random_base:300} #the second number is the length, while the first number is the offset location
first_element_substring=$(echo $first_element_substring | tr -d ' ' )
```
```
dir_array=($(ls)) # stores ls as an array into dir_array

dir_array=( "${dir_array[@]/$element}" ) #this will remove any instance of anything that has that element string from the array and then store it again in the same named array

dir_array2=( "${dir_array[@]}" ) #this will store an existing bash array into a new array

#note: when you remove the element in the for loop from the array, it will jump to the next element without skipping! the same should be applicable to anything deleted, I'm assuming...

i=1
for fastq_name in dir_array; do
	file1="$fastq_name"
	file2="${dir_array[1]}" #this will always be 1 because we keep deleting the first two elements at the end of every loop...
	genome_sample_number="${fastq_name:0:11}"
	
	#here we delete the first two elements:
	dir_array=( "${dir_array[@]/"$genome_sample_number"}" ) #this will remove all instances of the sample number...
	
	
i=1
for fastq_name in dir_array; do
	if [[ i%2 == 0 ]]; then
		i++
		continue
	fi
	i++
	file1=dir_array
	
	
	
	#THIS IS PROBABLY IT!!!!!! TAKE A BREAK AND THEN COME BACK AND INTERPRET IT AND GO THOUGH IT TO SEE IF IT WOULD WORK AND IF THERE ARE ANY ISSUES WITH THE DIRECTORY NAMES ETC.
array_length=$( echo "${dir_array[@]}" | wc -w)
for i in {1.."$array_length"}; do 
	if [[ i%2 == 0 ]]; then
		continue
	fi
	file1="${dir_array[i]}"
	file2="${dir_array[i+2]}"
	genome_sample_number="${file1:0:11}"
	## continue what you want to do here:
	
	#trim and do qc using fastp and then output it into the trimmed files
	fastp -i fastq_files/"$file1" -I fastq_files/"$file2" \
	-o trimmed/"${file1:0:-9}".trim.fastq.gz \
	-O trimmed/"${file2:0:-9}".trim.fastq.gz
	
	#do fastqc on both of those files
	#mkdir -p qc
	#fastqc trimmed/"$file1".* -O qc/ && fastqc trimmed/"$file2".* -O qc/
	
	#make the assembly folder and output the assembly using spades there!
	#after proof of function, we can then try piping it
	mkdir -p "$genome_sample_number"_assembly
	spades.py -1 trimmed/"${file1:0:-9}".trim.fastq.gz \
	-2 trimmed/"${file2:0:-9}".trim.fastq.gz --phred-offset 33 --isolate \
	-o "$genome_sample_number"_assembly/
	
	dir_array=( "${dir_array[@]/"$genome_sample_number"}" )
	
	#remove the temp files
	#rm -rf trimmed
	
	#get a 300 bp string and then blast it!
	mapfile -d \> contig_array \
	< "$genome_sample_number""_assembly/contigs.fasta" 
	#here we use mapfile to store the contig fasta into an array which is delemited using the '>' symbol
	contig_length=$(echo "${contig_array[1]}" | wc -c) #this is the approximate length of the first and longest contig; we use wc -c for the first element of the array
	random_base=$(shuf -i 100-"$contig_length" -n 1) #this is a random base from 100 to the length of the first contig
	
	first_element="${contig_array[1]}" # this is the first and longest contig, called as the first element of the array
	first_element_substring=${first_element:$random_base:300} #the second number is the length, while the first number is the offset location
	blast_query=$(echo $first_element_substring | tr -d ' ' )
	
	#now I have to blast it...
	mkdir -p blast_results
	blastn -db nt -query <(echo "$blast_query") -remote \
	-max_target_seqs 10 \
	-out blast_results/"$genome_sample_number"_blast_results.out #this gives the top 10 matches and searches online through the NCBI -nt database
	
	#get my AMR genes!
	mkdir -p AMR
	abricate "$genome_sample_number"_assembly/contigs.fasta > AMR/"$genome_sample_number"_amr_tab.tab
	
	#get my virulence factors/toxins!
	mkdir -p VFs
	abricate --db vfdb "$genome_sample_number"_assembly/contigs.fasta > VFs/"$genome_sample_number"_VFs_tab.tab	
done
```

The final code is: 
```
#!/bin/bash

dir_array=($(ls fastq_files))
array_length=$( echo "${dir_array[@]}" | wc -w)
for ((z=0;z<="$array_length";z++)); do
	if (( $z%2 == 1 )); then
		continue
	fi
	let j=$z+1
	file1="${dir_array[$z]}"
	echo "$file1"
	file2="${dir_array[$j]}"
	echo "$file2"
	genome_sample_number="${file1:0:11}"
	echo "$genome_sample_number"
	## continue what you want to do here:

	echo "trimming time!"
	#trim and do qc using fastp and then output it into the trimmed files
	fastp -i fastq_files/"$file1" -I fastq_files/"$file2" \
	-o trimmed/"${file1:0:-9}".trim.fastq.gz \
	-O trimmed/"${file2:0:-9}".trim.fastq.gz

	echo "trim complete!"
	echo "time to build the genome!"
	#do fastqc on both of those files
	#mkdir -p qc
	#fastqc trimmed/"$file1".* -O qc/ && fastqc trimmed/"$file2".* -O qc/

	#make the assembly folder and output the assembly using spades there!
	#after proof of function, we can then try piping it
	mkdir -p "$genome_sample_number"_assembly
	spades.py -1 trimmed/"${file1:0:-9}".trim.fastq.gz \
	-2 trimmed/"${file2:0:-9}".trim.fastq.gz --phred-offset 33 \
	--threads 24 -o "$genome_sample_number"_assembly/

	#remove the temp files
	#rm -rf trimmed

	#get a random 300 bp string and then blast it!
	#here we use mapfile to store the contig fasta into an array which is delemited using the '>' symbol
	mapfile -d \> contig_array \
	< "$genome_sample_number""_assembly/contigs.fasta"

	contig_length=$(echo "${contig_array[1]}" | wc -c) #this is the approximate length of the first and longest contig; we use wc -c for the first element of the array
	random_base=$(shuf -i 100-"$contig_length" -n 1) #this is a random base from 100 to the length of the first contig

	first_element="${contig_array[1]}" # this is the first and longest contig, called as the first element of the array
	first_element_substring=${first_element:$random_base:300} #the second number is the length, while the first number is the offset location
	blast_query=$(echo $first_element_substring | tr -d ' ' )

	#now I have to blast it...
	mkdir -p blast_results
	blastn -db nt -query <(echo "$blast_query") -remote \
	-max_target_seqs 10 \
	-out blast_results/"$genome_sample_number"_blast_results.out #this gives the top 10 matches and searches online through the NCBI -nt database

	#get my AMR genes!
	mkdir -p AMR
	abricate "$genome_sample_number"_assembly/contigs.fasta > AMR/"$genome_sample_number"_amr_tab.tab

	#get my virulence factors/toxins!
	mkdir -p VFs
	abricate --db vfdb "$genome_sample_number"_assembly/contigs.fasta > VFs/"$genome_sample_number"_VFs_tab.tab	
done
```


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