# MultiTax

Pyhton library that provides a common interface to obtain, parse and interact with biological taxonomies (GTDB, NCBI, Silva, Greengenes, Open Tree taxonomy) + "Custom" formatted taxonomies.

## Goals
 
 - Common interface to use different taxonomies
 - Fast, intuitive, generalized and easy to use
 - Enable integration and compatibility with multiple taxonomies without any effort
 - Translation between taxonomies (not yet implemented)

## Installation

    python setup.py install --record files.txt

## Documentation
    
https://pirovc.github.io/multitax/

## Basic Example with GTDB

    from multitax import GtdbTx
    
    # Download taxonomy
    tax = GtdbTx()

    # Get lineage for the Escherichia genus  
    tax.lineage("g__Escherichia")
    # ['1', 'd__Bacteria', 'p__Proteobacteria', 'c__Gammaproteobacteria', 'o__Enterobacterales', 'f__Enterobacteriaceae', 'g__Escherichia']

## Further Examples

[List of all functions](https://pirovc.github.io/multitax/multitax/multitax.html)

```python
from multitax import GtdbTx

# Download and parse in memory
tax = GtdbTx()

# Parse local files
tax = GtdbTx(files=["bac120_taxonomy.tsv.gz", "ar122_taxonomy.tsv.gz"])

# Download, write and parse files
tax = GtdbTx(output_prefix="my/path/") 

# List parent node
tax.parent("g__Escherichia")
# f__Enterobacteriaceae

# List children nodes
tax.children("g__Escherichia")
# ['s__Escherichia flexneri', 's__Escherichia coli', 's__Escherichia dysenteriae', 's__Escherichia coli_D', 's__Escherichia albertii', 's__Escherichia marmotae', 's__Escherichia coli_C', 's__Escherichia sp005843885', 's__Escherichia sp000208585', 's__Escherichia fergusonii', 's__Escherichia sp001660175', 's__Escherichia sp004211955', 's__Escherichia sp002965065']

# Get specific rank parent node
tax.parent_rank("s__Lentisphaera araneosa", "phylum")
# 'p__Verrucomicrobiota'

# Get lineage
tax.lineage("g__Escherichia")
# ['1', 'd__Bacteria', 'p__Proteobacteria', 'c__Gammaproteobacteria', 'o__Enterobacterales', 'f__Enterobacteriaceae', 'g__Escherichia']

# Get lineage of names
tax.name_lineage("g__Escherichia")
# ['root', 'Bacteria', 'Proteobacteria', 'Gammaproteobacteria', 'Enterobacterales', 'Enterobacteriaceae', 'Escherichia']

# Get lineage of ranks
tax.rank_lineage("g__Escherichia")
# ['root', 'domain', 'phylum', 'class', 'order', 'family', 'genus']

# Get specific lineage
tax.lineage("g__Escherichia", root_node="p__Proteobacteria", ranks=["phylum", "class", "family", "genus"])
# ['p__Proteobacteria', 'c__Gammaproteobacteria', 'f__Enterobacteriaceae', 'g__Escherichia']

# Pre-build lineages in memory for faster access
tax.build_lineages()

# Get leaf nodes
tax.leaves("p__Hadarchaeota")
# ['s__DG-33 sp004375695', 's__DG-33 sp001515185', 's__Hadarchaeum yellowstonense', 's__B75-G9 sp003661465', 's__WYZ-LMO6 sp004347925', 's__B88-G9 sp003660555']

# Show stats of loaded tax
tax.stats()
#{'leaves': 31910,
# 'names': 45503,
# 'nodes': 45503,
# 'ranked_leaves': Counter({'species': 31910}),
# 'ranked_nodes': Counter({'species': 31910,
#                          'genus': 9428,
#                          'family': 2600,
#                          'order': 1034,
#                          'class': 379,
#                          'phylum': 149,
#                          'domain': 2,
#                          'root': 1}),
# 'ranks': 45503}

# Filter ancestors (desc=True for descendants)
tax.filter(['g__Escherichia', 's__Pseudomonas aeruginosa'])
tax.stats()
#{'leaves': 2,
# 'names': 11,
# 'nodes': 11,
# 'ranked_leaves': Counter({'genus': 1, 'species': 1}),
# 'ranked_nodes': Counter({'genus': 2,
#                          'family': 2,
#                          'order': 2,
#                          'class': 1,
#                          'phylum': 1,
#                          'domain': 1,
#                          'species': 1,
#                          'root': 1}),
# 'ranks': 11}

# Write tax to file
tax.write("custom_tax.tsv", cols=["node", "rank", "name_lineage"])

#g__Escherichia             genus    root|Bacteria|Proteobacteria|Gammaproteobacteria|Ent#erobacterales|Enterobacteriaceae|Escherichia
#f__Enterobacteriaceae      family   root|Bacteria|Proteobacteria|Gammaproteobacteria|Enterobacterales|Enterobacteriaceae
#o__Enterobacterales        order    root|Bacteria|Proteobacteria|Gammaproteobacteria|Enterobacterales
#c__Gammaproteobacteria     class    root|Bacteria|Proteobacteria|Gammaproteobacteria
#...
```

## The same goes for the other taxonomies

```python
# NCBI
from multitax import NcbiTx
tax = NcbiTx(files="taxdump.tar.gz")
tax.lineage("561")    
# ['1', '131567', '2', '1224', '1236', '91347', '543', '561']

# Silva
from multitax import SilvaTx
tax = SilvaTx(files="tax_slv_ssu_138.1.txt.gz")
tax.lineage("46463")    
# ['1', '3', '2375', '3303', '46449', '46454', '46463']

# Open Tree taxonomy
from multitax import OttTx
tax = OttTx(files="ott3.2.tgz")
tax.lineage("474503")
# ['805080', '93302', '844192', '248067', '822744', '768012', '424023', '474503']

# GreenGenes
from multitax import GreengenesTx
tax = GreengenesTx(files="gg_13_5_taxonomy.txt.gz")
tax.lineage("f__Enterobacteriaceae")
# ['1', 'k__Bacteria', 'p__Proteobacteria', 'c__Gammaproteobacteria', 'o__Enterobacteriales', 'f__Enterobacteriaceae']
```

## LCA integration

Using pylca: https://github.com/pirovc/pylca

```python
from pylca.pylca import LCA
from multitax import GtdbTx

tax = GtdbTx()
L = LCA(tax._nodes)

L("s__Escherichia dysenteriae", "s__Pseudomonas aeruginosa")
# 'c__Gammaproteobacteria'
```

## General information

 - Taxonomies are parsed into nodes. Each node is annotated with a name and a rank.
 - A single root node is defined by default. This node can be changed `root_node` when loading the taxonomy (as well as annotations `root_parent`, `root_name`, `root_rank`).
 - Standard values for unknown/undefined nodes can be configured with `undefined_node`,`undefined_name` and `undefined_rank`. Those are default values returned when nodes/names/ranks are not found.
 - Taxonomy files are automatically download or can be loaded from disk (`files` parameter). Alternative `urls` can be provided. When downloaded, files are handled in memory. It is possible to save the downloaded file to disk with `output_prefix`.
 
## Translation between taxonomies

Not yet implemented. The goal here is to map different taxonomies if the linkage data is available. That's what I think it will be possible.

 |from/to |NCBI   |GTDB   |SILVA   |OTT   |GG  |
 |--------|-------|-------|--------|------|----|
 |NCBI    |-      |part   |part    |part  |no  |
 |GTDB    |full   |-      |no      |no    |no  |
 |SILVA   |full   |no     |-       |part  |no  |
 |OTT     |part   |no     |part    |-     |no  |
 |GG      |no     |no     |no      |no    |-   |

## Further ideas

- Add/remove nodes
- Write on specific taxonomy format

## Similar projects

- https://github.com/FOI-Bioinformatics/flextaxd
- https://github.com/shenwei356/taxonkit
- https://github.com/bioforensics/pytaxonkit
- https://github.com/chanzuckerberg/taxoniq
