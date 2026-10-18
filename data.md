# Downloading the data

This project uses the **Genomes to Fields (G2F) 2022 Maize Genotype by Environment Prediction Competition** dataset. The data is not stored in this repo (`Training_Data/` and `Testing_Data/` are in `.gitignore`), so you need to download it before running anything.

- **Dataset page:** https://dc.cyverse.org/dataset/d4534168-8aa3-4f4b-aaca-94ae926d46e9
- **DOI:** [10.25739/tq5e-ak26](https://doi.org/10.25739/tq5e-ak26)
- **Data paper:** Lima et al. 2023, *BMC Research Notes*, [10.1186/s13104-023-06421-z](https://doi.org/10.1186/s13104-023-06421-z)
- **License:** ODC Public Domain Dedication and License (ODC-PDDL). No account is needed.
- **Total size:** about 425 MB to download. The genotype VCF unzips to about 8.6 GB.

## Option 1: Command line with `gocmd` (recommended)

`gocmd` is CyVerse's official download tool. Use it on servers: plain `wget`/`curl` requests to `data.cyverse.org` often get stuck on a browser verification page.

### 1. Install `gocmd` (Linux x86_64)

```bash
mkdir -p ~/bin && cd ~/bin
U=https://github.com/cyverse/gocommands/releases/download/v0.12.5/gocmd-v0.12.5-linux-amd64.tar.gz
curl -L -o gocmd.tar.gz $U
curl -sL $U.md5; md5sum gocmd.tar.gz   # the two checksums should match
tar zxf gocmd.tar.gz && chmod +x gocmd
./gocmd --version
```

For other platforms, see the [gocommands releases](https://github.com/cyverse/gocommands/releases).

### 2. Configure anonymous access

```bash
mkdir -p ~/.irods_cyverse_anon
cat > ~/.irods_cyverse_anon/irods_environment.json <<'EOF'
{
  "irods_host": "data.cyverse.org",
  "irods_port": 1247,
  "irods_user_name": "anonymous",
  "irods_zone_name": "iplant",
  "irods_authentication_scheme": "native"
}
EOF
```

### 3. Download

Run this from the repo root:

```bash
P=/iplant/home/shared/commons_repo/curated/GenomesToFields_GenotypeByEnvironment_PredictionCompetition_2023

# Optional: list what is available
~/bin/gocmd -c ~/.irods_cyverse_anon ls -l $P/Training_data $P/Testing_data

mkdir -p _download
~/bin/gocmd -c ~/.irods_cyverse_anon get -r \
  $P/Training_data $P/Testing_data $P/COMPETITION_DATA_README.docx $P/readme.txt \
  ./_download/
```

### 4. Move files into place

```bash
mkdir -p Training_Data Testing_Data docs_data
mv _download/Training_data/* Training_Data/
mv _download/Testing_data/*  Testing_Data/
mv _download/COMPETITION_DATA_README.docx _download/readme.txt docs_data/
rm -r _download

cd Training_Data && unzip 5_Genotype_Data_All_Years.vcf.zip && cd ..
```

The repo already contains `Training_Data/Hybrid_subpopulations`, `Training_Data/year_list.txt` and `Testing_Data/Test.txt`. None of the downloaded files have those names, so they won't be overwritten.

## Option 2: Browser

Open these folders, pass the verification check if one appears, and download each file into the matching local folder:

- Training: https://data.cyverse.org/dav-anon/iplant/home/shared/commons_repo/curated/GenomesToFields_GenotypeByEnvironment_PredictionCompetition_2023/Training_data → `Training_Data/`
- Testing: https://data.cyverse.org/dav-anon/iplant/home/shared/commons_repo/curated/GenomesToFields_GenotypeByEnvironment_PredictionCompetition_2023/Testing_data → `Testing_Data/`

Keep the original filenames, because the scripts look for them by name. Then unzip `5_Genotype_Data_All_Years.vcf.zip` inside `Training_Data/`.

## Expected files

After downloading, you should have:

```
Training_Data/
├── 1_Training_Trait_Data_2014_2021.csv          26.5 MB   yield + traits, one row per plot (target)
├── 2_Training_Meta_Data_2014_2021.csv            80 KB    field / location metadata
├── 3_Training_Soil_Data_2015_2021.csv            23 KB    soil tests
├── 4_Training_Weather_Data_2014_2021.csv        8.3 MB    daily weather
├── 5_Genotype_Data_All_Years.vcf                8.6 GB    genetic markers (~437k markers, 4,928 hybrids)
├── 5_Genotype_Data_All_Years.vcf.zip            386 MB    (can be deleted after unzipping)
├── 6_Training_EC_Data_2014_2021.csv             1.6 MB    environmental covariates
├── All_hybrid_names_info.csv                    278 KB    hybrid names and parents
├── GenoDataSources.txt                          214 KB
├── GenoDataSourcesWithUpdatedBioProject.txt     168 KB
├── Hybrid_subpopulations                                  (in repo)
└── year_list.txt                                          (in repo)

Testing_Data/
├── 1_Submission_Template_2022.csv               404 KB    Env x Hybrid pairs to predict
├── 2_Testing_Meta_Data_2022.csv                  12 KB
├── 3_Testing_Soil_Data_2022.csv                 3.4 KB
├── 4_Testing_Weather_Data_2022.csv              731 KB
├── 6_Testing_EC_Data_2022.csv                   249 KB
├── Test_Set_Observed_Values_ANSWER.csv          529 KB    actual 2022 yields (released after the competition)
└── Test.txt                                               (in repo)

docs_data/
├── COMPETITION_DATA_README.docx                           description of every file
└── readme.txt
```

## Notes

- **How the data is split:** the organisers split by year. 2014–2021 is training and 2022 is testing. `Test_Set_Observed_Values_ANSWER.csv` holds the 2022 yields as one mean per location × hybrid, not per plot.
- **Columns differ slightly between years:** the 2022 soil file is missing 6 micronutrient columns (Boron, Copper, Iron, Manganese, Zinc, BpH). About 130 rows of the 2022 weather file are blank in the original data.
- **Text encoding:** `2_Training_Meta_Data_2014_2021.csv` is not UTF-8. Read it with `encoding="cp1252"` (or `latin-1`).
- **Generated files:** several scripts expect processed files that are not part of this download, such as the cleaned/imputed genotype VCF (`*_maf005_maxmiss090_cleaned_BEAGLE_imputed.vcf`), weather/EC PCA tables, and `env_list.txt`. These have to be generated from the raw data above.
