This a small automaton to prepare biological data banks to be ready to use by bioinformatics tools.
It is based on ansible.

It is able to download a bank or genome, uncompress it if necessary and creates indexes for a panel of
bioinformatics tools like blast+, golden, bowtie2, bwa, ...
The systems can be easily extend to new kind of indexes or source of data.

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
The workflows are designed in roles/tasks/main.yml
Two parameters must be set for each  role:
* bank_src: the url where the bank/genome can be downloaded
* bank_name: the final name you want to give to this bank/genome 
A tag is also associated for each role to allow to rebuild only one bank/genome
of course you can add tags to rebuild a set of banks.

The workflows are design in roles but lot of step are common between workflows.
So this common steps are design in common_tasks directory and are includes in the workflows (aka roles).
 
## Provided tasks

### setup_general_bank_tree

prepare the file tree for the bank and indexes 

### setup_specific_bank_tree

create the file tree for this bank
 * create the directory for this bank
 * create the subdirectory uncompressed

### get_data

* check if data (basename of src_bank) is in distbanks, if none download it.
* copy the bank in uncompressed directory corresponding to this bank

It uncompress data if necessary and rename files according to bank_name
defined in role

### link_fasta

Create link in general fasta directory toward the fasta file in fasta directory of the bank

### blast2_indexing

* clean the blast2 indexes link in general directory
* remove the blast2 indexes
* then compute new indexes
* and creates new links


# Provided Workflows

## bank in fasta format

![Alt text](http://g.gravizo.com/g?
  digraph G {
  aize ="4,4";
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
  setup_general_bank_tree -> setup_specific_bank_tree;
  setup_specific_bank_tree -> get_data;
  get_data -> copy_indexes;
  }
)