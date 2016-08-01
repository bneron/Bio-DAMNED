This based on BIO-DAMNED project ([fork me here](https://github.com/bneron/Bio-DAMNED)) 
to prepare biological data banks to be ready to use by bioinformatics tools.
It is based on ansible.

It is able to download a bank or genome, uncompress it if necessary and creates indexes for a panel of
bioinformatics tools like blast+, golden, bowtie2, bwa, ... .
The system can be easily extend to new kind of indexes or source of data.

# Requirements

 1. ansible >= 2.1.0.0
 2. and bioinformatics tools needed to creates indexes 
    * makeblastdb
    * goldin
    * bowtie2
    * bwa
    * ...
 
# Architecure

The playbook give you an over view of all banks you have configured. 
Each role match to a bank.
In a role we assign a predefined workflow given the kind of data.
The workflows are designed in `roles/<name of role>/tasks/main.yml`
Two parameters must be set for each  role:
* bank_src: the url where the bank/genome can be downloaded
* bank_name: the final name you want to give to this bank/genome 
A tag is also associated for each role to allow to rebuild only one bank/genome
of course you can add tags to rebuild a set of banks.

(_genome_ is formed of __one__ sequence, _bank_ is a __set of sequences__)
The workflows are design in roles but lot of step are common between workflows.
So this common steps are design in common_tasks directory and are includes in the workflows (aka roles).
 
## Provided tasks

### setup_general_bank_tree

Prepare the file tree for the bank and indexes.

### setup_specific_bank_tree

Create the file tree for this bank
1. create the directory for this bank.
2. create the subdirectory `uncompressed`.

### get_data

1. check if data (basename of src_bank) is in `distbanks` directory, if none download it.
2. copy the bank in `uncompressed` directory corresponding to this bank.

It uncompress data if necessary and rename files according to bank_name
defined in role.

### link_fasta

Create link in general fasta directory toward the fasta file in fasta directory of the bank

### blast2_indexing

1. clean the blast2 indexes link in general directory
2. remove the blast2 indexes
3. then compute new indexes
4. and creates new links


# Provided Workflows

Below the diagrams of the workflows used to prepare the bank/genome
* The ellipse box mean this step is in common_tasks, then it can used in different workflows. 
  The description of this step is done in upper paragraph "Provided tasks"
* the square box mean this step is directly coded in the workflow, then it is specific to this workflow.

## bank in fasta format

![Alt text](http://g.gravizo.com/g?
  digraph G {
  aize ="4,4";
  specific_fasta_stuff [shape=box];
  setup_general_bank_tree -> setup_specific_bank_tree;
  setup_specific_bank_tree -> get_data;
  get_data -> specific_fasta_stuff;
  specific_fasta_stuff -> link_fasta;
  link_fasta -> blast2_indexing;
  }
)

## genome in fasta format
![Alt text](http://g.gravizo.com/g?
  digraph G {
  aize ="4,4";
  bank_fasta -> bowtie2_indexing;
  bowtie2_indexing -> bwa_indexing;
  }
)
  
## bank in genbank format
![Alt text](http://g.gravizo.com/g?
  digraph G {
  aize ="4,4";
  reformatting_in_fasta [shape=box];
  setup_general_bank_tree -> setup_specific_bank_tree;
  setup_specific_bank_tree -> get_data;
  get_data -> reformatting_in_fasta;
  reformatting_in_fasta -> link_fasta;
  link_fasta -> golden_indexing;
  golden_indexing -> blast2_indexing;
  }
)
  
## genome in genbank format

![Alt text](http://g.gravizo.com/g?
  digraph G {
  aize ="4,4";
  bank_genkank -> bowtie2_indexing;
  bowtie2_indexing -> bwa_indexing;
  }
)

## blast bank (set of blast indexes)

![Alt text](http://g.gravizo.com/g?
  digraph G {
  aize ="4,4";
  copy_indexes [shape=box];
  setup_general_bank_tree -> setup_specific_bank_tree;
  setup_specific_bank_tree -> get_data;
  get_data -> copy_indexes;
  }
)