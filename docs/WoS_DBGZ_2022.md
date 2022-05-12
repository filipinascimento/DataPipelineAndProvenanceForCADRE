# Accessing the Web of Science data

Three versions of the Web of Science data are available for the science genome project, corresponding to the years 2019, 2020, and 2022. All include the CORE dataset and Science Citation Index Expanded. 

## XML Raw Files
The data are originally available as raw XML files according to the WOS Schema and can be found in the `IUNI1` machine.

Paths to the XML files and number of entries for each version:

| Version | XML Path in `IUNI1`| Count |
| --- | --- | --- |
| 2019 | `/raw/WoS_18_19` | 74883966 |
| 2020 | `/raw/WoS_20_21` | 78395509 |
| 2022 | `/raw/WoS_2022` | 84162157 |


## Documentation for the RAW Files
Clarivate provides documentation for the XML raw files in:
https://clarivate.libguides.com/c.php?g=593069&p=4101845

Including the Schema user guide:
https://clarivate.libguides.com/ld.php?content_id=27109663


## Fast access to the RAW data via DBGZ (only 2022)
A [DBGZ](https://github.com/filipinascimento/dbgz) version of the 2022 data is available both in `IUNI1` and `Cinder`/`Ember` machines. This provides a fast way to access the raw data. About 2 hours are needed to iterate over all the entries in sequence. The framework also provides a way to access the entries in a random order based on the `WOS ID`(UID).

### Installation
You need to use the `dbgz` python package to open it (make sure you have at least version 0.5):

```bash
pip install -U dbgz
```

### Location
The aggregated file can be found in
```bash
/gpfs/sciencegenome/WOS/WoS_2022_All.dbgz
```
for `Cinder`/`Ember` and
```bash
/raw/WOS/WoS_2022_All.dbgz
```
for `IUNI1`.


### Usage
To iterate over all the entries, you can use the following code:

```python
from pathlib import Path
import dbgz
from tqdm.auto import tqdm
WOSArchivePath = Path("/gpfs/sciencegenome/WOS/WoS_2022_All.dbgz")

with dbgz.DBGZReader(WOSArchivePath) as fd:
    # getting the number of entries
    print("\t Number of entries: ", fd.entriesCount)
    # getting the schema (currently UID and data)
    print("\t Scheme: ", fd.scheme)
    # TQDM can be used to print the progressbar
    for entry in tqdm(fd.entries, total=fd.entriesCount):
        UID = entry["UID"]
        entryData = entry["data"] # XML records data
        ... # do something with the data
```

The entries are all based on the RAW XML and were not further processed. It contains only two fields at the first level:
- `UID`: WoS ID
- `data`: Python dictionary directly converted from the XML record (see the XML guides). The only addition is the field `origin` which stores the name of the original zip file containing this entry.


### Random access to the WoS entries
We also provide an index based on UID, which allow random access to the entries data:
```bash
/gpfs/sciencegenome/WOS/WoS_2022_All_byUID.idbgz
```
for `Cinder`/`Ember` and
```bash
/raw/WOS/WoS_2022_All_byUID.idbgz
```
for `IUNI1`

If using in the science-genome systems, we recommend to move the `.dbgz` and `.idbgz` to a local folder (such as `/home/<user>` or `/data/`), otherwise the random access can become slow.

Example usage of the index:
```python
import dbgz
from pathlib import Path
from tqdm.auto import tqdm

WOSIndexPath = Path("/gpfs/sciencegenome/WoS_2022_DBGZ/WoS_2022_All_byUID.idbgz")
print("Loading the index dictionary")
indexDictionary = dbgz.readIndicesDictionary(WOSIndexPath)
wosUID = "WOS:000200919700142" # or any other UID
with dbgz.DBGZReader(WOSArchivePath) as fd:
    wosEntry = fd.readAt(indexDictionary[wosUID][0])
...
```


## JSON Version (deprecated)
The data for 2019 and 2020 were processed and converted to JSON files by means of the following spark script:

https://github.com/iuni-cadre/DataPipelineAndProvenanceForCADRE/blob/master/wosParseYan/WoScompactTable.ipynb

The dataset for 2019 contains `74883966` entries and for 2020, `78395509` entries.

The files json for 2019 and 2020 are available in:

```bash
/gpfs/sciencegenome/WoSjson2019/
```
and
```bash
/gpfs/sciencegenome/WoSjson2020/
```

Processing the JSON files may take over 8 hours to finish. Please check [[Fast raw access to WoS 2019 2020]] for an alternative way to quickly access the Web of Science data from the aggregated version.



### JSON Schema (old)
The schema for the JSON files can be found in:

https://github.com/iuni-cadre/DataPipelineAndProvenanceForCADRE/blob/master/wosParseYan/WOS_TabDelimitedFields.csv

### Extra data (old)

The citation edges list can also be accessed from the paths:
```bash
/gpfs/sciencegenome/WoSjson2019/citeEdges.csv/
```
and
```bash
/gpfs/sciencegenome/WoSjson2019/json/citeEdges.csv/
```
Please, note that these paths are directories and not files.

### Misc. (old)
 - Cleaned from raw XML: `/gpfs/sciencegnome/WoSjson2019/`
 - citing and cited edges: `/gpfs/sciencegnome/WoSjson/WoSedges/`
 - 2019 update: `/gpfs/sciencegnome/WoSjson2019/`
 - 2019 cited edges: `/gpfs/sciencegnome/WoSjson2019/WoSedges/citeEdges.csv/`
 - 2019 WoS keywords, keywords plus and category_info, `/gpfs/sciencegnome/WoSjson2019/category_keywords_updated.csv` (updated to include multiple labels for category_info)
 - XML schema: (https://github.com/iuni-cadre/Collaborative-projects/blob/master/xmlFieldsCompact.csv)

 - Descriptions of the fields: (https://github.com/iuni-cadre/DataPipelineAndProvenanceForCADRE/blob/master/wosParseYan/WOS_TabDelimitedFields.csv)

 - Detailed data dictionary (Updated for the 2019 version):
https://cadre.iu.edu/resources/documentation

- Doi-UID mapping table: 
  - File: `/gpfs/sciencegnome/WoSjson2019/uid-doi-mapping-table.csv` 
  - Script: `/gpfs/sciencegenome/WoSjson2019/make_wos_doi_mapping_table.py`
